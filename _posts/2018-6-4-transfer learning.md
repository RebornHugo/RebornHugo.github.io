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

## How can we frame transfer learning problems? 

1. “Forward” transfer: train on one task, transfer to a new task
   * Just try it and hope for the best
   * Architectures for transfer: progressive networks
   *  Finetune on the new task
   * Randomize source task domain
2. Multi-task transfer: train on many tasks, transfer to a new task 
   * Model-based reinforcement learning 
   * Model distillation 
   * Contextual policies 
   * Modular policy networks 
3. Multi-task meta-learning: learn to learn from many tasks 
   * RNN-based meta-learning 
   * Gradient-based meta-learning 

# Try to Solve

## Forward transfer method

Policies trained for one set of circumstances might just work in a new domain, but no promises or guarantees. 这里省略机器人到拧盖子和倒水的两个例子，这种方法能否成功在于source domain与target domain的相似性。因此需要一些方法来stack the odds in favor

### Finetuning

The most popular transfer learning method in (supervised) deep learning! 

![1528118561670](/assets/images/post_images/transfer learning/1528118561670.png)

但是在RL与DL不同: **Where are the “ImageNet” features of RL?** 所以在RL中做finetune需要care更多的东西。

Challenges with finetuning in RL:

1. RL tasks are generally much less diverse 
   * Features are **less general**
   * Policies & value functions become **overly specialized** 
2. Optimal policies in **fully observed MDPs** are **deterministic**
   * Loss of exploration at convergence
   * Low-entropy policies adapt very slowly to new settings

### Finetuning with maximum-entropy policies 

How can we increase diversity and entropy? 回忆之前学过的connection between inference and control中学到的Max-ENT policy，还有quadruped robot的例子。

![1528119584067](/assets/images/post_images/transfer learning/1528119584067.png)

Act **as randomly as possible** while collecting high rewards! 

Example: pre-training for **robustness**

![1528119783507](/assets/images/post_images/transfer learning/1528119783507.png)

Example: [pre-training for **diversity**](https://rebornhugo.github.io/reinforcement%20learning/2018/05/17/Connections-Between-Inference-and-Control/#tractable-)

![1526735825995](https://rebornhugo.github.io/assets/images/post_images/Connections%20Between%20Inference%20and%20Control/1526735825995.png) 

Downside:

1. worse in the source domain
2. algorithm is more complex

### Architectures for transfer: progressive networks 

* An issue with finetuning
  * Deep networks work best when they are **big**
  * When we finetune, we typically want to use a little bit of experience
  * Little bit of experience + big network = **overfitting**
  * Can we somehow finetune a **small network**, but still pretrain a big network?
* Idea 1: finetune just a few layers 
  * Limited expressiveness(如果只finetune最后几层)
  * Big error gradients can wipe out initialization(如果finetune全部，并有overfitting的可能)
* Idea 2:add *new* layers for the new task(参考paper[<<Progressive Neural Networks>>](https://arxiv.org/abs/1606.04671))
  * Freeze the old layers, so no forgetting(不会wipe out initialization)
  * 由于有new architecture(smaller), 所以old conv layer不会wipe out information we need for new game

| *finetuning最后几层*                                         | *添加新结构*                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![1528163116841](/assets/images/post_images/transfer learning/1528163116841.png) | ![1528163177450](/assets/images/post_images/transfer learning/1528163177450.png) |

**pros**: alleviates some issues with finetuning

**cons**: not obvious how serious these issues are

### Finetuning Summary

* Try and hope for the best 
  * Sometimes there is enough variability during training to generalize 
* Finetuning 
  * A few issues with finetuning in RL
  * Maximum entropy training can help
* Architectures for finetuning: progressive networks
  * Addresses some overfitting and expressivity problems by construction 

### Randomize source task domain

What if we can manipulate the source domain? 

* So far: source domain (e.g., empty room) and target domain (e.g., corridor) are fixed(或者前面的ant learnining how to run in empty room and transfer to hallway)
* What if we can **design** the source domain, and we have a **difficult** target domain?
  * Often the case for simulation to real world transfer
* Same idea: the more diversity we see at training time, the better we will transfer! 

###  EPOpt: randomizing physical parameters 

下面的环境中，the discrepancy between the source and target consists of  physical parameter of the hopper.

> Generally the conventional wisdom for example in robust control is if you train under greater variabilty you will get more robustness as the cost of task performance. The interesting thing about DRL is that in general that might still be the case, 但有些情况下这些robust的policy可以不牺牲performance，仍然足够robust. 比如上图中用ensemble train出来的policy在mass为3/6/9时的performance与直接用对应的mass train出来的相同

![1528199498217](/assets/images/post_images/transfer learning/1528199498217.png)

> the more you randomized the physical parameter during training the more like you are to succeed when a test time there's a *new physical phenomena that you did not vary*.(**unmodeled effect**!!!)

reference: paper[<<EPOpt: Learning Robust Neural Network Policies Using Model Ensembles>>](https://arxiv.org/abs/1610.01283)

### Preparing for the unknown: explicit system ID 

![1528265887466](/assets/images/post_images/transfer learning/1528265887466.png)

> also vary physical parameter: 横坐标为offset of the center of the mass

Reference: [<<Preparing for the Unknown: Learning a Universal Policy with Online System Identification>>](https://arxiv.org/abs/1702.02453)，该系统由两部分组成: a Universal Policy (UP) and a function for Online System Identification (OSI). 后者为一个RNN，输出环境中的参数(诸如friction/mass之类). UP-OSI系统在这个task中效果接近UP-true(正确的环境参数)的效果。

Reference:[<<Sim-to-Real Transfer of Robotic Control with Dynamics Randomization>>](https://arxiv.org/abs/1710.06537)，这篇paper结合和上述两种方法: randomizing physical parameters和使用recurrent Policy，但是没有explicitly attempt to predict parameter而是一个implicit system identification.

Reference:[<<Sadeghi et al., “CAD2RL: Real Single-Image Flight without a Single Real Image” >>](https://arxiv.org/abs/1611.04201) Collision Avoidance via Deep Reinforcement Learning，在CAD模拟器中训练quadrotor(四旋翼飞行器)，并迁移到真实环境中。其中模拟的环境要求: less realistic simulator but one that is highly **randomized**. 这篇paper里观察到了: **enough diversity in the source domain can go a long way toward enabling transfer but you do have to make sure that you avoid sort of pathological regularity like the lack of reflections.**

------

前面几种方法是在无法观察target domain时所采取的但是, What if we can peek at the target domain? 

* So far: pure 0-shot transfer: learn in source domain so that we can succeed in **unknown** target domain

* Not possible in general: if we know nothing about the target domain, the best we can do is be as robust as possible

* What if we saw a few images of the target domain?

> 这个问题在传统RL中有很多研究，但是在DRL中这个问题fairly new

###  Better transfer through domain adaptation

Refrence: [<<Adapting Deep Visuomotor Representations with Weak Pairwise Constraints>>](https://arxiv.org/abs/1511.07111)

Confusion loss: A classifier that tries to look at the features inside the convolutional nn and guess are these features of simulated image or real image, and then the gradient of the classifier is **reversed** and bp back into convolutional layers.

This confusion loss is providing domain adaptation machinery just for the perception(希望从source/target 提取出来的features是invariance的). It's not attempting in any way to account for the physical discrepancy.

![1528286735408](/assets/images/post_images/transfer learning/1528286735408.png)

### Domain adaptation at the pixel level 

相比于上一篇paper中propose的方法，这里in addition to trying to regularize the features with this kind of confusion loss, it also attempts to **make simulated observations themselves look more like real world ones**.利用到了GAN的架构

![1528289788401](/assets/images/post_images/transfer learning/1528289788401.png)

### Forward transfer summary 

* Pretraining and finetuning

  * Standard finetuning with RL is hard
  * Maximum entropy formulation can help

* How can we modify the source domain for transfer?

  * Randomization can help a lot: the more diverse the better! 

* How can we use modest amounts of target domain data?

  * Domain adaptation: make the network unable to distinguish observations from the two domains
  * or modify the source domain observations to look like target domain
  * Only provides **invariance** – assumes all differences are **functionally irrelevant**; this is not always enough! 

> 关于这里的functionally irrelevant: if there's some object or some kind of property in simulation that is not present in the real world or vice-versa, you can simply **ignore** it and still succeed at the task.
>
> 但是很多情况下是不能简单的ignore的，比如模拟行车的环境，若是模拟器没有模拟出行人，我们不能通过上述方法ignore掉pedestrians的存在。
>
> functional relevant vairation between two domain(**frontier of current research** in this field)

## Multi-task transfer

### Multiple source domains 

* So far: more diversity = better transfer
* Need to design this diversity
  * E.g., simulation to real world transfer: randomize the simulation
* What if we transfer from multiple ***different*** tasks?
  * In a sense, closer to what people do: build on a lifetime of experience
  * Substantially harder: past tasks don’t directly tell us how to solve the task in the target domain! 

## Model-based reinforcement learning

* If the past tasks are all different, what do they have in common? 