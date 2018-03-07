---
title: RL-3 Planning by Dynamic Programming
categories:
 - Reinforcement Learning
tag:
 - ML,RL

typora-copy-images-to: ..\assets\images\post_images\强化学习3
typora-root-url: ..
---
本节主要讲了动态规划。需要重点理解**Policy Iteration**（evaluation 和 improvement两部分）和**Value Iteration**两类经典算法的原理异同，然后讲了asynchronous backups。

# Material

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/DP.pdf)

[Video by David Silver](https://www.bilibili.com/video/av9930653/)

[Video by Siraj](https://www.youtube.com/watch?v=5R2vErZn0yw&index=2&list=PL2-dafEMk2A5FZ-MnPMpp3PBtZcINKwLA)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf) 第四章

[GridWorld**小游戏**](https://cs.stanford.edu/people/karpathy/reinforcejs/gridworld_dp.html)  by karpathy

# Introduction

## What is Dynamic Programming?

*Dynamic* sequential or temporal component to the problem

*Programming* optimising a “program”, i.e. a policy

## Requirements for Dynamic Programming

![1520256251454](/assets/images/post_images/强化学习3/1520256251454.png)

## Planning by Dynamic Programming

Dynamic programming assumes full knowledge of the MDP
It is used for ***planning*** in an MDP

> PLANNING
>
> - A model of the environment is known
> - The agent performs computations with its model (without any external interaction)
> - The agent improves its policy

可用于prediction和control，区别在于是否对policy$\pi$进行optimize

# Policy Evaluation（prediction）

![1520338645843](/assets/images/post_images/强化学习3/1520338645843.png)

![1520338712432](/assets/images/post_images/强化学习3/1520338712432.png)

david课上以及Sutton的书上都有一个gridworld的demo。karpathy的网站上有一个用js写的大格子版本的游戏。

# Policy Iteration（control）

## How to Improve a Policy（贪心法）

![1520339161131](/assets/images/post_images/强化学习3/1520339161131.png)

## Policy Iteration

![1520339333337](/assets/images/post_images/强化学习3/1520339333337.png)

david课上以及Sutton的书上都有一个Jack’s Car Rental的example。

## Policy Improvement（含证明）

![1520339482788](C:\Users\51163\AppData\Local\Temp\1520339482788.png)

![1520339633167](/assets/images/post_images/强化学习3/1520339633167.png)

## Modified Policy Iteration

![1520340407198](/assets/images/post_images/强化学习3/1520340407198.png)

# Value Iteration

## Value Iteration in MDPs

### Principle of Optimality

Any optimal policy can be subdivided into two components:

* An optimal first action A∗
* Followed by an optimal policy from successor state $S\prime$

![1520340513202](/assets/images/post_images/强化学习3/1520340513202.png)

![1520340620924](/assets/images/post_images/强化学习3/1520340620924.png)

![1520340703871](/assets/images/post_images/强化学习3/1520340703871.png)

### Summary of DP Algorithms

![1520342558131](/assets/images/post_images/强化学习3/1520342558131.png)

# Difference between Policy Iteration and Value Iteration(重点理解)

来自[stackoverflow](https://stackoverflow.com/questions/37370015/what-is-the-difference-between-value-iteration-and-policy-iteration)

![img](https://i.stack.imgur.com/wGuj5.png)

Key points:

1. **Policy iteration** includes: **policy evaluation** + **policy improvement**, and the two are repeated iteratively until policy converges.
2. **Value iteration** includes: **finding optimal value function** + one **policy extraction**. There is no repeat of the two because once the value function is optimal, then the policy out of it should also be optimal (i.e. converged).
3. **Finding optimal value function** can also be seen as a combination of policy improvement (due to **max**) and truncated policy evaluation (the reassignment of $v(s)$ after just one sweep of all states regardless of convergence).
4. The algorithms for **policy evaluation** and **finding optimal value function** are highly similar except for a max operation (as highlighted)
5. Similarly, the key step to **policy improvement** and **policy extraction** are identical except the former involves a stability check.

*Policy iteration* is faster than *value iteration*, as a policy converges more quickly than a value function.

# Extensions to Dynamic Programming

![1520342589950](/assets/images/post_images/强化学习3/1520342589950.png)