---
title: RL-1 Introduction to Reinforcement Learning
categories:
 - Reinforcement Learning
tag:
 - ML,RL

typora-copy-images-to: ..\assets\images\post_images\强化学习1
typora-root-url: ..
---
Reinforcement learning 是LeCun所说cake顶端的cherry，可见是非常有意思了。David Silver的公开课非常著名。立一个flag，三月底之前，学完全部课程并且完成project。这篇blog是david silver第一讲的笔记，对RL做了一个简介，并着重讲解了什么是RL problem。

# Material

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/intro_RL.pdf)

[Video1](https://www.youtube.com/watch?v=2pWv7GOvuf0)(English)

[Video2](https://www.bilibili.com/video/av9831889/?from=search&seid=14117299732296388423)(Chinese)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf)

# About RL

## Many Faces of RL

![many_faces_of_RL](\assets\images\post_images\强化学习1\many_faces_of_RL.PNG)

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

## ![environments](\assets\images\post_images\强化学习1\environments.PNG)

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

### Agent State

* The agent state  $S_t^e$ is the agent’s internal representation
* i.e. whatever information the agent uses to pick the next action
* i.e. it is the information used by reinforcement learning algorithms
* It can be any function of
  history:
  $$
  S_t=f(H_t)
  $$













### Information State

An information state (a.k.a.(*also known as*) **Markov state**) contains all useful information from the history.

Definition

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

### Fully Observable Environments
**Full observability**: agent **directly** observes environment state

$$O_t = S_t^a=S_t^e$$

* Agent state = environment state = information state
* Formally, this is a **Markov decision process** (MDP)

### Partially Observable Environments

**Partial observability**: agent **indirectly** observes environment

* Now agent state $\neq $ environment state
* Formally this is a **partially observable Markov decision process**(POMDP)
* Agent must construct its own state representation $S_t^a$, e.g.
  * Complete history: $S_t^a=H_t$
  * **Beliefs** of environment state:$S_t^a=(P[S_t^e=s^n])$
  * Recurrent neural network: $S_t^a=\sigma(S_{t-1}^aW_s+O_tW_o)$

# Inside An RL Agent

Major Components of an RL Agent

* Policy: agent’s behaviour function
* Value function: how good is each state and/or action
* Model: agent’s representation of the environment

## Policy

* A policy is the agent’s behaviour

* It is a **map** from **state** to **action**， e.g.

* **Deterministic** policy: $a=\pi(s)$

* **Stochastic** policy:  

  $$\pi(a|s)=\mathbb{P}[A_t=a|S_t=s]$$

## Value Function

* Value function is a prediction of future reward **(expected future total reward)**

* Used to evaluate the goodness/badness of states

* And therefore to select between actions, e.g.

  $$v_\pi(s)=E_{\pi}[R_{t+1}+\gamma R_{t+2}+\gamma^2R_{t+3}+...|S_t=s]$$

## Model

build a model is not always required !!!

* A **model** predicts what the environment will do next

* **Transitions**: $\mathcal{P}$ predicts the next state. (state transition model)

* **Rewards**: $\mathcal{R}$ predicts the next (immediate) reward, e.g.

  $$\mathcal{P}_{ss\prime}^a=\mathbb{P}[S_{t+1}=s\prime|S_t=s,A_t=a]$$

  $$\mathcal{R}_s^a=\mathbb{E}[R_{t+1}|S_t=s, A_t=a]$$

## RL Agent Taxonomy

![rl agent taxonomy](\assets\images\post_images\强化学习1\rl agent taxonomy.PNG)

# Problems within RL

## Learning and Planning

* Reinforcement Learning
  * The environment is initially unknown
  * The agent interacts with the environment
  * The agent improves its policy
* Planning
  * A model of the environment is known
  * The agent performs computations with its model (without any external interaction)
  * The agent improves its policy


## Exploration and Exploitation Trade Off

**Exploration** finds more information about the environment
**Exploitation** exploits known information to maximise reward

## Prediction and Control

* Prediction: evaluate the future
  * Given a policy
* Control: optimise the future
  * Find the best policy