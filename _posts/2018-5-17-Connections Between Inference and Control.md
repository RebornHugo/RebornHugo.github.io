---
titQ-learning with soft optimalityle: RL-9  Connections Between Inference and Control
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

# Inference with A probabilistic graphical model of decision making 

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
上上张ppt有推导过程。这里有三点很重要：**backward message, policy, forward message**。

## Backward messages 

### computation

指当前状态当前动作下，未来的Optimality都为1的概率。计算过程如下：

![1526628809975](/assets/images/post_images/Connections Between Inference and Control/1526628809975.png)

注意这里两点：

1. $$
   \beta_T(s_T,a_t)=p(\mathcal{O}_{T}|s_T,a_T)
   $$

2. $$
   p(\mathcal{a}_{t+1}|s_{t+1})被称为priori，因为它不取决于optimality。可以设定为uniform distribution。
   $$





### connection with value iteration

![1526643212644](/assets/images/post_images/Connections Between Inference and Control/1526643212644.png)

这里对V和Q重新定义，得到V,Q之间的关系。并且，当Q越来越大，得到V等于Q，这也被称为一种softmax（不是DL里面的），指softening of the max operator。然而
$$
V_t(s_t)->max_aQ_t(s_t,a_t)
$$
这种形式很容易让人联想到value iteration。

![1526646140282](/assets/images/post_images/Connections Between Inference and Control/1526646140282.png)

参见[RL_3](https://rebornhugo.github.io/reinforcement%20learning/2018/03/04/Planning-by-Dynamic-Programming/#value-iteration)。

但是这里先去exponential再取log的做法会带来过于乐观（optimistic transition）的问题。但是deterministic transition下这不是问题，但是在stochastic transition下会带来严重的问题。

### Summary

如下图：

![1526646521270](/assets/images/post_images/Connections Between Inference and Control/1526646521270.png)

从$$\beta_T$$开始递归到$$\beta_1$$来计算backward message。并且$$\beta_t$$ is "Q-function-like"。

当action prior不为uniform distribution时，如下图：

![1526647759608](/assets/images/post_images/Connections Between Inference and Control/1526647759608.png)

## Policy computation 

计算policy，使用贝叶斯公式可以得到下图中的公式：

![1526648056249](/assets/images/post_images/Connections Between Inference and Control/1526648056249.png)

### policy computer with value functions

![1526648210420](/assets/images/post_images/Connections Between Inference and Control/1526648210420.png)

使用backward message 计算 value function，其中policy的值是exponential of advantage function, 这个结果是很符合直觉的。如果某一个action对应的advantage较大，则其在policy中的取值概率也应该较大。

### Summary

![1526648684081](/assets/images/post_images/Connections Between Inference and Control/1526648684081.png)

* Natural interpretation: better actions are more probable 
* Random tie-breaking
* Analogous to Boltzmann exploration 
* Approaches greedy policy as temperature decreases 

## Forward messages

> 与下一讲的Inverse Reinforcement Learning右较强的联系

### derivation

### ![1526649774382](/assets/images/post_images/Connections Between Inference and Control/1526649774382.png)

### Forward/backward message intersection 

![1526650367425](/assets/images/post_images/Connections Between Inference and Control/1526650367425.png)

## Summay	

1.  Probabilistic graphical model for optimal control 

   ![1526650616049](/assets/images/post_images/Connections Between Inference and Control/1526650616049.png)

2. . Control = inference (similar to HMM, EKF, etc.) 

3. Very similar to dynamic programming, value iteration, etc. (but “soft”) 

# Algorithm with Soft Optimality

## Q-learning with soft optimality 

使用softening max 代替 normal max来求下一个state下的max value function。

并且计算policy的方式变为对advantage funtion取exponential。

![1526717351798](/assets/images/post_images/Connections Between Inference and Control/1526717351798.png)

## Policy gradient with soft optimality 

$$
\pi(a|s)=exp(Q_\phi(s,a)-V(s))
$$

optimizes
$$
\sum_tE_{\pi(s_t,a_t)}[r(s_t,a_t)]+E_{\pi(s_t)}[\mathcal{H}(\pi(a_t|s_t))]
$$
即使用policy gradient with soft optimality所要提升的目标除了expected future total reward之外，还有一项policy entropy作为正则项。其意义是防止policy提早崩塌为deterministic policy。

![1526719788682](/assets/images/post_images/Connections Between Inference and Control/1526719788682.png)

参见两篇paper

[Equivalence Between Policy Gradients and Soft Q-Learning](https://arxiv.org/abs/1704.06440) 

[Bridging the Gap Between Value and Policy Based Reinforcement Learning](https://arxiv.org/abs/1702.08892)

说明policy gradient with soft optimality 与Q learning with soft optimality之间联系十分紧密，以下是推导过程： 

![1526721615691](/assets/images/post_images/Connections Between Inference and Control/1526721615691.png)

可以发现，policy gradient的计算形式与Q-learning是即为相似的，前者是gradient ascent所以是加一个(正)值，后者是gradient descent所以是减去一个(负)值。

## Benefits of soft optimality 

* Improve exploration and prevent entropy collapse 
* Easier to specialize (finetune) policies for more specific tasks 
* Principled approach to break ties 
* Better robustness (due to wider coverage of states) 
* Can reduce to hard optimality as reward magnitude increases 
* Good model for modeling human behavior (more on this later inverse reinforcement learnig) 