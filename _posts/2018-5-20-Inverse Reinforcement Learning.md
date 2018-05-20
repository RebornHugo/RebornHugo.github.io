---
title: RL-10  Inverse Reinforcement Learning
categories:
 - Reinforcement Learning
tag:
 - RL,DRL

typora-copy-images-to: ..\assets\images\post_images\Inverse Reinforcement Learning
typora-root-url: ..
---

在上一节的基础上，引入逆强化学习，用于解决无法直接获取reward的情况，并且为后面的exploration与transfer learning做铺垫。

# Material

[Video from Berkeley](https://www.bilibili.com/video/av20957290/index_12.html?t=2058#page=13)

[PDF by Levin](http://rll.berkeley.edu/deeprlcourse/f17docs/lecture_12_irl.pdf)

# Introduction

## Where does the reward function come from? 

像Atari game之类的游戏，可以使用得分作为reward，但是在实际生活中的task是没有reward的，比如机器人倒水的task中机器人甚至不知道是否自己倒进了杯子。chat-bot中机器人也没有直接的reward，自动驾驶问题中，有很多人类implicitly的知识难以直接定义reward，个人认为是因 为机器缺少人类所拥有的一些**先验知识**。过去的做法是直接使用hand engineered reward，这样设计reward不仅麻烦而且也很难表示人类的目的。

![1526799913479](/assets/images/post_images/Inverse Reinforcement Learning/1526799913479.png)

但是，这一类问题有一些共同点：很容易提供expert demonstration，所以可以采用Inverse Reinforcement Learning来infer出针对于某类task的reward function。

## Why should we learn the reward? 

在imitation learning中，可以直接让agent进行clone expert的behaviour。

Alternative: directly mimic the expert (behavior cloning) 

- simply “ape” the expert’s motions/actions
- doesn’t necessarily capture the salient parts of the behavior
- what if the expert has **different capabilities**? 

> 适用于观测数据很多，且域漂移 (domain shift) 比较小的情况 

1. 但这样做agent并不能理解环境，不知道objective。而且还会clone到一些错误的行为。所以我们需要reward，这样才能让agent理解环境并学到合适的policy。
2. robot和expert可能有不同的capability，所以他们完成同一个task的最优策略很可能是不同的，blind mimic无法学到最优策略。如果agent直到了goal/objective，则可以学到optimal policy。

以下是非常形象的例子：

1.![1526801042046](/assets/images/post_images/Inverse Reinforcement Learning/1526801042046.png)这里的小孩(robot)在观察了大人(expert)的动作（尝试将书放入柜子却因为被门挡住而无法成功）后，学会了打开柜子的动作。这是个inference problem而不是imitation problem。

2.

![1526801336198](/assets/images/post_images/Inverse Reinforcement Learning/1526801336198.png)

这里robot帮助expert捡起了东西让expert将它放在衣服上。注意这里的大人小孩有着不同的**capability**（小孩甚至摸不到高处的衣服），但小孩直到如果自己捡起物品并举起手就可以解决这个问题。

## Challenges

Inverse Optimal Control(IOC) / Inverse Reinforcement Learning(IRL)

infer reward function from demonstrations

given:

* state & action space
* samples from π*
* dynamics model (sometimes) 

goal:

* recover reward function
* then use reward to get policy

Challenges:

* **underdefined** problem

  ![1526816213198](/assets/images/post_images/Inverse Reinforcement Learning/1526816213198.png)

  > 缺少先验知识，无法解释circle为什么会朝着右边的triangle移动。
  >
  > **Ambiguity**  -> Many Ambiguous Answer！

* difficult to evaluate a learned reward

  > IRL会有着improve reward function的架构。但是在evaluate reward function的时候会需要gradient之类的东西，这就需要solve the corresponding forward reinforcement learning problem，然后反复迭代。

* demonstrations may **not be precisely optimal** 

# How to Solve

## Problem Definition

![1526816906233](/assets/images/post_images/Inverse Reinforcement Learning/1526816906233.png)

## Feature matching IRL

传统方法，与如今的方法大有不同。核心思想是假设feature **f**真正important的，那就让学习到的policy与expert policy拥有着相同的feature f的期望。但是这样做仍然存在ambiguous的问题，因为我们会有无数种可能解，就跟SVM里面超平面的选取有无数种可能一样。那么可以采用SVM里面的trick

![1526818349143](/assets/images/post_images/Inverse Reinforcement Learning/1526818349143.png)

这里的
$$
\pi^{\star}\in\prod
$$
带来的问题是最终的margin就会为0。于是有

![1526819094184](/assets/images/post_images/Inverse Reinforcement Learning/1526819094184.png)

## Optimal Control as a Model of Human Behavior 

使用[上节课](https://rebornhugo.github.io/reinforcement%20learning/2018/05/17/Connections-Between-Inference-and-Control/)中学习的概率图模型而不是直接求解最大的expected future cumulative discounted reward。

A probabilistic graphical model of decision making 

![1526822436140](/assets/images/post_images/Inverse Reinforcement Learning/1526822436140.png)

