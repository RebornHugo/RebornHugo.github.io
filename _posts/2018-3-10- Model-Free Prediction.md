---
title: RL-5  Model-Free Prediction
categories:
 - Reinforcement Learning
tag:
 - RL

typora-copy-images-to: ..\assets\images\post_images\Model-Free Prediction
typora-root-url: ..
---
Model-free prediction是指

>  Estimate the value function of an unknown MDP

对于没有model的问题进行prediction（本节课不考虑control），主要介绍了Monte-Carlo学习算法，Temporal-Difference算法（TD(0)）以及其推广延伸的TD($\lambda$)，需要理解$TD(\lambda)$的前向和后向视角。

# Material

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/MC-TD.pdf)

[Video by David Silver](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/MC-TD.pdf)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf) Chapter5,6

# Monte-Carlo Learning

## Monte-Carlo Reinforcement Learning

* MC methods learn directly from episodes of experience
* MC is model-free: no knowledge of MDP transitions / rewards
* MC learns from complete episodes: no **bootstrapping**
* MC uses the simplest possible idea: value = **mean return**
* Caveat: can only apply MC to **episodic（片段的）** MDPs
  * All episodes must **terminate**

## Monte-Carlo Policy Evaluation

## ![52111595023](/assets/images/post_images/Model-Free Prediction/1521115950230.png)

## First-Visit Monte-Carlo Policy Evaluation

![52111657543](/assets/images/post_images/Model-Free Prediction/1521116575436.png)

## Every-Visit Monte-Carlo Policy Evaluation

![52111663300](/assets/images/post_images/Model-Free Prediction/1521116633005.png)

**NOTICE**

> 1. The first-visit MC method estimates vπ(s) as the average of the returns following **first** visits to s, whereas the every-visit MC method averages the returns following **all** visits to s.
> 2. **First**-visit MC has been **most** widely studied
> 3. Every-visit MC extends more naturally to function approximation and eligibility traces （看不懂233333）
>
> ——摘录自Sutton's <<RL>>

## Blackjack Example

![52111705634](/assets/images/post_images/Model-Free Prediction/1521117056343.png)

![52111708669](/assets/images/post_images/Model-Free Prediction/1521117086692.png)

## Incremental Monte-Carlo

The mean µ1, µ2, ... of a sequence x1, x2, ... can be computed **incrementally**,

![52111829062](/assets/images/post_images/Model-Free Prediction/1521118290621.png)

![52111879996](/assets/images/post_images/Model-Free Prediction/1521118799966.png)

# Temporal-Difference Learning

* TD methods learn directly from episodes of experience
* TD is model-free: no knowledge of MDP transitions / rewards
* TD learns from incomplete episodes, by bootstrapping
* TD updates a guess towards a guess

![52111909102](/assets/images/post_images/Model-Free Prediction/1521119091022.png)

## Driving Home Example

![52112152667](/assets/images/post_images/Model-Free Prediction/1521121526674.png)

## Advantages and Disadvantages of MC vs. TD

* TD can learn before knowing the final outcome
  * TD can learn online after every step
  * MC must wait until end of episode before return is known
* TD can learn without the final outcome
  * TD can learn from incomplete sequences
  * MC can only learn from complete sequences
  * TD works in continuing (non-terminating) environments
  * MC only works for episodic (terminating) environments

## Bias/Variance Trade-Off

![52112159713](/assets/images/post_images/Model-Free Prediction/1521121597130.png)

![52112166794](/assets/images/post_images/Model-Free Prediction/1521121667940.png)

## Batch MC and TD

![52181149467](/assets/images/post_images/Model-Free Prediction/1521811494674.png)

![52181157139](/assets/images/post_images/Model-Free Prediction/1521811571394.png)

## Unified View

![52181160011](/assets/images/post_images/Model-Free Prediction/1521811600112.png)

![52181161792](/assets/images/post_images/Model-Free Prediction/1521811617921.png)

![52181163193](/assets/images/post_images/Model-Free Prediction/1521811631930.png)

![52181165385](/assets/images/post_images/Model-Free Prediction/1521811653857.png)

> bootstrap是向前推导，自举。Sample是采样，采取action向前走一个step或者多个step直至一整个episode。

![52181176648](/assets/images/post_images/Model-Free Prediction/1521811766480.png)

# TD(λ)

## n-Step TD

使用n-step return 来作为TD target即可

![52181180896](/assets/images/post_images/Model-Free Prediction/1521811808968.png)

![52181188285](/assets/images/post_images/Model-Free Prediction/1521811882857.png)

## Forward View of TD(λ)

![52181195230](/assets/images/post_images/Model-Free Prediction/1521811952304.png)

![52181198505](/assets/images/post_images/Model-Free Prediction/1521811985059.png)

> 很容易推导每一个step的系数都加起来，最终的和为1

![52181212616](/assets/images/post_images/Model-Free Prediction/1521812126162.png)

## Backward View TD(λ)

* Forward view provides theory
* Backward view provides mechanism
* Update online, every step, from incomplete sequences

### Eligibility Traces

![52181218232](/assets/images/post_images/Model-Free Prediction/1521812182328.png)

这是一个较难理解的概念，我的理解是对每个state都要分配credit或者说是weight，对应到公示上就是一个系数$E_t(s)$，在这里用一个table来存储（在下一节function approximation里面可以用nn）。这里的Eligibility Traces的计算规则很简单，如上图所示，含义是对recency和frequency的state分配更多的credit。

![52241736262](/assets/images/post_images/Model-Free Prediction/1522417362622.png)

## Relationship Between Forward and Backward TD

当$\lambda$为0时是TD(0), $\lambda$为1时是MC

![52181245977](/assets/images/post_images/Model-Free Prediction/1521812459777.png)

![52181247238](/assets/images/post_images/Model-Free Prediction/1521812472380.png)

![52181249737](/assets/images/post_images/Model-Free Prediction/1521812497370.png)