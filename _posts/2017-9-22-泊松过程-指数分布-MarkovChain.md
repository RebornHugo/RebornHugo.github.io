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

