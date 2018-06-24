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

> 注意这里的regret并不是在exploration strategy的所有地方都会用到的，regret只是一种用来估量一个strategy的效果quantity。实做上这些算法并没有怎么用到regret，不需要计算regret。

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
* 接下来将要把foundation derived from bandits apply to more complex MDP

# Exploration in Deep RL

## Classes of exploration methods in deep RL 

* **Optimistic** exploration:
  * new state = good state 
  * requires estimating **state visitation frequencies** or **novelty**  (count based)
  * typically realized by means of **exploration bonuses** 
* Thompson sampling style algorithms
  * learn distribution over Q-functions or policies 
  * **sample** and act according to sample 
* Information gain style algorithms 
  * reason about **information gain** from visiting new states

## Optimistic exploration in RL 

### Optimistic exploration in RL 

optimistic exploration很容易推广到deep RL，只需要修改reward为reward+bonus，其中bonus需要随之访问该state(或者state action pair)次数增加而减少。这里的bonus是count based。

![1527685263894](/assets/images/post_images/Exploration/1527685263894.png)

### The trouble with counts 

在蒙特祖玛的复仇游戏中，如果把图片帧当做state，那么由于游戏中的单位在变化，很难找到两个相同的state，所以难以做count。在continuous的情况下，比如拿锤子敲钉子情况会更加dire。解决方法是很符合intuition的：一些state/frame在pixel上不对应，却有着极为相似的性质，可以把它们count起来。

![1527685866683](/assets/images/post_images/Exploration/1527685866683.png)

### Fitting generative models(pseudo count) 

![1527686285553](/assets/images/post_images/Exploration/1527686285553.png)

![1527686631640](/assets/images/post_images/Exploration/1527686631640.png)

可以参考paper[<<Unifying Count-Based Exploration and Intrinsic Motivation>>](https://arxiv.org/abs/1606.01868)


### What kind of bonus to use? 

Lots of functions in the literature, inspired by optimal methods for bandits or small MDPs, 所以这些方法推广到DRL上时会难以证明一些性质。以下是一些常用的bonus function

![1527749794415](/assets/images/post_images/Exploration/1527749794415.png)

### What kind of model to use?

* density model/ generative model，也就是这里的$p_\theta(s)$，需要输出density的值，但是不一定非要产生samples。

* 和流行生成模型不同(eg. GAN, VAE)

* 上述paper中采用了名为CTS的模型：

   ![1527752226452](/assets/images/post_images/Exploration/1527752226452.png)

  详细可以参考paper。

### Counting with hashes 

* What if we still count states, but in a different space? 

* 也就是说，可以对image进行降维，哈希碰撞后相同编码的image理解为同一个state

  ![1527752628497](/assets/images/post_images/Exploration/1527752628497.png)

  > 对降维后的编码值维护一个tabel，表示count值。
  >
  > 参考paper[<<Exploration: A Study of Count-Based Exploration for Deep Reinforcement Learning>>](https://arxiv.org/abs/1611.04717)

### Implicit density modeling with exemplar models 

这套方法Exploit以下这条性质：

>  $p_\theta(s)$ 只需要能输出densities，不必非要产生samples

Can we explicitly compare the new state to past states? 

* Intuition：the state is **novel** if it is **easy** to distinguish from all previous seen states by a classifier

![1527757702627](/assets/images/post_images/Exploration/1527757702627.png)

以上的公式非常weird，特别是$D_s(s)$，下标s代表是用s作为训练集(还有$\mathcal{D}$)train出来的分类器，括号内的s代表这个classifier assigns to s of being in its own set( 当前输入是s)。这个classifier在试图解决一个非常weird的问题

> Is $s = s$?

下面是对这个问题的解释：

如果有一个small MDP，其中只有两种状态$A,B$，并且至今已经经历了状态A 10次，状态B 9次，接下来的s为B，注意这里dataset D本身就包含了s(的拷贝)。所以这里的分类器在解决一个ambiguous problem，并且optimal $D_s(s)\neq1$。

![1527767095715](/assets/images/post_images/Exploration/1527767095715.png)

> 推广到连续空间或者图片作state时，不会有重复状态，因此需要**regularize**

细节参考paper[<<EX2: Exploration with Exemplar Models for Deep Reinforcement Learning>>](https://arxiv.org/abs/1703.01260)

## Posterior sampling in deep RL 

首先回忆Thompson sampling，并考虑思考如何推广到DRL情况：

![1527771836779](/assets/images/post_images/Exploration/1527771836779.png)

如何表示所有Q函数的分布呢？(这里的$p(Q)$)

### Bootstrap

bootstrap是很容易想到的，这就是机器学习里ensemble方法中的一种。核心思想是从dataset里面做resemble，以获得多个不同的nn，也就是这里的Q函数。

![1527943606915](/assets/images/post_images/Exploration/1527943606915.png)

这里有两个trick：

> 1. 实做上可以不做resample，而是用random seed做多个nn的参数初始化，因为神经网络不是convex optimization problem，如果用SGD来解，那么它则是一个stochastic optimization problem。因此对不同的$Q_i$做不同的initialization，作用在同样的dataset上，得到的model之间仍然会有很大的variability。
> 2. train多个nn代价昂贵，可以共享前面的卷积层参数。

### Why does this work 

关于posterior sampling以及bootstrap方法的合理性：以游戏为例，如果采用随机动作来进行探索，那么效果会很差，因为可能会采取一些毫无意义的策略，比如向前走再向后走(**oscillate back and forth**)，这样虽然会带来diversity，但是难以探索到真正有用的state。

> **Cover the action space evenly with a near uniform distribution is not the same as covering the state space evenly**

但是如果使用随机的Q函数做探索，不会像随机动作一样做愚蠢的事情(**internally consistent**)。

以上便是著名的bootstrap DQN，参考paper[<<Deep Exploration via Bootstrapped DQN>>](https://arxiv.org/abs/1602.04621)

![1527946217772](/assets/images/post_images/Exploration/1527946217772.png)

这类方法的优点在于不需要像optimistic exploration一样添加bonus来修改reward function，所以实做上会更简单(less fuss)，但往往没有optimistic exploration的效果好，看上面第五个曲线图，bootstrap DQN在Montezuma's Revenge游戏上的效果仍然很差。

## Reasoning about Information gain(approximately)

### information gain about *what*?

回忆information gain: 
$$
IG(z,y|a)=E_y[\mathcal{H}(\hat{p}(z))-\mathcal{H}(\hat{p}(z)|y))|a]
$$
第一个需要解决的问题是：**information gain about *what*?**

以下是几种选择思路

* information gain about reward function $r(s,a)​$

  > cons: not very useful if reward is sparse

* state density $p(s)$

  > peculiar, make sense: 玩蒙特祖玛的复仇时如果新开了一扇门，那么这个state的density会上升

* dynamics 
  $$
  p(s\prime|s,a)
  $$

  > directly analogous to the bandit setting, $\theta$ in bandit 就是它的model, 而这里dynamics也是model
  >
  > > good proxy for *learning* the **MDP**, though still heuristic 

### VIME

但是以上的方法通常都是**intractable** to use exactly, regardless of what is being estimated!所以我们需要approximation。以下是几种approximation的思路

* prediction gain: $log\mathcal{p_{\theta\prime}}(s)-log\mathcal{p_\theta}(s)$

  > 可以把prediction gain当做bonus, 以上的p是count based method 里面的 **density** model

* variational inference:  首先，当使用dynamics来作为$Z$(latent variable)时，IG等价于
  $$
  D_{KL}(p(\theta|h,s_t,a_t,s_{t+1}||p(\theta|h)))
  $$
  ![1528019715834](/assets/images/post_images/Exploration/1528019715834.png)

  这套方法叫做VIME，参考[<<VIME: Variational Information Maximizing Exploration>>](https://arxiv.org/abs/1605.09674) Implementation如下： 

  ![123](/assets/images/post_images/Exploration/123-1528031151310.PNG)

> $$
> p(h|\theta)
> $$
>
> 比
> $$
> p(\theta|h)
> $$
> 容易获得
>
> 注意区分上面两个KL divergence。

简单来说，就是使用独立的高斯分布作为dynamics nn的参数，仍然使用bp来update对应的参数$\phi$。

使用最终的

$$
D_{KL}(q(\theta|\phi\prime)||q(\theta|\phi))
$$
作为bonus来进行exploration。

![1528033223285](/assets/images/post_images/Exploration/1528033223285.png)

实做上效果很好，橙色是VIME的效果。

### Exploration with model errors 

VIME中，
$$
D_{KL}(q(\theta|\phi\prime)||q(\theta|\phi))
$$
是用来更新网络中的$\phi$值，如果抛开information gain的思路，可以有更简单的方法来measure

Stadie et al. 2015:

* encode image observations using auto-encoder
* build predictive model on auto-encoder latent states
* use **model error**(判断下一state能否被model预测出来) as exploration bonus 

Schmidhuber et al. (see, e.g. <<Formal Theory of Creativity, Fun, and Intrinsic Motivation>>)：

* exploration bonus for model error
* exploration bonus for model gradient
* many other variations 

> 注意上述方法都是用于获得bonus，便于修改reward function，所以有着count based method所拥有的downside。