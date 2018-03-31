---
title: RL-7  Value Function Approximation
categories:
 - Reinforcement Learning
tag:
 - RL

typora-copy-images-to: ..\assets\images\post_images\Value Function Approximation
typora-root-url: ..
---

Function Approximation 的目的是用类似于linear combination或者nn的方法来构造一个计算Q值的funtion。因为许多问题的state和action是无限大的或者是continuous的，无法用一个简单的Q tabel来描述。本节重点DQN。


# Material

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/FA.pdf)

[Video by David Silver](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/MC-TD.pdf)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf) 

[Medium](https://medium.com/@awjuliani/simple-reinforcement-learning-with-tensorflow-part-4-deep-q-networks-and-beyond-8438a3e2b8df)

# Introduction

## Large-Scale Reinforcement Learning

Reinforcement learning can be used to solve large problems, e.g.

* Backgammon: 1020 states
* Computer Go: 10170 states
* Helicopter: continuous state space

## Value Function Approximation

![52246658675](/assets/images/post_images/Value Function Approximation/1522466586753.png)

![52246667603](/assets/images/post_images/Value Function Approximation/1522466676039.png)

# Incremental Methods

## Gradient Descent

![52246809855](/assets/images/post_images/Value Function Approximation/1522468098559.png)

梯度下降，MLDL的基础

## SGD

![52246820310](/assets/images/post_images/Value Function Approximation/1522468203109.png)

## Linear Function Approximation

### Feature Vectors

特征向量，描述一个state。在OpenAI Gym库中，一般用env.step()后返回的observation来表示。在Atari game中，用frame来表示。

![52246840781](/assets/images/post_images/Value Function Approximation/1522468407810.png)

![52246843723](/assets/images/post_images/Value Function Approximation/1522468437237.png)

### Table Lookup Features

类似于Q learning 或者sarsa里面的q table，下图说明当linear combination与table lookup的关系，此处的feature table是一个identical matrix。

![52246953166](/assets/images/post_images/Value Function Approximation/1522469531669.png)

### Incremental Prediction Algorithms

![52247662006](/assets/images/post_images/Value Function Approximation/1522476620065.png)

后面讲的是使用MC TD(0) TD($\lambda$)计算q target，来作为superviser，或者是david 所说的oracle。

![52247737950](/assets/images/post_images/Value Function Approximation/1522477379507.png)

![52247739702](/assets/images/post_images/Value Function Approximation/1522477397024.png)

![52247742107](/assets/images/post_images/Value Function Approximation/1522477421075.png)

### Incremental Control Algorithms

![52248025035](/assets/images/post_images/Value Function Approximation/1522480250350.png)

这里的value function换为了action value funciton

![52248027227](/assets/images/post_images/Value Function Approximation/1522480272274.png)

这里的feature vector 表示的是某个state下，采取了特定的action。

![52248036379](/assets/images/post_images/Value Function Approximation/1522480363796.png)

![52248046235](/assets/images/post_images/Value Function Approximation/1522480462357.png)

### Convergence

![52248140665](/assets/images/post_images/Value Function Approximation/1522481406651.png)

上图说明很多情况下并不能convergence

# Batch Methods

* Gradient descent is simple and appealing
* But it is not sample efficient
* Batch methods seek to find the best fitting value function
* Given the agent’s experience (“training data”)

要做到batch methods，就需要一个“经验池”，定义好objective function后，依然可以使用gradient descent求解。

## Least Squares Prediction

![52248164588](/assets/images/post_images/Value Function Approximation/1522481645886.png)

这里的objective function可以采用least squares，然后使用SGD和experience replay来求解。

![52248170501](/assets/images/post_images/Value Function Approximation/1522481705019.png)

### DQN

![52248178124](/assets/images/post_images/Value Function Approximation/1522481781247.png)

这里是DQN的具体流程，DQN的关键在于experience replay与fixed Q-targets。计算Q-target的nn的参数w，需要在经过一定的episode之后再assign为Q-eval的参数。

* 以下摘自Arthur Juliani's [medium](https://medium.com/@awjuliani/simple-reinforcement-learning-with-tensorflow-part-4-deep-q-networks-and-beyond-8438a3e2b8df)

>  Experience Replay 的好处：
>
> The basic idea is that by storing an agent’s experiences, and then randomly drawing batches of them to train the network, we can more robustly learn to perform well in the task.
>
> Separate Target Network 的原因：
>
>  The issue is that at every step of training, the Q-network’s values shift, and if we are using a constantly shifting set of values to adjust our network values, then the value estimations can easily spiral out of control. 

### DQN in Atari

![52248231110](/assets/images/post_images/Value Function Approximation/1522482311105.png)

端到端的学习，Q net的输出为vector，每一维对应一个action的Q(S,A)；输入为画面帧。

### Linear Least Squares Prediction (2)

![52248355188](/assets/images/post_images/Value Function Approximation/1522483551883.png)

上图提供了一种暴力做法，可以直接解简单的linear combination。

## Least Squares Control

使用LSPI

![52248386752](/assets/images/post_images/Value Function Approximation/1522483867520.png)