---
title: RL-8  Policy Gradient Methods
categories:
 - Reinforcement Learning
tag:
 - RL

typora-copy-images-to: ..\assets\images\post_images\Policy Gradient Methods
typora-root-url: ..
---

Policy Gradient Methods是Policy Based的方法，与value based方法（如q learning ，sarsa($\lambda$)等）的区别在于这类方法不需要计算value function，而是直接对policy进行optimize。注意，两类方法都是model-free RL method。本节重点，对policy gradient的理解，actor critic以及A3C。


# Material

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/pg.pdf)

[Video by David Silver](https://www.youtube.com/watch?v=JVIr_2hnZbo&index=7&list=PLjSwXXbVlK6K2enbNmPGjnmB8QBRgCv5s)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf) 

[Medium](https://medium.com/@awjuliani/simple-reinforcement-learning-with-tensorflow-part-4-deep-q-networks-and-beyond-8438a3e2b8df)

# Introduction

![52257463695](/assets/images/post_images/Policy Gradient Methods/1522574636954.png)

![52257536014](/assets/images/post_images/Policy Gradient Methods/1522575360144.png)

## Advantages of Policy-Based RL

* Advantages:

  * Better convergence properties

  * Effective in high-dimensional or continuous action spaces

    > 因为value based method比如q learning或者sarsa中，必定会有计算max的一步操作，求max在连续空间或高维度空间几乎无法解决。

  * Can learn stochastic policies

    > value based method 中，通过policy improvment得到的policy都是deterministic policy，然后遇到了有些问题无法解决。比如石头剪刀布（Rock-Paper-Scissors）。

* Disadvantages:

  * Typically converge to a local rather than global optimum
  * Evaluating a policy is typically inefficient and high variance

## Example: Aliased Gridworld 

下图中，gray grid无法被区分，如果使用deterministic policy，则会陷入stuck。

![52258056670](/assets/images/post_images/Policy Gradient Methods/1522580566707.png)

但是如果使用stochastic policy则有机会跳出并找到optimality solution。

![52258401581](/assets/images/post_images/Policy Gradient Methods/1522584015819.png)

## Policy Search

![52258431878](/assets/images/post_images/Policy Gradient Methods/1522584318783.png)

> 上图第三个公式很重要

可以使用无梯度算法（遗传算法等）或者梯度算法（梯度下降、拟牛顿等）来求解。

# Finite Difference Policy Gradient

找到了objective function可以使用gradient ascend求解。

![52258461016](/assets/images/post_images/Policy Gradient Methods/1522584610169.png)

通过perturb 不同维度的$\theta$ a little 然后try it again 理论上的确可以找到maximize objective function的$\theta$，但是计算量会很大。

![52258489019](/assets/images/post_images/Policy Gradient Methods/1522584890194.png)

# Monte-Carlo Policy Gradient

## Likelihood Ratios

score function是一个较难理解的概念，对policy关于$\theta$求梯度可以得到新的式子，有两个term组成，左边是policy本身，右项是score function。

![52258688938](/assets/images/post_images/Policy Gradient Methods/1522586889381.png)

demo

![52258719021](/assets/images/post_images/Policy Gradient Methods/1522587190219.png)

> 总softmax policy中，对feature进行linear combination，然后取exponential得到的值与采取action概率成正比。
>
> 这里score function是非常intuitive的。如果某个action的feature值比平常（average）要大，则score function要大，则根据gradient ascend，policy会偏向使这个action出现的概率高的方向增长。

![52258800293](/assets/images/post_images/Policy Gradient Methods/1522588002936.png)

> gaussian policy 在continuous的场景下很常见。Intuition与softmax policy类似。

## Policy Gradient Theorem

![52258846455](/assets/images/post_images/Policy Gradient Methods/1522588464557.png)

在one-step MDPs中，关于J求梯度结果在上图。推广到multi-step MDPs后结果如下：

![52258862610](/assets/images/post_images/Policy Gradient Methods/1522588626103.png)

如果使用从一个state开始，完成整个episode后得到return来估计这里的Q值，则得到了Monte-Carlo Policy Gradient (REINFORCE)算法：

![52258872611](/assets/images/post_images/Policy Gradient Methods/1522588726113.png)

## Actor-Critic Policy Gradient

但是reinforce算法会有极大的variance，如果使用Critic来减少variance效果提升。这里的critic是使用policy evaluation去估计action-value function。

![52258886083](/assets/images/post_images/Policy Gradient Methods/1522588860834.png)

使用另一个q-table 或者 nn得到的action-value来近似这里的$Q^{\pi_\theta}(s,a)$

> 这里actor-critic的概念非常像generative adversarial nerword里的generator 和 discriminator。

* The critic is solving a familiar problem: **policy evaluation**
* How good is policy $π_\theta$ for current parameters θ?
* This problem was explored in previous two lectures, e.g.
  * Monte-Carlo policy evaluation
  * Temporal-Difference learning
  * TD(λ)
* Could also use e.g. least-squares policy evaluation


下图是使用linear TD(0) 计算critic的算法。

![52258941035](/assets/images/post_images/Policy Gradient Methods/1522589410351.png)

> 过去的value based method的做法中，没有gradient ascend来提高policy的objective function值的这一步，而是使用$\epsilon-greddy$来进行policy improvment，这里的policy是implicit的。
>
> actor-critc中，由$\theta$进行衡量的policy $\pi_{\theta}$则是explicit的。在对$\theta$进行gradient ascend时，learning rate或者说是study step $\alpha$ 决定了policy 的变化速率。特别的，当$\alpha$ 趋近于$+\infin$时，这里的policy improvment就达到了greedy的效果。
>
> 另外，以上方法容易陷入local optima。

## Advantage Function Critic

![52267781009](/assets/images/post_images/Policy Gradient Methods/1522677810099.png)

> policy gradient $\triangledown_{\theta} J(\theta)$ 减去baseline function 可以 reduce variance without change expectation

上图第一个式子成立的前提是baseline function只是关于state而不关于action。

特别的，这里的baseline选取为state value function效果很好，并且定义advantage function = action value function - state value function。

> dual dqn 里面也提到了adavantage的概念。将action value funciton拆成了两个term。

![52267887189](/assets/images/post_images/Policy Gradient Methods/1522678871891.png)

这里计算advantage function的方法有两种，最容易想到的一种是使用两套parameters同时estimate V和Q，但这样显得即为complex。可以采用TD error ($\delta^{\pi_{\theta}}$)来作为sampe of advantage function。

![1522759220941](/assets/images/post_images/Policy Gradient Methods/1522759220941.png)

> TD error is an unbias sample of advantage function
>
> 使用TD error来estimate的方法可以只计算State value function V而不用计算action value function Q。

## Eligibility Traces

![1522761067296](/assets/images/post_images/Policy Gradient Methods/1522761067296.png)

![1522761080954](/assets/images/post_images/Policy Gradient Methods/1522761080954.png)

注意上图中policy gradient与不减去**baseline** function的区别！！！（最终值不变，但variance会减小）

后两个公式中的$V_v(s_{t})$是baseline function，左边用红字描述的是$Q^{\pi_\theta}(s,a)$ 的估计值，分别用**complete return**和 **one-step TD target** 来估计。

![1522762348635](/assets/images/post_images/Policy Gradient Methods/1522762348635.png)

