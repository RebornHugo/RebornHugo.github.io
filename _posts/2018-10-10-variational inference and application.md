---
title: variational inference and application
categories:
 - Machine Learning
tag:
 - ML

typora-copy-images-to: ../assets/images/post_images/variational inference and application
typora-root-url: ..
---

理解并推导变分推断，并应用于variational linear regression，variational EM，variational autoencoder， Weight Uncertainity in Neural Networks， VIME。

> 所谓变分法(variational method)，相对于(standard calulars)而言，后者用于finding derivatives of functions，前者涉及到泛函分析，用于define a functional as a mapping that takes a function as the input and that returns the value of the functional as the output。此处主要用于逼近posterior。

# Material

PRML chapter10

[course from 徐亦达](https://www.bilibili.com/video/av24062247)

[Video from Berkeley CS294-112](https://www.bilibili.com/video/av31527294/?p=14)

[PDF by Levin](chrome-extension://oemmndcbldboiebfnladdacbdfmadadm/http://211.65.106.2/cache/3/03/rail.eecs.berkeley.edu/40194678f8984f42d12fd804da2f998c/lec-14.pdf)

# EM (maximize likelihood)

## latent variable in general

z可以代表observation x来自第几个高斯，也可以用更general的形式给出。

$p(z)$可以是一个均值0方差I的简单高斯，此时z只是一个低维流形中的一个点，通过一个神经网络得到对应的$\mu_{nn}(z),\sigma_{nn}(z)$，便有了生成模型$p(x|z)$

![1539243159894](/assets/images/post_images/variational inference and application/1539243159894.png)



## Intuition

* Intuition: "guess" most likely z given $x_i$, and pretend it's the right one

* $\theta \leftarrow arg\max_\theta\frac{1}{N}\sum_iE_{z\sim p(z|x_i)}[logp_\theta(x_i,z)]$

* In the E step, we use the current parameter values θ old to find the posterior distribution of the latent variables given by p(Z|X, θ old ). We then use this posterior distribution to find the expectation of the **complete-data log likelihood** evaluated for some general parameter value θ

## Proof

在EM算法中，为了最大化似然(常见于GMM)
$$
\log p(X|\theta)=\mathcal{L}(q, \theta)+KL(q||p)\\
\mathcal{L}(q, \theta)=\sum_zq(Z)\log\frac{p(X,Z|\theta)}{q(Z)}\\
KL(q||p)=-\sum_zq(X)\log{\frac{p(Z|X,\theta)}{q(Z)}}
$$
显然，ELBO是最终的$\log p(X|\theta)$的下界。

E step: compute posterior, set $p(Z|X, \theta)​$ to $q(Z)​$ to minimize KL

![1539222143821](/assets/images/post_images/variational inference and application/1539222143821.png)

![1539222160718](/assets/images/post_images/variational inference and application/1539222160718.png)



M step: maximize ELBO w.r.t. $p(X,Z|\theta)$

![1539222239786](/assets/images/post_images/variational inference and application/1539222239786.png)

## Problem

这里最大的问题是: 如何计算posterior !

* 当z离散且少量，可以用贝叶斯公式计算，但是当遇上continuous latent variable时，无法直接计算，这时可以使用变分推断来近似后验。

  ![1539176150945](/assets/images/post_images/variational inference and application/1539176150945.png)

![1539176178872](/assets/images/post_images/variational inference and application/1539176178872.png)

# Variational Inference (approximate posterior)

## mean field

变分推断的核心是利用$$q(Z)$$来逼近后验$p(Z|X)$，根据问题的不同，有时还会需要最大化似然$p_\theta(x)$，比如上述
EM算法中的生成模型中，最大化似然其核心目标，不过会用到VI来计算后验。但是在以variational linear regression为代表的conditional  models中，likelihood $$\log p(t|x)$$是不变的，逼近后验使得$$q(w,\alpha)\approx p(w,\alpha|t,x)$$，最终利用此后验来构建predictive distribution($$p(t|x,\mathbb{t})=\int p(t|x,w)p(w|\mathbb{t})dw$$)才是该问题核心。

利用PRML  p465中的推导(mean field)，可以最大化ELBO得到迭代解。

![1539176968555](/assets/images/post_images/variational inference and application/1539176968555.png)

![1539177017440](/assets/images/post_images/variational inference and application/1539177017440.png)

> 显然这里是iterative method，因为更新$q_j(Z_j)$需要用到$q_{i(i\neq j)}$
>
> 截图自PRML chapter10

上述推导的核心在于将多个隐变量后验$p(Z|X)$近似为$q(Z)=\prod_{i=1}^Mq_i(Z_i)$，假设此处的$Z_i$独立(注意后验中的多个隐变量不独立，比如变分线性回归中的隐变量$w,\alpha$分别代表weight和weight的precision)，update $q(Z)$来最大化ELBO，当ELBO最大(等于$lnp(X)$)时，KL散度为0，即做到了$p(Z|X)=q(Z)$

上述使用mean field theory来最大化ELBO的方法，在变分线性回归中有使用，它存在的最大问题是**假设后验中每个隐变量是独立**的，这可能带来较为严重的**精度问题**。

## variational linear regression

variational linear regression的基础是bayesian linear regression(PRML chapter 3)，后者直接计算后验(此问题中为$p(w,\alpha|\mathbb{t,x})$ )，变分法则是近似算出后验。最终使用后验用来预测target的分布$$p(t|\mathbb{x,t})=\int p(t|x,w)p(w|\mathbb{t})dw$$

![1539249110290](/assets/images/post_images/variational inference and application/1539249110290.png)

* linear regression与EM，VAE这类generative model不同，回归问题应该是conditional model $p(t|x)$，注意该模型中$x$永远在conditional bar的右边，对每个$x_i$应该有个对应的target $t_i$。

* EM中$P_\theta(X)$通常是GMM等生成模型，我们还需要通过调整$\theta $ (GMM中的每个高斯的均值和方差/general latent variable中nn的参数)来最大化似然。但是在(variational) bayesian linear regression中，此处的$P_\theta(X)$对应$P(t|X)$，且$P(t|X)$不含$\theta$，不会改变，故**只需要逼近后验**，而**无需MLE**。

  > $$t\sim\mathcal{N}(w^T\phi_n(x), \beta^{-1}) $$
  >
  > $$ w\sim\mathcal{N}(\mathbb{0}, \alpha^{-1}I)$$
  >
  > $$\alpha\sim \Gamma(a_0, b_0)$$
  >
  > 我们假设$\beta $已知，对应于$$p_\theta(x)=\int p_{\theta}(x|z)p(z)dz$$中的各项有:
  >
  > $$p_\theta(x)=p(t|X)\\p_{\theta}(x|z)=p(t|X,w,\alpha)\\p(z)=p(w,\alpha)$$
  >
  > 注意：$\theta$在GMM中是每个高斯的均值和方差，在general latent variable中是nn的参数，在variational linear regression中**不存在**！
  >
  > 要点：此处t服从的高斯分布里没有额外的变量，因为$p(t|X,w,\alpha)$中囊括了该高斯所需的一切参数(即$w与\alpha$，假设$\beta$已知)，故使用mean field theory只会update $q(w)与q(\alpha)$，没有$\theta$来update，故$p_\theta(x)=p(t|X)$是一个常量。
  >
  > 问题：$\theta$与$Z(隐变量)$的界限是什么？能否把GMM中的$\theta$作为隐变量来update？能否把variational linear regression中的$w, \alpha$作为$\theta$来update？ 

## Weight Uncertainity in Neural Networks

参考gluon提供的[tutorial](https://gluon.mxnet.io/chapter18_variational-methods-and-uncertainty/bayes-by-backprop.html)，以及<<[Weight Uncertainity in Neural Networks](https://arxiv.org/abs/1505.05424)>>

本质上和variational linear regression一样，不过隐变量从linear combination的weight换成了neural network的weight。

![1539248945459](/assets/images/post_images/variational inference and application/1539248945459.png)

此处我们需要找到w的后验即：$P(w|X,t)$，but intractable !!!

于是假设$w\sim \mathcal{N}(w|\mu, sigma)$，令$$P(w|t,X)=q(w|\phi)$$，也就是说$$\phi$$是隐变量$w$的均值和方差。因此找到了最终逼近的后验$q(w|\phi)$就可以找到网络的权重，因此逼近后验是该问题的核心目标(而不是最大化似然$P_\theta(t|X)$)！

* 实做上与variational linear regression相同，通过最大化ELBO，update $\phi$逼近后验。

  注意由于w与X无关，故$p(w|X)=p(w)$


$$
\begin{equation} \label{eq1}
\begin{split}
\phi^\star&=arg\min_{\phi}KL[q(w|\phi)||p(w|X,t)]\\
&=arg\min_\phi\int q(w|\phi)\log \frac{q(w|\phi)}{p(w|X,t)}dw\\
&=arg\min_\phi\int q(w|\phi)\log \frac{q(w|\phi)p(t|X)}{p(w|X)p(t|w,X)}dw\\
&=arg\min_\phi KL[q(w|\phi)||p(w|X)]-\mathbb{E}_{q(w|\phi)}[\log p(t|w,X)]+E_{q(w|\phi)}[\log p(t|X)]\\
&=arg\min_\phi \log p(t|X)-\underbrace{(\mathbb{E}_{q(w|\phi)}[\log p(t|w,X)-\log q(w|\phi)+\log p(w)])}_{ELBO}
\end{split}
\end{equation}
$$


所以objective为minimize
$$
\begin{equation} \label{eq2}
\begin{split}
-ELBO&=\mathbb{E}_{q(w|\phi)}[\log q(w|\phi)-\log p(w)-\log p(t|w,X)])\\
&=\mathbb{E}_{\epsilon\sim \mathcal{N(0,1)}}[\log q((\mu+\epsilon\sigma)|\phi)]-\log p(\mu+\epsilon\sigma)-\log p(t|(\mu+\epsilon\sigma),X) \\
&\approx\sum_{i=1}^{n}\log q(w^{(i)}|\phi)-\log p(w^{(i)})-\log p(t|w^{(i)},X)
\end{split}
\end{equation}
$$

其中$$\mu, \sigma=\phi[0, 1]$$

> problem：必须使用重参数化！！不然此处近似有误！
>
> 因为$w$是从$q(w|\phi)$中sample出来的，对$\phi$求导时有影响





三项分别为: variational posterior, prior, likelihood

最后使用gradient descend来更新$\phi$

* for **sequential data** (usually occur in reinforcement learning):

* $$
  \begin{equation} \label{eq4}
  \begin{split}
  -ELBO&=KL[q(w|\phi)||P(w)]-\mathbb{E}_{q(w|\theta)}[\log P(\mathcal{D}|w)]\\
  &=\int q(w|\phi)\log \frac{q(w|\phi)}{P(w)P(\mathcal{D}_{t-1},x_t|w)}dw\\
  &=\int q(w|\phi)\log \frac{q(w|\phi)}{P(\mathcal{D}_{t-1},x_t,w)}dw\\
  &=\int q(w|\phi)\log \frac{q(w|\phi)P(\mathcal{D}_{t-1})}{P(\mathcal{D}_{t-1},x_t,w)}dw-\underbrace{\int q(w|\phi)\log P(\mathcal{D}_{t-1})dw}_{const=\log P(\mathcal{D}_{t-1})} \\
  &=\int q(w|\phi)\log \frac{q(w|\phi)P(\mathcal{D}_{t-1})}{P(x_t,w|\mathcal{D}_{t-1})P(\mathcal{D}_{t-1})}dw-const\\
  &=\int q(w|\phi)\log \frac{q(w|\phi)}{P(x_t,w|\mathcal{D}_{t-1})}dw-const\\
  &=\int q(w|\phi)\log \frac{q(w|\phi)}{P(w|\mathcal{D}_{t-1})P(x_t|w,\mathcal{D}_{t-1})}dw-const\\
  &=\int q(w|\phi)\log \frac{q(w|\phi)}{P(w|\mathcal{D}_{t-1})P(x_t|w,\mathcal{D}_{t-1})}dw-const\\
  &=KL[q(w|\phi)||P(w|\mathcal{D}_{t-1})]-\mathbb{E}_{w\sim q(\cdot|\phi)}[\log P(x_t|w,\mathcal{D}_{t-1})]-const\\
  \end{split}
  \end{equation}
  $$

  if previous step is fully update, then $q(w|\phi_{t-1})=P(w|D_{t-1})$, this is a **prior** w.r.t the new data $x_t$, we can easily find in these deduce process, the first row is analogous to the last row.

* $$
  \begin{equation} \label{eq5}
  \begin{split}
  \phi^\prime&=arg\min_\phi[-ELBO]\\
  &=arg\min_\phi[KL[q(w|\phi)||q(w|\phi_{t-1})]-\mathbb{E}_{w\sim q(\cdot|\phi)}[\log P(x_t|w,\mathcal{D}_{t-1})]]
  \end{split}
  \end{equation}
  $$

  in the case of <<VIME>>, we use this equation (12) to update BNN, note that $\mathcal{D}_{t-1} $ is past trajectories $\xi_t,a_t$,  and $x_t$ is $s_t$


> 问题：把每个维度的$p(w|X,t)$看作独立的分布，带来的精度问题？
>
> 参考[Importance Weighted Autoencoders](https://arxiv.org/abs/1509.00519)

## VIME

我在[exploration一节](https://rebornhugo.github.io/reinforcement%20learning/2018/05/28/Exploration/)已经总结过vime算法，它exploration的要点是information gain，并扩展到了deep RL中。

### recap information theory

参考PRML chapter 1.6 & wiki <u>[Mutual information](https://en.wikipedia.org/wiki/Mutual_information)</u>   [<u>Entropy</u>](https://en.wikipedia.org/wiki/Entropy_(information_theory))   <u>[conditional entropy](https://en.wikipedia.org/wiki/Conditional_entropy)</u>

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d4/Entropy-mutual-information-relative-entropy-relation-diagram.svg/256px-Entropy-mutual-information-relative-entropy-relation-diagram.svg.png)

$I(X;Y)$表示X，Y的互信息: we can view the mutual information as the **reduction in the uncertainty** about x
by virtue of being told the value of y (or vice versa)
$$
I(X;Y)=\int_\mathcal{Y}\int_{\mathcal{X}}p(x,y)log(\frac{p(x,y)}{p(x)p(y)})dxdy=H(Y)-H(Y|X)
$$

$$
\begin{equation} \label{eq3}
\begin{split}
I(X;Y)&=\int_\mathcal{Y}\int_{\mathcal{X}}p(y)p(x|y)log(\frac{p(y)p(x|y)}{p(x)p(y)})dxdy\\
&=\int_{\mathcal{Y}}p(y)D_{KL}(p(x|y)||p(x))\\
&=\mathbb{E}_Y[D_{KL}(p(x|y)||p(x))]
\end{split}
\end{equation}
$$

> Relation to Kullback–Leibler divergence  该公式会在后面推导信息增益时会使用

### information gain

information gain的作用是quantify从一个action中获取的信息

![1527667118711](/assets/images/post_images/Exploration/1527667118711.png)

### information gain for what

根据
$$
IG(z,y|a)=E_y[\mathcal{H}(\hat{p}(z))-\mathcal{H}(\hat{p}(z)|y))|a]
$$
第一个需要解决的问题是：**information gain about *what*?**

以下是几种选择思路

- information gain about reward function $r(s,a)$

  > cons: not very useful if reward is sparse

- state density $p(s)$

  > peculiar, make sense: 玩蒙特祖玛的复仇时如果新开了一扇门，那么这个state的density会上升

- dynamics 
  $$
  p(s\prime|s,a)
  $$

  > directly analogous to the bandit setting, $\theta$ in bandit 就是它的model, 而这里dynamics也是model
  >
  > > good proxy for *learning* the **MDP**, though still heuristic 

### VIME

但是以上的方法通常都是**intractable** to use exactly, regardless of what is being estimated!所以我们需要approximation。以下是几种approximation的思路

- prediction gain: $log\mathcal{p_{\theta\prime}}(s)-log\mathcal{p_\theta}(s)$

  > 可以把prediction gain当做bonus, 以上的p是count based method 里面的 **density** model

- variational inference:  首先，当使用dynamics来作为$Z$(latent variable)时，根据前文推导的公式(relation to KL)，IG等价于：
  $$
  H(\Theta|\xi_t,a_t)-H(\Theta|S_{t+1},\xi_t,a_t)=I(S_{t+1};\mathbb{\Theta}|\xi_t,a_t)=\mathbb{E_{s_{t+1}\sim \mathcal{P(\cdot|\xi_t,a_t)}}}D_{KL}(p(\theta|h,s_t,a_t,s_{t+1}||p(\theta|h)))
  $$













![1528019715834](/assets/images/post_images/Exploration/1528019715834.png)

>  这里的后验$p(\theta|h)不可计算的原因是: $$p(\theta|h)=\frac{p(h|\theta)p(\theta)}{\int_\theta p(h|\theta)p(\theta)}d\theta$   这里要对神经网络的参数积分，由于参数过多故无法计算 

这套方法叫做VIME，参考[<<VIME: Variational Information Maximizing Exploration>>](https://arxiv.org/abs/1605.09674) Implementation如下： 

  ![123](/assets/images/post_images/Exploration/123-1528031151310.PNG)

> $$p(h|\theta)$$比$$p(\theta|h)$$容易获得，注意区分上面两个KL divergence
>

简单来说，就是使用独立的高斯分布作为dynamics nn的参数，仍然使用bp来update对应的参数$\phi$。

使用最终的
$$
D_{KL}(q(\theta|\phi\prime)||q(\theta|\phi))
$$
作为reward的bonus来进行exploration。显然，这里的dynamics nn与前文所述的<<weight uncertainty in nn>>中的网络无本质区别，推导完全相同，无非是bnn的输入从$X$换成了$S_t,a_t$，输出从$t$换成了$S_{t+1}$

> 由于使用的是reward + bonus，故这里的exploration会推迟进行

![1528033223285](/assets/images/post_images/Exploration/1528033223285.png)

实做上效果很好，橙色是VIME的效果。


# Variational auto-encoder

上述两个conditional model的例子中，$p_\theta(t|X)$内其实是**没有参数**$\theta$的，也就是说最终的似然是恒定的。但是

1. 在GMM+EM+Variational Inference中，$p_\theta(X)$中的$\theta$是指每一个高斯的均值方差。
2. 在general latent variable中, $p_\theta(X|Z)$中$\theta$是用来计算$\mu_{nn}(z),\sigma_{nn}(z)$的神经网络参数

如何更新$\theta$来使得likelihood $p_\theta(X)$最大？

* GMM中简单，一阶导为零
* nn中使用gradient ascend来maximize ELBO

$$
logp_{\theta}(x_i)= \overbrace{E_{z\sim q_i(z)}[logp_\theta(x_i|z)+logp(z)]+\mathcal{H}(q_i)}^{\mathcal{L_i(p,q_i)}=ELBO}+D_{KL}(q_i(z)||p(z|x_i))
$$

## variational EM

1. 更新$q_i(z)$来逼近后验$p(z|x_i)$来最小化KL
2. 更新$\theta$来最大化ELBO $\mathcal{L_i}$

上述1,2迭代进行

## In general

![1539271218454](/assets/images/post_images/variational inference and application/1539271218454.png)

> (这里有typo 应该是$z\sim q_i(z)$

上述方法存在的最大的问题是：对于每一个$x_i$，都有对应的$q_i$去近似后验$p(z|x_i)$，带来的问题是参数量过于庞大，故可以采用一个神经网络，输入$x_i$，得到对应的z的分布。

![1539271495468](/assets/images/post_images/variational inference and application/1539271495468.png)

观察到sample的z来自的分布含有需要优化的$\phi$，而计算期望的式子并不含$\phi$，故可以使用策略梯度中的score function的trick。

但是，使用policy gradient会带来variance过大的问题！因此引入重参rep数化（**reparametrization trick**）！

![1539271698790](/assets/images/post_images/variational inference and application/1539271698790.png)

将sample z转化为从固定的简单高斯$\mathcal{N}(0,1)$ sample $\epsilon$，从而降低方差。

## Another way to look at it

![1539516967763](/assets/images/post_images/variational inference and application/1539516967763.png)

## Reparameterization trick vs. policy gradient

![1539517020863](/assets/images/post_images/variational inference and application/1539517020863.png)

> 重参数化的要求更强(连续隐变量)，但一般能用重参数化尽量采用
>
> reparameterization trick and the basic REINFORCE-like update are **both unbiased** estimators of the same gradient, but reparameterization trick update tends to have lower variance in practice because it makes use of the log-likelihood gradients with respect to the **latent variables**.

## variational autoencoder

![1539517158851](/assets/images/post_images/variational inference and application/1539517158851.png)

$$p(x)=\int p(x|z)p(z)dz$$

sampling: $$z\sim p(z)$$   $$x\sim p(x|z)$$

why dose this work?  $$\mathcal{L}_i=E_{z\sim q_{\phi}(z|x_i)}[\log p_\theta(x_i|z)]-D_{KL}(q_\phi(z|x_i)||p(z ))$$

观察这个loss函数，直觉上可以理解为：使用encoder $q_\phi(z|x_i)$来逼近先验$p(z)$，故从$q_\phi(z|x_i)$中sample出来的z对应的$p_\theta(x_i|z)$可以表示为$x_i$