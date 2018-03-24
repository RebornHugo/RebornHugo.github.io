---
title: 泊松过程-指数分布-MarkovChain
categories:
 - Probability Theory
tag:
 - 软件可靠性
---
分析总结泊松过程，指数分布，隐式马尔可夫链之间的联系

# Poisson Distribution

## Definition

描述某段**时间**内，事件具体的发生概率。

> Example: 
>
> - 某医院平均每小时出生3个婴儿
> - 某网站平均每分钟有2次访问

## Forum1

$$P(N(t)=n)=\frac{(\lambda t)^ne^{-\lambda t}}{n!}$$

> 等号的左边，P 表示概率，N表示某种函数关系，t 表示时间，n 表示数量，1小时内出生3个婴儿的概率，就表示为 P(N(1) = 3) 。等号的右边，λ 表示事件的频率。

## Forum2

$$P(X = n)=\frac{e^{-\lambda}\lambda^n}{n!}$$

参数λ是**单位时间**（或单位面积）内随机事件的平均发生率。

> 单位时间内出生婴儿数为n的概率为p



# Poisson Process

## Definition

随机过程的一种，是以事件的发生时间来定义的，随即过程$$N(t)$$是一个时间齐次的一维泊松过程，满足的条件:

* 在两个互斥（不重叠）的区间内所发生的事件的数目是互相独立的随机变量

* 在区间$$[t, t+\tau]$$内发生的事件的数目的概率分布为

  ![P[(N(t+\tau )-N(t))=k]={\frac  {e^{{-\lambda \tau }}(\lambda \tau )^{k}}{k!}}\qquad k=0,1,\ldots ](https://wikimedia.org/api/rest_v1/media/math/render/svg/2854eb9a06534aa6a1beb2b86d721eb86cbccd93)

  λ是一个正数，是固定的参数，通常称为抵达率。

  给定时间区间$$[t, t+\tau]$$之中事件发生的数目k，则随机变数$$N(t+\tau)-N(t)$$呈现泊松分布，参数为$$\lambda t$$

## Extension

时间齐次的**泊松过程**也是时间齐次的连续时间[Markov过程](https://zh.wikipedia.org/wiki/Markov%E9%81%8E%E7%A8%8B)的例子。一个时间齐次、一维的**泊松过程**是一个[纯出生过程](https://zh.wikipedia.org/w/index.php?title=%E7%B4%94%E5%87%BA%E7%94%9F%E9%81%8E%E7%A8%8B&action=edit&redlink=1)，是一个[出生-死亡过程](https://zh.wikipedia.org/wiki/%E5%87%BA%E7%94%9F-%E6%AD%BB%E4%BA%A1%E9%81%8E%E7%A8%8B)的最简单例子。

# Exponential Distribution

## Definition

事件发生的**时间间隔为t**的概率 —> 事件在时间 t 之内发生的概率 

​                                                         —> 1 - P(时间间隔t内时间不发生)

## Example

* 婴儿出生的时间间隔

  > Can be derived from poisson process
  >
  > 如果下一个婴儿要间隔时间 t ，就等同于 t 之内(泊松分布)没有任何婴儿出生。
  >
  > $$P(x \leq t) = 1-P(x>t) = 1-P(N(t)=0)=1-\frac{(\lambda t)^0e^{-\lambda t}}{0!}=1 - e^{-\lambda t}$$

* 网站访问的时间间隔

## Forum

### Probability density function

![ f(x;\lambda) = \begin{cases}\lambda e^{-\lambda x} & x \ge 0, \\0 & x < 0.\end{cases}](https://wikimedia.org/api/rest_v1/media/math/render/svg/a693ce9cd1fcd15b0732ff5c5b8040c359cc9332)

### Cumulative distribution function

![F(x;\lambda) = \begin{cases}1-e^{-\lambda x} & x \ge 0, \\0 & x < 0.\end{cases}](https://wikimedia.org/api/rest_v1/media/math/render/svg/0702eba425ed0588b4e39702e7eb5bf724e3a699)

# About Markov 
## Markov Property
### Definition
> 当一个随机过程在给定现在状态及所有过去状态情况下，其未来状态的条件概率分布仅依赖于**当前状态**；换句话说，在给定现在状态时，它与过去状态（即该过程的历史路径）是**条件独立**的，那么此随机过程即具有马尔可夫性质。具有马尔可夫性质的过程通常称之为马尔可夫过程。

### Math Model
> 如果为$X(t),t>0$为一个随机过程，则**马尔可夫性质**就是指![https://wikimedia.org/api/rest_v1/media/math/render/svg/25a6129f9ba1a9ed334f56c45590db0743eabe58就是指](https://wikimedia.org/api/rest_v1/media/math/render/svg/25a6129f9ba1a9ed334f56c45590db0743eabe58)马尔可夫过程通常称其为（时间）齐次，如果满足
> ![https://wikimedia.org/api/rest_v1/media/math/render/svg/b635c7476d943f655adcba42f3e7a55ab8f5c079](https://wikimedia.org/api/rest_v1/media/math/render/svg/b635c7476d943f655adcba42f3e7a55ab8f5c079) 除此之外则被称为是（时间）非齐次的。

## Markov Process

|          |           **可数或有限的状态空间**           |          连续或一般的状态空间           |
| -------- | :--------------------------------: | :---------------------------: |
| **离散时间** |         在可数且有限状态空间下的马尔可夫链          | Harris chain (在一般状态空间下的马尔可夫链) |
| **连续时间** | **Continuous-time Markov process** |    任何具备马尔可夫性质的连续随机过程，如维纳过程    |



## Markov chain
### Definition
> 又称离散时间马尔可夫链。为状态空间中经过从一个状态到另一个状态的转换的随机过程。该过程要求具备“无记忆”的性质：下一状态的概率分布只能由当前状态决定，在时间序列中它前面的事件均与之无关。

### Math Model
马尔可夫链是满足马尔可夫性质的随机变量序列$X1, X2, X3, ...$，即给出当前状态，将来状态和过去状态是相互独立的。从形式上看，
如果两边的条件分布有定义（即如果
![https://wikimedia.org/api/rest_v1/media/math/render/svg/28d761c07cb61ec53bc6ed3e5d2d52b8b9c88ad5](https://wikimedia.org/api/rest_v1/media/math/render/svg/28d761c07cb61ec53bc6ed3e5d2d52b8b9c88ad5)则![https://wikimedia.org/api/rest_v1/media/math/render/svg/5bc8275bf0982e56b0563650eeb2dd2eed89d6bd则](https://wikimedia.org/api/rest_v1/media/math/render/svg/5bc8275bf0982e56b0563650eeb2dd2eed89d6bd)
Xi的可能值构成的可数集S叫做该链的“状态空间”。 
![https://upload.wikimedia.org/wikipedia/commons/thumb/2/2b/Markovkate_01.svg/563px-Markovkate_01.svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2b/Markovkate_01.svg/563px-Markovkate_01.svg.png)
## Hidden Markov model
### Definition
HMM is a statistical Markov model in which the system being modeled is assumed to be a **Markov process** with **unobserved** (i.e. hidden) states.
> 在正常的马尔可夫模型中，状态对于观察者来说是直接可见的。这样状态的转换概率便是全部的参数。而在隐马尔可夫模型中,状态并不是直接可见的，但受状态影响的某些变量则是可见的。每一个状态在可能输出的符号上都有一概率分布。因此输出符号的序列能够透露出状态序列的一些信息。

The hidden markov model can be represented as the simplest dynamic Bayesian network. 

### Example

* Part-of-speech tagging(词性标注)

![HMMs in Sentence](/assets/images/post_images/HMMs/HMMs in Sentence.png)

### Solution 

1. [Viterbi algorithm](https://en.wikipedia.org/wiki/Viterbi_algorithm)

   >  a dynamic programming algorithm for finding the most likely sequence of hidden states – called the Viterbi path – that results in a sequence of observed events, especially in the context of Markov information sources and hidden Markov models.