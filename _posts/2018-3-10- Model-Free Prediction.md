---
title: RL-5  Model-Free Prediction
categories:
 - Reinforcement Learning
tag:
 - RL

typora-copy-images-to: ..\assets\images\post_images\Model-Free Prediction
typora-root-url: ..
---
Model Free 是指

> Estimate the value function of an unknown MDP

对于没有model的问题进行prediction（本节课不考虑control），主要介绍了Monte-Carlo学习算法，Temporal-Difference算法（TD(0)）以及其推广延伸的TD($\lambda$)

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