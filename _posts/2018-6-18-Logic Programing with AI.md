---
title: Logic Programing with AI
categories:
 - AI
tag:
 - Logic Programing
 - Formal Verification
 - Artificial Intelligence

typora-copy-images-to: ../assets/images/post_images/Logic Programing with AI
typora-root-url: ..
---

今早刷知乎时看到了这样一个问题, [为什么 AI 理解不了逻辑问题？](https://www.zhihu.com/question/269017357/answer/347226738)这学期在学习形式化验证这门课时我接触到了大量的逻辑推理,当时也思考过这个问题.当下state of the art 的AI模型普遍是基于统计的模型,无论是贝叶斯流派还是深度学习,其核心都是统计模型,需要大量的数据以及频繁的训练.这带来了两个直观的问题:

* AI在解决问题上是没有**Intuition**的, 无法理解问题本身, 举例而言, 当前在NLP领域的很潮的LSTM,Attention的方法,CBOW,n-GRAM的模型, 它们都是基于词向量的表示, 词向量只是一种简单的统计. 机器甚至不能理解词的本身, 只能通过之间的Cosine similarity来判断他们的关系. 但人类不同, 比如"热"这个词, 人类真正会理解它是因为人类会出汗,  回想吃冰冷的失误来降温, 而这些都是最简单的Intuition和response而已. 不过通过transfer learning或者Inverse Reinforcement Learning等方法也许可以解决部分问题.
* 机器做不到"乌鸦"的感知、认知、推理、学习、和执行, 参考朱松纯教授的[Invited Talk](https://mp.weixin.qq.com/s/-wSYLu-XvOrsST8_KEUa-Q). 

-----

# Theoretical Impediments to Machine Learning

实际上, 早期的人工智能是以数理逻辑的表达与推理为主, 甚至许多人因此而获得了图灵奖, 80年代后则是以概率统计模型为主. 今年年初, 图灵奖得主Judea Pearl指出"To achieve human level intelligence, learning machines need the guidance of a model of reality, similar to the ones used in **causal inference** tasks. "

> 1. [[Paper](http://ftp.cs.ucla.edu/pub/stat_ser/r475.pdf)] Theoretical Impediments to Machine Learning With Seven Sparks from the Causal Revolution
> 2. [[Video](https://www.microsoft.com/en-us/research/video/plenary-2-the-mathematics-of-causal-inference-with-reflections-on-machine-learning/?from=http%3A%2F%2Fresearch.microsoft.com%2Fapps%2Fvideo%2Fdefault.aspx%3Fid%3D191888)] The Mathematics of Causal Inference: with Reflections on Machine Learning
> 3. [[Lecture Slides](http://bayes.cs.ucla.edu/jsm-august2016-bw.pdf)] Causal Inference in Statistics

这篇paper旨在从因果推理的角度阐述当下的机器学习的局限性. 因为almost entirely statistical 的方法无法做到推理, 只能做到

> learning machines improve their performance by optimizing parameters over a stream of sensory inputs received from the environment

也就是把从环境中所得作为输入, 通过解一个优化问题来不断调参, 最终达到不错的效果. 这类似于达尔文进化论中的自然选择(也有点像西部世界S2E7中Ford所说的西部世界原理). 但这会带来很强的局限性, 比如这套理论可以解释鹰**和蛇在百万年的进化后获得了很强的视力**, 但是无法解释**人类仅仅使用千年便学会了发明望远镜和眼镜**. 人类拥有着其它物种所缺乏的**mental representation**. 

![1529239931344](/assets/images/post_images/Logic Programing with AI/1529239931344.png)

> three-level hierarchy that governs inferences in causal reasoning

1. 第一个层次叫做Association(联想), 因为它使用的是纯统计学方法, 并且利用naked data. 例子: 购买牙膏的顾客也更容易购买牙线, 这种关联可以使用条件期望直接从观测数据中推断出来, 完全不需要因果推理.
2. 第二个层次叫做Intervention(干预), 它涉及的不仅仅是看到什么，还包括改变我们所看到的(What if...)。例子：如果我们将价格加倍会发生什么？这样的问题不能单从销售数据来回答，因为它们涉及到客户行为的变化, 这些都是无法观测到的。
3. 第三个层次叫做Counterfactuals(反事实), 这个术语可以追溯到哲学家大卫·休谟和约翰·斯图尔特·穆勒....例子: 如果我采取了不同的行动会怎么办，这需要追溯推理。

以上是因果关系层级. 所谓casusal inference(因果推理), 指**ability to choreograph a mental representation of their environment**. 比如上图中2,3 两层中的问题. 人类可以回答"What if I do X?"这类Intervention问题, 也可以回答"What if I had acted differently?"这类retrospective or explanatory问题. 但是目前基于统计学习的AI还无法理解这些问题.

每一层因果推理都有syntactic signature形式

* association layer: $$P(y|x) = p$$ stating that the probability of event Y = y given that we observed event X = x is equal to p. 可以使用贝叶斯网络或者深度学习来解决.
* interventional layer: $$P(y|do(x), z)$$  denotes “The probability of event Y = y given that we intervene and set the value of X to x and subsequently observe event Z = z, 可以使用随机试验来估计或者Causal Bayesian Networks来具体分析求解. 小孩需要通过playful manipulation of the environment (usually in a deterministic playground) 学会Interventions的结果, AI则是通过exercising their designated sets of actions学会的. 也就是说这些Interventional expressions是不能单独地被动地就可以推断出的.
* counterfactual layer: $$P(y_x|x\prime, y\prime)$$ stands for "The probability that event Y = y would be observed had X been x, given that we actually observed X to be $x\prime$ and Y to be $y\prime$" 例如，已知Joe读两年大学，他的工资就会是$y\prime$, 求Joe读完大学后薪资为y的概率. 只有在我们拥有functional or Structural Equation models时或具有这些模型的属性时，才能计算出来这类问题.

上述三层结构及其所需要的形式限制解释了为什么基于纯统计的机器学习方法无法进行推理. 它也告诉了我们统计方法之外所需要其他的信息.



# ProbLog: A Probabilistic Prolog and its Application in Link Discovery

Formal Verification这门课上, 我学习了prolog语言, 但是这类纯推理语言有局限性, 它可以表示一个自动机却无法构造一个MDP(马尔科夫决策过程). 如果要使当下纯统计方法的机器学习能拥有推理能力, 就需要对prolog进行扩展,  这篇古老的paper <<[ProbLog: A Probabilistic Prolog and its Application in Link Discovery](https://ijcai.org/Proceedings/07/Papers/397.pdf)>>(顶会IJCAI 07年)做了这个工作.

ProbLog程序通过为每个子句指定其属于随机采样程序的概率来定义逻辑程序的分布，并且这些概率是相互独立的. ProbLog的语义定义为query的成功概率，该概率对应于查询在随机采样程序(randomly sampled program)中成功的概率. 这篇paper的主要贡献在于引入了一种能够高效计算成功率的方法. 

这篇paper从挖掘biological graphs作为案例出发. 这类问题是很难解决的, 举例:

> That gene A of organism B codes for protein C, which is expressed in tissue D, or that genes E and F are likely to be related since they co-occur often in scientific articles

这些互联的各种生物数据可以构成一个庞大的带权网络图. 针对这个datebase, 一个典型的query可以是: *某种疾病是否与某种基因有关联*. 在概率图中, 一个关联的importance可以用两个节点之间存在路径的概率来衡量(这里假设每条边的存在都以某一概率为true, 并且边之间都是相互独立的). 这类query可以轻易地通过定义(non-probabilistic) predicate path(N1, N2)来表示. 每一次query成功的概率就对应着两节点之前存在路径的概率.

Problog程序和prolog类似, 有很多define clauses. 但是Problog中每个子句都被标注了一个概率$p_i$.

**Example **  consider:

1.0: likes(X,Y):- friendof(X,Y).
0.8: likes(X,Y):- friendof(X,Z),likes(Z,Y).
0.5: friendof(john,mary).
0.5: friendof(mary,pedro).
0.5: friendof(mary,tom).
0.5: friendof(pedro,tom).

ProbLog program $$T = \{p1 : c1, ··· ,pn : cn\}$$ defines a probability distributions over logic programs $$L\subseteq L_T=\{c_1,...,c_n\}$$ in the following way:

$$P(L|T)=\Pi_{c_i\in L}p_i\Pi_{c_i\in L_T\\L}(1-p_i)$$



Prolog语言关心的是一个query能否成功, 而Problog关心的则是一个query成功的概率. 

并且再Problog程序T中的query q的成功概率  $$P(q|T)$$  可以定义为:

![1529334863567](/assets/images/post_images/Logic Programing with AI/1529334863567.png)

这篇paper后面重点讲解了如何计算query的成功概率. 并且进行了实验上证明.

今年五月份时arxiv上出现了一篇Paper <<[DeepProbLog: Neural Probabilistic Logic Programming](https://arxiv.org/abs/1805.10872)>>, 这篇paper使用neural predicate方法将深度学习和Problog相结合, 提出了一种新的概率逻辑编程框架. 并且通过实验证明了DeepProlog支持了

* both symbolic and sub-symbolic representations and inference,
* program induction
* probabilistic (logic) programming
* (deep) learning from examples

作者说这是目前第一个同时集成了常用神经网络, 概率逻辑建模, 推理的框架.

# Relational inductive biases, deep learning, and graph networks

这篇paper是今年六月发表再arxiv上的, 值得一提的是这篇[paper](http://arxiv.org/abs/1806.01261)的作者居然有27位, 我很少见过如此多人署名的一篇paper. 并且都来自于DeepMind, google brain, MIT, Edinburgh顶级研究机构.

这篇paper同时还是意见书, 综述, 和unification. 其中提到: 如果AI要实现人类一样的能力, 必须将组合泛化（combinatorial generalization）作为重中之重, 而结构化的表示和计算是实现这一目标的关键. 图(Graph)的一阶逻辑和概率推理

在论文里，作者探讨了如何在深度学习结构（比如全连接层、卷积层和递归层）中，使用关系归纳偏置（relational inductive biases），促进对实体、对关系，以及对组成它们的规则进行学习。

他们提出了一个新的AI模块——**图网络（graph network）**，是对以前各种对图进行操作的神经网络方法的推广和扩展。图网络具有强大的关系归纳偏置，为操纵结构化知识和生成结构化行为提供了一个直接的界面。

作者还讨论了图网络如何支持关系推理和组合泛化，为更复杂、可解释和灵活的推理模式打下基础。

