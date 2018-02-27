---
title: David Silver 强化学习 1
categories:
 - Reinforcement Learning
tag:
 - ML
---
Reinforcement learning 是LeCun所说的cake顶端的cherry，可见是非常有意思了。David Silver的公开课非常出名，今天立一个flag三月底之前，学完全部课程并且完成project。

# Material

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/intro_RL.pdf)

[Video1](https://www.youtube.com/watch?v=2pWv7GOvuf0)(English)

[Video2](https://www.bilibili.com/video/av9831889/?from=search&seid=14117299732296388423)(Chinese)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf)

# About RL

## Many Faces of RL

![many_faces_of_RL](..\assets\images\post_images\强化学习1\many_faces_of_RL.PNG)

## Differences of other ML paradigms and  RL

* There is no supervisor, only a reward signal
* Feedback is delayed, not instantaneous
* Time really matters (sequential, non i.i.d(non independent and identically distributed~非独立同分布) data)
* Agent’s actions affect the subsequent data it receives (这里有一点类似于**active learning**不知道有没有人写过将RL应用于主动学习的paper)

# The RL Problem

## Rewards

A reward $R_t$ is a scalar feedback signal indicates how well agent is doing at step t. The agent’s job is to maximise cumulative reward.

### Definition (Reward Hypothesis)

All goals can be described by the maximisation of expected cumulative reward

### Sequential Decision Making

* Actions may have long term consequences

* Reward may be delayed

* It may be better to sacrifice immediate reward to gain more long-term reward

  Examples:

> A financial investment (may take months to mature)
 Refuelling a helicopter (might prevent a crash in several hours)
 Blocking opponent moves (might help winning chances many
 oves from now)

## Environments

### Agent and Environment

## ![environments](..\assets\images\post_images\强化学习1\environments.PNG)

## State

### History and State

* The **history** is the sequence of observations, actions, rewards
  $$
  H_t = O_1,R_1,A_1,...,A_{t-1},O_t,R_t
  $$

* i.e. all observable variables up to time t

* i.e. the sensorimotor stream of a robot or embodied agent

* What happens next depends on the history:

  * The agent selects actions
  * The environment selects observations/rewards

* **State** is the information used to determine what happens next

* Formally, state is a function of the history:
  $$
  S_t=f(H_t)
  $$





### Environment State

* The environment state $S_t^e$ is the environment’s private representation
* i.e. whatever data the environment uses to pick the next observation/reward
* The environment state is not usually visible to the agent
* Even if $S_t^e$ is visible, it may contain irrelevant information

## Agent State

* The agent state  $S_t^e$ is the agent’s internal representation
* i.e. whatever information the agent uses to pick the next action
* i.e. it is the information used by reinforcement learning algorithms
* It can be any function of
  history:
  $$
  S_t=f(H_t)
  $$




## Information State

An information state (a.k.a.(*also known as*) **Markov state**) contains all useful information from the history.

### Definition

> A state  $S_t^e$ is Markov if and only if 
> $$
> P[S_{t+1}|S_t] = P[S_{t+1}|S_1,...,S_t]
> $$
>

* “The future is independent of the past given the present”
  $$
  H_{1:t}->S_t->H_{t+1:\infty}
  $$

* Once the state is known, the history may be thrown away
* i.e. The state is a sufficient statistic of the future
* The environment state $S_t^e$ is Markov
* The history $H_t$ is Markov