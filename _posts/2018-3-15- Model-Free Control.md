---
title: RL-6  Model-Free Control
categories:
 - Reinforcement Learning
tag:
 - RL

typora-copy-images-to: ..\assets\images\post_images\Model-Free Control
typora-root-url: ..
---

Model-free control 是指

> Optimise the value function of an unknown MDP

本节与model free predict 衔接，重点GLIE Monte-Carlo Control，SARSA, SARSA$(\lambda)$, Q Learning

# Intro

## Uses of Model-Free Control

For most of these problems, either:

>  MDP model is unknown, but **experience** can be sampled
>
> MDP model is known, but is too big to use, except by samples

**Model-free contro**l can solve these problems

## On and Off-Policy Learning

![52180050214](/assets/images/post_images/Model-Free Control/1521800502145.png)

> 二者本质区别在于实际take action的**behaviour** policy $\mu$与进行Evaluate 计算$v_{\pi}(s) 与q_{\pi}(s,a)$的 **target** policy π 是否是同一个policy。像q-learning这种off policy方法进行evaluate的next action并不一定是真正采取的next action。

# Material

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/control.pdf)

[Video by David Silver](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/MC-TD.pdf)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf) Chapter5,6

#  On-Policy Monte-Carlo Control

![52180087076](/assets/images/post_images/Model-Free Control/1521800870762.png)

这是dp一节讲的policy iteration过程，MC control在此基础上，将state value function 改进为 action value function, pure greedy 改进为 $\epsilon-greedy$


## Using Action-Value Function

![52180862989](/assets/images/post_images/Model-Free Control/1521808629890.png)

因为当前是model free的环境，故不知道env的transition dynamics $\mathcal{P}^a_{ss\prime}$

所以无法用上图中的第一个公式进行policy improvment，但是action value function Q(s, a)是可求的，故使用Q进行policy improvment。

## $\epsilon-greedy$  exploration

不用naive greedy很容易理解，是为了进行explore，增添发现optimality的概率。

![52180852337](/assets/images/post_images/Model-Free Control/1521808523379.png)

下图证明了$\epsilon-greedy$ 的确可以进行improvment

![52180918970](/assets/images/post_images/Model-Free Control/1521809189706.png)

![52180991207](/assets/images/post_images/Model-Free Control/1521809912072.png)

> always to act greedily with respect to the freshest most recent estimate value funtion !!! 一个episode后就可以直接update value function slightly better, 而不必去使用 old estimate of the value function to generate behavior

## GLIE

![52181024408](/assets/images/post_images/Model-Free Control/1521810244089.png)

> GLIE的目的是guarantee最终的policy是best possible policy （$\pi_\star$）
>
> 因此需要balance两样:
>
> * explore to make sure we don't exclude things which are better
> * 当最终获得optimal policy 后，停止 explore 



![52181061891](/assets/images/post_images/Model-Free Control/1521810618915.png)

# On-Policy Temporal-Difference Learning

* Temporal-difference (TD) learning has several advantages over Monte-Carlo (MC)

  * Lower variance
  * Online
  * Incomplete sequences

* Natural idea: use TD instead of MC in our control loop

  * Apply TD to Q(S, A)
  * Use $\epsilon$-greedy policy improvement
  * Update every time-step

## Sarsa(λ)

![52181083957](/assets/images/post_images/Model-Free Control/1521810839574.png)

> S: state A: action R: reward S: next state A: next action
>
> 类比TD(0)

![52181095227](/assets/images/post_images/Model-Free Control/1521810952278.png)

![52181097927](/assets/images/post_images/Model-Free Control/1521810979278.png)

![52181113812](/assets/images/post_images/Model-Free Control/1521811138120.png)

> stochastic approximation theory

#### n-Step Sarsa

![52181122765](/assets/images/post_images/Model-Free Control/1521811227650.png)

>  类比n-Step TD, 使用n-step Q-return来作为target即可

#### Forward View Sarsa(λ)

![52241625680](/assets/images/post_images/Model-Free Control/1522416256804.png)

> 对每一个$q_t^{(n)}$的系数weight累加，和为1（等比数列）。

#### Backward View Sarsa(λ)

![52241636492](/assets/images/post_images/Model-Free Control/1522416364923.png)

> eligibility traces，与上一节相同。维护一个eligibility table E, 对recency和frequency的state分配更多的credit。

![52241661207](/assets/images/post_images/Model-Free Control/1522416612074.png)

![52241663207](/assets/images/post_images/Model-Free Control/1522416632073.png)

这个gridworld游戏展示了使用standard sarsa与sarsa($\lambda$)的区别。本例中只在最后一个step获得reward，如果使用one-step Sarsa，则TD error只会往前propagate一步使最后一步的Q值增大（体现在二图中仅一个箭头加深颜色）。如果使用Sarsa($\lambda$)，则会根据E table（eligibility trace）对所有state的action value均进行update。显然根据E的公式，早期的step由于不够recency因而Q值较小，所以箭头颜色较浅。

# Off-Policy Learning

![52241830636](/assets/images/post_images/Model-Free Control/1522418306363.png)

## Importance Sampling

![52246360857](/assets/images/post_images/Model-Free Control/1522463608573.png)

![52246372376](/assets/images/post_images/Model-Free Control/1522463723761.png)

## Q-Learning

![52246383474](/assets/images/post_images/Model-Free Control/1522463834742.png)

> behaviour policy与 update Q值的target policy不同。

![52246398520](/assets/images/post_images/Model-Free Control/1522463985200.png)

![52246400742](/assets/images/post_images/Model-Free Control/1522464007425.png)

这里的$Q(S\prime， a\prime)$计算用到了bellman optimality Equation

![52246422706](/assets/images/post_images/Model-Free Control/1522464227060.png)

![52246461697](/assets/images/post_images/Model-Free Control/1522464616978.png)

![52246463661](/assets/images/post_images/Model-Free Control/1522464636613.png)

我在阅读[Deep Reinforcement Learning with Double Q-learning](https://arxiv.org/abs/1509.06461)一文时，了解到Q-leanring 或者DQN算法，由于使用了在对q_target 取了max，会带来bias。而且反复的取max会导致overestimating，并且无法恢复。

