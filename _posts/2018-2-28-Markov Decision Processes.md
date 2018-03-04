---
title: RL-2 Markov Decision Processes
categories:
 - Reinforcement Learning
tag:
 - ML,RL
typora-root-url: ..
typora-copy-images-to: ..\assets\images\post_images\强化学习2
---
这一节讲了MP, MRP, MDP, **Bellman Equation**以及解法。

# Material

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/MDP.pdf)

[Video1](https://www.youtube.com/watch?v=lfHX2hHRMVQ)(English)

[Video2](https://www.bilibili.com/video/av9833386/)(Chinese)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf)

# Markov Processes

## Intro to MDPs

* **Markov decision processes** formally describe an *environment* for reinforcement learning

* Where the environment is *fully observable*

* The current state completely characterises the process

* Almost all RL problems can be formalised as MDPs

## Markov Property

> The future is independent of the past given the present

**Definition**

A state $S_t$is *Markov* if and only if

 $$\mathbb{P}[S_{t+1}|S_t]=\mathbb{P}[S_{t+1}|S_1,...,S_t]$$

* The state captures all relevant information from the history
* Once the state is known, the history may be thrown away
* i.e. The state is a sufficient statistic of the future

### State Transition Matrix

![State Transition Matrix](/assets/images/post_images/强化学习2/State Transition Matrix.PNG)

## Markov Chains

### Markov Process

A Markov process is a memoryless random process, i.e. a sequence of random states $S_1, S_2$, ... with the Markov property

**Definition**

A Markov Process (or Markov Chain) is a tuple $<S,\mathcal{P}>$

* $\mathcal{S}$ is a  (finite) set of states
* $\mathcal{P}​$ is a state transition probability matrix, 
  $$\mathcal{P}_{ss\prime}=\mathbb{P}[S_{t+1}=s\prime|S_t=s]$$

# Markov Reward Process

## MRP
A Markov reward process is a Markov chain with values.

**Definition**

A *Markov Reward Process* is a tuple $<\mathcal{S,P,R,\gamma}>$

* $\mathcal{S}$ is a finite set of states

* $\mathcal{P}$ is a state transition probability matrix,

  $$\mathcal{P_{ss\prime}=}\mathbb{P}[\mathcal{S_{t+1}}=s\prime|S_t=s]$$

* $\mathcal{R}$ is a reward function, 

  $$\mathcal{R_s}= \mathbb{E}[R_{t+1}|S_t=s]$$

* $\gamma$  is a  discount factor, $\gamma \in[0,1]$ 

## Return

**Definition**

The **return $G_t$** is the total discounted reward from time-step t.

$$G_t = R_{t+1}+\gamma R_{t+2}+...=\sum_{k=0}^{\infty}\gamma ^kR_{t+k+1}$$

### Why discount?

* **Mathematically convenient** to discount rewards
* Avoids **infinite** returns in cyclic Markov processes
* **Uncertainty** about the future may not be fully represented
* ......

## Value Function

The value function $v(s)$ gives the long-term value of state $s$

**Definition**

The **state-value function** $v(s)$ of an MRP is the **expected return**
starting from state $s$

$$v(s)=\mathbb{E}[G_t|S_t=s]$$

(其中$\mathbb{E}$的意思为**expected value**, 即**数学期望**)

### Example: Student MRP Returns

![example1](/assets/images/post_images/强化学习2/example1.PNG)

**NOTICE!!!**

* The **returns** of sample 是 random的, **value function** 并不是随机的，而是**expectation** of those random variables(**returns**) !!!!

* **G ** => **Goal** 下标**t**是**time**

* **return**的定义中不含**expected**

  > The **return $G_t$** is the **total discounted reward** from time-step t.

  但是**value function**含有

  > The state **value function** $v(s)$ of an MRP is the **expected return**
  > starting from **state** $s$

* 与**return**相关的是**time**，与**value function**相关的是**state**

## Bellman Equation

> Do **recursive decomposition（递归分解）** based on value function

The value function can be decomposed into two parts:

* immediate reward $R_{t+1}$
* discounted value of successor state $\gamma v(S_{t+1})$

![bellman equation1](/assets/images/post_images/强化学习2/bellman equation1.PNG)

![1519908935088](/assets/images/post_images/强化学习2/1519908935088.png)

### Bellman Equation in Matrix Form(**重点**)![bellman equation2](/assets/images/post_images/强化学习2/bellman equation2.PNG)

### Solving the Bellman Equation

* The Bellman equation is a linear equation 

* It can be solved directly:

  $$\mathcal{v=R+\gamma Pv}$$
  $$\mathcal{(I-\gamma P)v=R}$$
  $$\mathcal{v=(I-\gamma P)^{-1}}R$$

* Computational complexity is $O(n^3)$ for $n$ states

* Direct solution only possible for small MRPs

* There are many iterative methods for large MRPs, e.g.

  * Dynamic programming
  * Monte-Carlo evaluation
  * Temporal-Difference learning


# Markov Decision Process

A Markov decision process (MDP) is a **Markov reward process** with
**decisions**. It is an environment in which **all states are Markov**.

**Definition**

A *Markov Decision Process* is a tuple $<\mathcal{S,A,P,R,\gamma}>$

* $\mathcal{S}$ is a finite set of states

* $\mathcal{A}$ is a finite set of actions

* $\mathcal{P}$ is a state transition probability matrix,

  $$\mathcal{P_{ss\prime}^a=}\mathbb{P}[\mathcal{S_{t+1}}=s\prime|S_t=s,A_t=a]$$

* $\mathcal{R}$ is a reward function, 

  $$\mathcal{R_s^a}= \mathbb{E}[R_{t+1}|S_t=s,A_t=a]$$

* $\gamma$  is a  discount factor, $\gamma \in[0,1]$ 

## Policy

**Definition**

A policy π is a distribution over actions given states

$$\pi(a|s)=\mathbb{P}[A_t=a|S_t=s]$$

> 这个公式是stochastic policy。
>
> 公式里没有reward是因为reward已经被囊括在state之中

* A policy fully defines the behaviour of an agent

* MDP policies depend on the current state (not the history)

* i.e. Policies are stationary (time-independent),

  $$A_t\sim \pi(\cdot|S_t),\forall t>0$$

> Policy实际上是一个随即转移矩阵（stochastic transition matrix）
>
> time-independent 但是 state dependent

![policy](/assets/images/post_images/强化学习2/policy.PNG)

## Value Function

**Definition**

* ***state-value*** function:  The state-value function $v_π(s)$ of an MDP is the expected return starting from **state** s, and then following policy π

  $$v_{\pi}=\mathbb{E}_{\pi}[G_t|S_t=s]$$

* ***action-value*** function:  The action-value function $q_π(s, a)$ is the expected return starting from state s, taking action a, and then following policy π

  $$q_{\pi}(s,a)=\mathbb{E}_{\pi}[G_t|S_t=s,A_t=a]$$

> 两种value-function的区别在于是否take action

## Bellman Expectation Equation（重点）

The state-value function can again be decomposed into **immediate reward** plus **discounted value** of successor state,

$$v_{\pi}(s)={\mathbb{E}_{\pi}[R_{t+1}+\gamma v_{\pi}(S_{t+1})|S_t=s]}$$

The action-value function can similarly be decomposed,

$$q_{\pi}(s,a)=\mathbb{E_{\pi}}[R_{t+1}+\gamma q_{\pi}(S_{t+1},A_{t+1}|S_t=s,A_t=a)]$$

![1520144576465](/assets/images/post_images/强化学习2/1520144576465.png)

## Optimal Value Function

The optimal **state-value** function $v_*(s)$ is the maximum value
function over all policies

$$v_*(s)=max_{\pi}v_{\pi}(s)$$

The optimal **action-value** function $q_*(s,a)$ is the maximum action-value function over all policies

$$q_*(s,a)=max_{\pi}q_{\pi}(s,a)$$

* The optimal value function specifies the best possible
  performance in the MDP
* An MDP is “solved” when we know the optimal value fn.

![1520144868320](/assets/images/post_images/强化学习2/1520144868320.png)

### Optimal Policy

Define a partial ordering over policies

$$\pi\ge\pi\prime \hspace{2mm} if  \hspace{2mm} v_\pi(s)\ge v_{\pi\prime}(s),\forall s$$

![1520145255686](/assets/images/post_images/强化学习2/1520145255686.png)