---
title: RL-3 实做RL_DP之frozen-lake
categories:
 - Reinforcement Learning
tag:
 - ML,RL,Coding

typora-copy-images-to: ..\assets\images\post_images\实做动态规划
typora-root-url: ..
---
实做强化学习下的动态规划，以8x8的frozen_lake为demo，openai的Gym toolkit为工具。**Policy Iteration**（evaluation 和 improvement两部分）和**Value Iteration**算法的理解以及Gym的使用为重点内容。

# Material

[Gym-Documentation](https://gym.openai.com/docs/)

[FrozenLake8x8-v0](https://gym.openai.com/envs/FrozenLake8x8-v0/)

[Solution](https://github.com/llSourcell/navigating_a_virtual_world_with_dynamic_programming)

[Video by Siraj](https://www.youtube.com/watch?v=5R2vErZn0yw&feature=youtu.be)

# Intro to the Environment

以下是openai[官方](https://gym.openai.com/envs/FrozenLake8x8-v0/)对frozen_lake的完整描述：

> Winter is here. You and your friends were tossing around a frisbee at the park when you made a wild throw that left the frisbee out in the middle of the lake. The water is mostly frozen, but there are a few holes where the ice has melted. If you step into one of those holes, you'll fall into the freezing water. At this time, there's an international frisbee shortage, so it's absolutely imperative that you navigate across the lake and retrieve the disc. However, the ice is slippery, so you won't always move in the direction you intend.
>
> The surface is described using a grid like the following:
>
> ```
> SFFF       (S: starting point, safe)
> FHFH       (F: frozen surface, safe)
> FFFH       (H: hole, fall to your doom)
> HFFG       (G: goal, where the frisbee is located)
> ```
> The episode ends when you reach the goal or fall in a hole. You receive a reward of 1 if you reach the goal, and zero otherwise.

简而言之，就是从start出发，经过frozen，到达goal，如果顺利到达，则得到1的reward，如果落入hole，则获得0的reward。RL agent的target永远是maximize **expaected feature discounted total reward**，可以采用DP的方法来解决问题。

# Intro to Gym

[官方文档](https://gym.openai.com/docs/#available-environments)有使用教程, 核心API在[这里](https://github.com/openai/gym/blob/master/gym/core.py), example在[这里](https://github.com/openai/gym/tree/master/examples), 这里仅对几个功能进行说明：

## gym.make

`env = gym.make('CartPole-v0')`这里make的env来自[openai](https://gym.openai.com/envs/)

```python
from gym import envs
print(envs.registry.all())
#> [EnvSpec(DoubleDunk-v0), EnvSpec(InvertedDoublePendulum-v0), EnvSpec(BeamRider-v0), EnvSpec(Phoenix-ram-v0), EnvSpec(Asterix-v0), EnvSpec(TimePilot-v0), EnvSpec(Alien-v0), EnvSpec(Robotank-ram-v0), EnvSpec(CartPole-v0), EnvSpec(Berzerk-v0), EnvSpec(Berzerk-ram-v0), EnvSpec(Gopher-ram-v0), ...
```

> It’s very easy to add your own enviromments to the registry, and thus make them available for `gym.make()`: just [`register()`](https://github.com/openai/gym/blob/master/gym/envs/__init__.py) them at load time.

官方文档说可以轻松使用register功能添加自定义的env，然后make。

## Class DiscreteEnv

```python
class DiscreteEnv(Env):

    """
    Has the following members
    - nS: number of states
    - nA: number of actions
    - P: transitions (*)
    - isd: initial state distribution (**)

    (*) dictionary dict of dicts of lists, where
      P[s][a] == [(probability, nextstate, reward, done), ...]
    (**) list or array of length nS


    """
```

离散的env拥有四个常用属性

* environment.nS: states数量

* environment.nA: action数量

* environment.P: david silver说的transition dynamics

  ```python
environment.Ps[s][a] # 表示状态s，动作a下的被env blow后的可能结果，返回一个list，其中每一个element为一个四元组tuple： [(probability, nextstate, reward, done), ...]
  ```


* isd: 初始的state分布

## reset, render, step

```python
def reset(self):
        """Resets the state of the environment and returns an initial observation.
        Returns: observation (object): the initial observation of the
            space.
        """
        raise NotImplementedError
```

```PYTHON
def render(self, mode='human'):
        """Renders the environment.
        The set of supported modes varies per environment. (And some
        environments do not support rendering at all.) By convention,
        if mode is:
        - human: render to the current display or terminal and
          return nothing. Usually for human consumption.
        - rgb_array: Return an numpy.ndarray with shape (x, y, 3),
          representing RGB values for an x-by-y pixel image, suitable
          for turning into a video.
        - ansi: Return a string (str) or StringIO.StringIO containing a
          terminal-style text representation. The text can include newlines
          and ANSI escape sequences (e.g. for colors).
        Note:
            Make sure that your class's metadata 'render.modes' key includes
              the list of supported modes. It's recommended to call super()
              in implementations to use the functionality of this method.
        Args:
            mode (str): the mode to render with
            close (bool): close all open renderings
        Example:
        class MyEnv(Env):
            metadata = {'render.modes': ['human', 'rgb_array']}
            def render(self, mode='human'):
                if mode == 'rgb_array':
                    return np.array(...) # return RGB frame suitable for video
                elif mode is 'human':
                    ... # pop up a window and render
                else:
                    super(MyEnv, self).render(mode=mode) # just raise an exception
        """
        raise NotImplementedError
```

```PYTHON
def step(self, action):
        """Run one timestep of the environment's dynamics. When end of
        episode is reached, you are responsible for calling `reset()`
        to reset this environment's state.
        Accepts an action and returns a tuple (observation, reward, done, info).
        Args:
            action (object): an action provided by the environment
        Returns:
            observation (object): agent's observation of the current environment
            reward (float) : amount of reward returned after previous action
            done (boolean): whether the episode has ended, in which case further step() calls will return undefined results
            info (dict): contains auxiliary diagnostic information (helpful for debugging, and sometimes learning)
        """
        raise NotImplementedError
```

# Solution

主要思想如下图：

![img](https://i.stack.imgur.com/wGuj5.png)

首先map数字到上下左右

```python
# Action mappings
action_mapping = {
    0: '\u2191',  # UP
    1: '\u2192',  # RIGHT
    2: '\u2193',  # DOWN
    3: '\u2190',  # LEFT
}
```

定义函数play_episodes用来模拟采取policy后，每个episode之后的wins和reward。

## Policy Iteration

### policy evaluation

重点是这里的四重循环：

```python
# Repeat until value change is below the threshold
    for i in range(int(max_iter)):

        # Initialize a change of value function as zero
        delta = 0

        # Iterate though each state
        for state in range(environment.nS):

            # Initial a new value of current state
            v = 0

            # Try all possible actions which can be taken from this state
            for action, action_probability in enumerate(policy[state]):

                # Evaluate how good each next state will be
                for state_probability, next_state, reward, terminated in environment.P[state][action]:

                    # Calculate the expected value
                    v += action_probability * state_probability * (reward + discount_factor * V[next_state])
```

第一层循环是添加迭代的上限，防止无限迭代。
最外层的终止条件：`delta < theta` 这里的delte，是指在一次循环中变化最大的state的state value function的变化量，当小于theta（这里是1e-9）时，说明近乎converge，循环结束外层循环，否则继续，直到max_iter再终止。

david silver说过

> Complexity 是 $O(mn^2) $per iteration, for m actions and n states

对应的是这里的后三层循环。

第二层：对选定的current state遍历。

第三层：对当前状态下可能采取的action遍历(使用policy[state])

第四层：在current state以及采取了特定的action情况下可能转移到的next state进行遍历（使用environment.P[state][action\]）

* 后三层是对Bellman Expectation Equation for state value funtion 的具体实现

  ![1520562916986](/assets/images/post_images/实做动态规划/1520562916986.png)

### policy iteration

```python
    # Repeat until convergence or critical number of iterations reached
    for i in range(int(max_iter)):

        stable_policy = True

        # Evaluate current policy
        V = policy_evaluation(policy, environment, discount_factor=discount_factor)

        # Go through each state and try to improve actions that were taken
        for state in range(environment.nS):

            # Choose the best action in a current state under current policy
            current_action = np.argmax(policy[state])

            # Look one step ahead and evaluate if current action is optimal
            # We will try every possible action in a current state
            action_value = one_step_lookahead(environment, state, V, discount_factor)

            # Select a better action
            best_action = np.argmax(action_value)

            # If action didn't change
            if current_action != best_action:
                stable_policy = False

            # Greedy policy update
            policy[state] = np.eye(environment.nA)[best_action]

        evaluated_policies += 1
```

policy iteration 主要是调用写好的policy evaluation然后进行Greedy Policy Improvement。

greedy 的做法是使用policy evaluation后的state value function对每个state进行one_step_lookahead，即寻找到当前state的action_value（A vector of length nA containing the expected value of each action)

> 注意：policy evaluation 计算的是state value function， 这里的action_value 是action value function，就是david ppt上的q。

![1520564922186](/assets/images/post_images/实做动态规划/1520564922186.png)

计算出每个state的action_value就可以derive出当前policy下的best action，并且进行

```python
# Greedy policy update
policy[state] = np.eye(environment.nA)[best_action]
```

判断和当前policy[state]下的current action是否相同，如果所有state的current action 已经是best action了，则已找到optimal policy。



## Value Iteration

以下摘自stackoverflow
>  **Value iteration** includes: **finding optimal value function** + one **policy extraction**. There is no repeat of the two because once the value function is optimal, then the policy out of it should also be optimal (i.e. converged).

###  finding optimal value function 

首先，使用Bellman Optimality(注意不是 ~~expected~~)Equation 找到optimal value funtion

```python
    for i in range(int(max_iterations)):

        # Early stopping condition
        delta = 0

        # Update each state
        for state in range(environment.nS):

            # Do a one-step lookahead to calculate state-action values
            action_value = one_step_lookahead(environment, state, V, discount_factor)

            # Select best action to perform based on the highest state-action value
            best_action_value = np.max(action_value)

            # Calculate change in value
            delta = max(delta, np.abs(V[state] - best_action_value))

            # Update the value function for current state
            V[state] = best_action_value

        # Check if we can stop
        if delta < theta:
            print(f'Value-iteration converged at iteration#{i}.')
            break
```

计算current state value 时，要先进行one_step_lookahead，找到当前state下的所有action_value，然后取max，即为best_action_value，即为当前state的optimality value function。

![1520566079439](/assets/images/post_images/实做动态规划/1520566079439.png)

### policy extraction

然后进行 one **policy extraction**，不用像policy improvement一样和policy evaluation 一起迭代。

``` python
    # Create a deterministic policy using the optimal value function
    policy = np.zeros([environment.nS, environment.nA])

    for state in range(environment.nS):

        # One step lookahead to find the best action for this state
        action_value = one_step_lookahead(environment, state, V, discount_factor)

        # Select best action based on the highest state-action value
        best_action = np.argmax(action_value)

        # Update the policy to perform a better action at a current state
        policy[state, best_action] = 1.0

    return policy, V
```

对每一个state进行one_step_lookahead，找到当前state的best action，然后`policy[state, best_action] = 1.0`，即让给定state下采取best_action的概率设为1。

> 注意在policy extraction之前并没有使用到过policy。

以上是核心过程，最后在play_episodes中调用两种方法return 的policy，然后使用`environment.step(action)`即可。

```python
def play_episodes(environment, n_episodes, policy):
    wins = 0
    total_reward = 0

    for episode in range(n_episodes):

        terminated = False
        state = environment.reset()

        while not terminated:

            # Select best action to perform in a current state
            action = np.argmax(policy[state])

            # Perform an action an observe how environment acted in response
            next_state, reward, terminated, info = environment.step(action)

            # Summarize total reward
            total_reward += reward

            # Update current state
            state = next_state

            # Calculate number of wins over episodes
            if terminated and reward == 1.0:
                wins += 1

    average_reward = total_reward / n_episodes

    return wins, total_reward, average_reward
```

# Answer Screenshot

![1520566859763](/assets/images/post_images/实做动态规划/1520566859763.png)