---
title: RL-12  transfer learning
categories:
 - Reinforcement Learning
tag:
 - RL
 - DRL

typora-copy-images-to: ..\assets\images\post_images\transfer learning
typora-root-url: ..
---

1. The benefits of sharing knowledge across tasks 
2. The transfer learning problem in RL 
3. Transfer learning with source and target domains
4. Next time: multi-task learning, meta-learning

* Goal:

  * Understand how reinforcement learning algorithms can benefit from structure learned on prior tasks 

  * Understand a few transfer learning methods

> transfer learning in RL 是一个open question，并不是一个solved problem。这节课重在理解各种新兴方法的pros and cons

# Material

[Video_transfer_learning_in_CS294-112](https://www.bilibili.com/video/av20957290/index_16.html?t=2108#page=16)

[PDF_by_Levin](http://rail.eecs.berkeley.edu/deeprlcourse-fa17/f17docs/lecture_14_transfer.pdf)

[Video_multi-task_learning_in_CS294-112](https://www.bilibili.com/video/av20957290/index_16.html?t=2108#page=17)

[PDF_by_Levin](http://rail.eecs.berkeley.edu/deeprlcourse-fa17/f17docs/lecture_15_multi_task_learning.pdf)

# Introduction

## Motivation

回忆，exploration那一节中讲到的Montezuma’s Revenge游戏，缺少先验知识会导致exploration效果不好。

![1528115800653](/assets/images/post_images/transfer learning/1528115800653.png)

* If we’ve solved prior tasks, we might acquire useful knowledge for solving a new task 
* How is the knowledge **stored**? 
  * Q-function: tells us which actions or states are good 
  * Policy: tells us which actions are potentially useful 
    * some actions are never useful! (可以避免，提高exploration效率)
  * Models(**dynamics model**): what are the laws of physics that govern the world?
  * Features/hidden states: provide us with a good representation 
    * Don’t underestimate this!

> Aside(关于上述方法中最后一条feature/hidden states):
> paper [<<Loss is its own Reward: Self-Supervision for Reinforcement Learning>>](https://arxiv.org/abs/1612.07307)中研究了在atari游戏中representation的价值
>
> ![1528116741950](/assets/images/post_images/transfer learning/1528116741950.png)
>
> 上图中的黑线是standard RL，红线对应的方法: 首先train了一个policy直至收敛，然后将最后一次nn的参数毁掉，但是保留了前面的卷积层参数，然后端到端retrain，其intuition是为了保留representation但是要移除decision-making ability。(而conv层并不能做决策，只有representation的功能。)
>
> The gap between initial optimization and recovery shows a representation learning bottleneck.

## Transfer learning terminology 

**transfer learning**: using experience from <u>**one set of tasks**</u>($\textcolor{green}{source\enspace domain} $) for faster learning and better performance on **<u>a new task</u>**($\textcolor{yellow}{target\enspace domain} $)

in RL, $\textcolor{Red}{task} $=$\textcolor{BLUE}{MDP} $!

* “shot”: number of attempts in the target domain 
* 0-shot: just run a policy trained in the source domain
* 1-shot: try the task once
* few shot: try the task a few times 



