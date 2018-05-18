---
title: RL-9  Connections Between Inference and Control
categories:
 - Reinforcement Learning
tag:
 - RL,DRL

typora-copy-images-to: ..\assets\images\post_images\Connections Between Inference and Control
typora-root-url: ..
---

用概率图的观点建立了优化控制与强化学习（Q-learning，policy gradient）之间联系，也可以解释人类动作，并且其中的inference部分是逆强化学习(Inverse Reinforcement Learning)的基础。

# Material

[PDF_FROM_BERKELY](http://rll.berkeley.edu/deeprlcourse/f17docs/lecture_11_control_and_inference.pdf)

[VODIE_BY_LEVIN](https://www.bilibili.com/video/av20957290/?p=12)

# About Human Behavior

问题的出发点是关于人类/expert的行为，如果我们构建一个model来解释它，那么仍然会使用过去的方法，maximize expected future cumulative discounted reward。

![1526625070706](/assets/images/post_images/Connections Between Inference and Control/1526625070706.png)

但是我们会发现，人类/猴子/专家的策略在很多情况下并非是最优的，而是近似于最优。下面这个例子，猴子只要最后拿到了橘子就行，过程中会有少许弯路，但是这并不重要。

![1526625223761](/assets/images/post_images/Connections Between Inference and Control/1526625223761.png)

这说明了以下三点：

* some mistakes matter more than others!
* behavior is **stochastic** 
* but good behavior is still the most likely 

因此我们需要引入概率图模型(A probabilistic graphical model of decision making )来解释人类的行为。

# A probabilistic graphical model of decision making 

## probabilistic graphical model

![1526625345029](/assets/images/post_images/Connections Between Inference and Control/1526625345029.png)

带来的好处：

* Can model suboptimal behavior (important for inverse RL) 
* Can apply inference algorithms to solve control and planning problems
* Provides an explanation for why stochastic behavior might be preferred (useful for exploration and transfer learning) 

上图中：
$$
\mathcal{O_t}是新定义的optimality变量，是可以被观测的。仅有1和0两种取值。含义是在s_t与a_t的条件下，人是否想要获得奖励。
$$

## Inference

![1526626096713](/assets/images/post_images/Connections Between Inference and Control/1526626096713.png)

这里规定了
$$
p(\mathcal{O_t}|s_t,a_t)\propto exp(r(s_t, a_t))
$$
后面会对其解释。这里有三点很重要：**backward message, policy, forward message**。

## Backward messages 

指当前状态当前动作下，未来的Optimality都为1的概率。