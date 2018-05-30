---
title: RL-11  Exploration
categories:
 - Reinforcement Learning
tag:
 - RL
 - DRL

typora-copy-images-to: ..\assets\images\post_images\Exploration
typora-root-url: ..
---

RL中永远存在的一个问题是exploration & exploitation trade off，这节课从多臂摇臂机模型(stateless)出发，探讨了Optimism-based / Posterior matching / Information-theoretic 三类探索方法的原理与实践，并推广到了DRL的情况。

# Material

[Video from Berkeley CS294-112](https://www.bilibili.com/video/av20957290/index_15.html?t=4715)

[PDF by Levin](http://rail.eecs.berkeley.edu/deeprlcourse-fa17/f17docs/lecture_13_exploration.pdf)

# Introduction

## Goals

* Understand what the exploration is
* Understand how theoretically grounded exploration methods can be derived
* Understand how we can do exploration in deep RL in practice 

## What's the problem

让机器玩atari游戏，发现有的游戏很容易，有的游戏(比如蒙特祖玛的复仇)很难

![1527647912129](/assets/images/post_images/Exploration/1527647912129.png)

因为这个游戏对于机器而言，显得peculiar，具体体现在：

* Getting key = reward

* Opening door = reward 

* Getting killed by skull = nothing (is it good? bad?) 

  > 因为被☠干掉后☠自身也会消失

* Finishing the game only weakly correlates with rewarding events

  > 比如一关有两个门，通过这一关不一定带来最终游戏的胜利

* We know what to do because we understand what these sprites mean! 

  > 机器缺少intuition about the world，没有prior knowledge，只能做trial and error，rules 对它而言没有用

  

Temporally extended tasks like Montezuma’s revenge become increasingly difficult based on

  * How extended the task is（how many complex things you have to do in the right order before you receive feedback, 比如让机器手拿锤子敲钉子这个动作(continuous control)，如果没有human demonstration， 机器手很难尝试到有feedback的动作）
  * How little you know about the rules 

我觉得有两个帮助exploration interesting things的方法：

* Prior knowledge，迁移学习也许可以用到
* additional guidance mechanism 

## Exploration and exploitation 

* Two potential definitions of exploration problem 
  * How can an agent discover high-reward strategies that require a **temporally extended sequence of complex behaviors** that, individually, are not rewarding? （尤其是连续性控制问题）
  * How can an agent decide whether to attempt new behaviors (to discover ones with higher reward) or continue to do the best thing it knows so far? 
* Actually the same problem: 
  * Exploitation: doing what you *know* will yield highest reward 
  * Exploration: doing things you *haven’t done before*, in the hopes of getting even higher reward 

例子有很多，比如尝试新饭店还是过去喜欢的饭店，投放新广告还是目前效益最好的广告......

## Theoretical

当问道Can we derive an optimal exploration strategy时，我们要先考虑到what does optimal even mean? 所以后面要定义一些名词

>  regret vs. Bayes-optimal strategy? more on this later… 

![1527652053112](/assets/images/post_images/Exploration/1527652053112.png)

以上指出了哪些exploration问题是理论上可解，我们将从multi-armed bandits出发推广到theoretical intractable的情况，并使用heuristic的方法来解。

# Solution within small MDP/Bandits

## What makes an exploration problem tractable

![1527652447274](/assets/images/post_images/Exploration/1527652447274.png)

## Bandits

这里的bandits不是强盗土匪，而是美国俚语里的老虎机。它是强化学习exploration问题中的drosophila(果蝇)，one-arm bandits 中摇臂机只有一个臂，状态空间为零，动作空间只有一个(摇臂)，并获得一个reward (满足
$$
r(a)\sim p(r|a)
$$
)。one-arm bandits没什么意思，我们要考虑的是multi-arm bandits。注意每个arm的reward都是stochastic。

![1527662613511](/assets/images/post_images/Exploration/1527662613511.png)

这个[网站](http://iosband.github.io/2015/07/28/Beat-the-bandit.html)提供了类似于multi-arm bandits的prescription游戏，并且给出了几种exporation方法的结果对比。

## Mathematical definition about bandits

![1527664237867](/assets/images/post_images/Exploration/1527664237867.png)

* Variety of relatively simple strategies(后面会介绍三种)
* Often can provide theoretical guarantees on regret
  * Variety of optimal algorithms (up to a constant factor)
  * But empirical performance may vary(不同exploration strategy实做上效果不同)… 
* Exploration strategies for more complex MDP(**DRL**) domains will be inspired by these strategies 

## Optimistic exploration

* Intuition: 没有探索的地方很可能会有好的回报，于是可以对没有exploration的action optimistic，在具体实现上可以加上一个uncertainty term。

  ![1527665366097](/assets/images/post_images/Exploration/1527665366097.png)

  > 比如上图中的UCB(upper confidence bound algorithm)

## Probability matching/posterior sampling 

这里要先回忆definition of bandits中提到的belief state，可以把它理解为bandit的model的distribution。也就是说
$$
\hat{p}(\theta_1, ...,\theta_n)
$$
可以理解为bandit这个model的参数为
$$
\theta_1,...,\theta_n
$$
的概率为$\hat{p}$。posterior sampling的过程如下：

![1527666474020](/assets/images/post_images/Exploration/1527666474020.png)

sample $\theta$比sample random action更有意义。参考[<<An Empirical Evaluation of Thompson Sampling>>](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/thompson.pdf)

## Information gain 

information gain的作用是quantify从一个action中获取的信息

![1527667118711](/assets/images/post_images/Exploration/1527667118711.png)

> 关于这里的entropy，像semi-supervised leanring里面entropy-based方法，附上[ppt链接](http://speech.ee.ntu.edu.tw/~tlkagk/courses/ML_2017/Lecture/semi.pdf)。

下面是一个例子，来自paper [<<Learning to Optimize via Information-Directed Sampling](https://arxiv.org/abs/1403.5556)>>

![1527667727961](/assets/images/post_images/Exploration/1527667727961.png)

## Summarize

General themes

![1527667930562](/assets/images/post_images/Exploration/1527667930562.png)

* Most exploration strategies require some kind of uncertainty estimation (even if it’s naïve) 
* Usually **assumes** some value to new information
  * Assume unknown = good (optimism)
  * Assume sample = truth
  * Assume information gain = good 