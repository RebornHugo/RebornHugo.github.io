---
title: RL-2 Markov Decision Processes
categories:
 - Reinforcement Learning
tag:
 - ML,RL
---
这一节讲了马尔可夫决策过程。

# Material

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/MDP.pdf)

[Video1](https://www.youtube.com/watch?v=lfHX2hHRMVQ)(English)

[Video2](https://www.bilibili.com/video/av9833386/)(Chinese)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf)

# Markov Processes

## Intro to MDPs

* Markov decision processes formally describe an *environment* for reinforcement learning

* Where the environment is *fully observable*

* The current state completely characterises the process

* Almost all RL problems can be formalised as MDPs

## Markov Property

> The future is independent of the past given the present

**Definition**

A state $S_t$is *Markov* if and only if $$\mathbb{P}[S_{t+1}|S_t]=\mathbb{P}[S_{t+1}|S_1,...,S_t]$$

* The state captures all relevant information from the history
* Once the state is known, the history may be thrown away
* i.e. The state is a sufficient statistic of the future

### State Transition Matrix

![State Transition Matrix](..\assets\images\post_images\强化学习2\State Transition Matrix.PNG)

## Markov Chains

### Markov Process

A Markov process is a memoryless random process, i.e. a sequence of random states $S_1, S_2$, ... with the Markov property

**Definition**

A Markov Process (or Markov Chain) is a tuple $<S,\mathcal{P}>$

* $\mathcal{S}$ is a  (finite) set of states
* $\mathcal{P}$ is a state transition probability matrix, $$\mathcal{P}_{ss\prime}=\mathbb{P}[S_{t+1}=s\prime|S_t=s]$$

# Markov Reward Process

## MRP
A Markov reward process is a Markov chain with values.

**Definition**

A *Markov Reward Process* is a tuple $<\mathcal{S,P,R,\gamma}>$

* $\mathcal{S}$ is a finite set of states

* $\mathcal{P}$ is a state transition probability matrix,

  $\mathcal{P_{ss\prime}=}\mathbb{P}[\mathcal{S_{t+1}}=s\prime|S_t=s]$

* $\mathcal{R}$ is a reward function, $\mathcal{R_s}= \mathbb{E}[R_{t+1}|S_t=s]$

* $\gamma$  is a  discount factor, $\gamma \in[0,1]$ 

## Return

**Definition**

The **return $G_t$** is the total discounted reward from time-step t.

$$G_t = R_{t+1}+\gamma R_{t+2}+...=\sum_{k=0}^{\infty}\gamma ^kR_{t+k+1}$$

### Why discount?

* Mathematically convenient to discount rewards
* Avoids infinite returns in cyclic Markov processes
* **Uncertainty** about the future may not be fully represented
* ......

## Value Function

The value function $v(s)$ gives the long-term value of state $s$

**Definition**

The state value function $v(s)$ of an MRP is the **expected return**
starting from state $s$

$$v(s)=\mathbb{E}[G_t|S_t=s]$$

### Example: Student MRP Returns

![example1](D:\Coding\WorkSpace\RebornHugo.github.io\assets\images\post_images\强化学习2\example1.PNG)

**NOTICE!!!**

* The **returns** of sample 是 random的, **value function** 并不是随机的，而是**expectation** of those random variables(**returns**) !!!!

* **G ** => **Goal** 下标**t**是**time**

* **return**的定义中不含**expected**

  > The **return $G_t$** is the **total discounted reward** from time-step t.

  但是**value function**含有

  > The state **value function** $v(s)$ of an MRP is the **expected return**
  > starting from **state** $s$

* 与**return**相关的是**time**，与**value function**相关的是**state**