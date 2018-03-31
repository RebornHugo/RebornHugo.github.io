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

[PDF](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/MC-TD.pdf)

[Video by David Silver](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/MC-TD.pdf)

[<<An Introduction to Reinforcement Learning, Sutton and Barto, 1998>>](http://incompleteideas.net/book/bookdraft2017nov5.pdf) 

# Introduction

## Large-Scale Reinforcement Learning

Reinforcement learning can be used to solve large problems, e.g.

* Backgammon: 1020 states
* Computer Go: 10170 states
* Helicopter: continuous state space

## Value Function Approximation

![52246658675](/assets/images/post_images/Value Function Approximation/1522466586753.png)