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

# Intro to [Gym](https://gym.openai.com/docs/#available-environments)

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

![img](https://i.stack.imgur.com/wGuj5.png)

首先是policy evaluation，重点是这里的四重循环：

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

david silver说过

>Complexity 是 $O(mn^2) $per iteration, for m actions and n states