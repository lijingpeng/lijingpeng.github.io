title: 理解Beta分布和Dirichlet分布
date: 2015-09-03 14:18:04
tags:
category: machine learning
---

</br>
在Machine Learning中，有一个很常见的概率分布叫做Beta Distribution：
![](/images/algo/j8b261e3942f702b2a36d5221f9a006bf.png)
同时，Dirichelet Distribution：

![](/images/algo/j24fc194e3126c49d315032f55d0a52e2.png)
### 解释
-------------------------------------
　　如果给你一个硬币，投这个硬币有\theta的概率抛出Head，有(1-\theta)的概率抛出Tail。如果在未来抛了五次这个硬币，有三次是Head，有两次是Tail，这个\theta最有可能是多少呢？如果你必须给出一个确定的值，并且你完全根据目前观测的结果来估计\theta，那么\theta = 3/5。

![](/images/algo/a.png)

　　如果未来抛出五次硬币，全部都是Head。那么按照1中的逻辑，你将估计\theta为1。也就是说，你估计这枚硬币不管怎么投，都朝上！可是，你想这或许是巧合：世界上没有这么屌的硬币，硬币还是有一定可能抛出Tail的。就算观测到再多次的Head，抛出Tail的概率还是不可能为0。这时候，Bayesian公式横空出世（如下图所示）。我们在估计\theta时，心中先有一个估计，即先验概率。这个估计，表现在Probability中，就是一个概率分布。通俗得来讲，我们不再认为\theta是个固定的值了。

![](/images/algo/j8b6e228eaeb570260d0f8c12a18add50.png)

　　在上面的Bayesian公式中，p(\theta)就是个概率分布。这个概率分布可以是任何概率分布，比如高斯分布，比如我们想要说的Beta Distribution。下图是Beta(5,2)的概率分布图。如果我们将这个概率分布作为p(\theta)，那么我们在还未抛硬币前，便认为\theta很可能接近于0.8，而不大可能是个很小的值或是一个很大的值。即，我们在抛硬币前，便估计这枚硬币更可能有0.8的概率抛出正面。

![](/images/algo/j9f3ba3610336773ac92573a548621b3e.png)

　　虽然p(\theta)可以是任何种类的概率分布，但是如果使用Beta Distribution，会让之后的计算更加方便。我们接着继续看便知道这是为什么了。况且，通过调节Beta Distribution中的a和b，你可以让这个概率分布变成各种你想要的形状！Beta Distribution已经很足够表达你事先对\theta的估计了。现在我们已经估计好了p(\theta)为一个Beta Distribution，那么p(X|\theta)是多少呢？其实就是个二项分布。继续以1中抛5次硬币抛出3次Head为例，X=抛5次硬币抛出3个Head的事件。

![](/images/algo/ja3a99d005b464c5164166547afc9e13b.png)
　　Bayesian公式下的p(X)是个Normalizer，或者叫做marginal probability。在\theta离散的情况下，p(X)就是\theta为不同值的时候，p(X|\theta)的求和。比如，如果我们事先估计硬币抛出正面的概率只可能是0.5或者0.8，那么p(X) = p(X|\theta=0.5)+p(X|\theta=0.8)，计算时分别将\theta=0.5和\theta=0.8代入到7中的公式中。而如果我们用Beta Distribution，\theta的概率分布在[0,1]之间是连续的，所以要用积分。

![](/images/algo/j48e8020e87f818d9414e03ffb9fcf248.png)

　　p(\theta)是个Beta Distribution，那么在观测到X=抛5次硬币中有3个head的事件后，p(\theta|X)依旧是个Beta Distribution！只是这个概率分布的形状因为观测的事件而发生了变化。
![](/images/algo/j0eca37b51f989944703ffbabadb40086.png)

　　因为观测前后，对\theta估计的概率分布均为Beta Distribution，这就是为什么使用Beta Distribution方便我们计算的原因了。当我们得知p(\theta|X)=Beta(\theta|a+3, b+2)后，我们就只要根据Beta Distribution的特性，得出\theta最有可能等于多少了。（即\theta等于多少时，观测后得到的Beta distribution有最大的概率密度）。例如下图，仔细观察新得到的Beta Distribution，和（5）中的概率分布对比，发现峰值从0.8左右的位置移向了0.7左右的位置。这是因为新观测到的数据中，5次有3次是head（60%），这让我们觉得，\theta没有0.8那么高。但由于我们之前觉得\theta有0.8那么高，我们觉得抛出head的概率肯定又要比60%高一些！这就是Bayesian方法和普通的统计方法不同的地方。我们结合自己的先验概率和观测结果来给出预测。

![](/images/algo/j01a0b8b112291e2355890ac32572b01b.png)

　　如果我们投的不是硬币，而是一个多面体（比如筛子），那么我们就要使用Dirichlet Distribution了。使用Dirichlet Distributio的目的，也是为了让观测后得到的posterior probability依旧是Dirichlet Distribution。比如，我们抛掷一个三面体。抛出这三个面的概率分别为\theta_1, \theta_2和\theta_3。不论\theta_1, \theta_2和\theta_3如何分布，它们相加必须等于1。那它们的概率分布，是在一个立体的空间里的一个面。这个面由\theta_1+\theta_2+\theta_3=1表示。这个面上的任意一点，表示某种\theta_1, \theta_2和\theta_3组合的概率密度。下三图分别由不同的\alpha vector初始化得到不同的Dirichlet Distribution，红颜色代表概率密度较大，蓝颜色的区域概率密度较小。

![](/images/algo/j3f8307ef170c6664eea5996932322a29.png)
　　Dirichlet Distribution和Beta Distribution都叫做Conjugate Prior。根据你的likelihood function，你可以选择对应的conjugate prior作为你对p(\theta)事先的估计。
![](/images/algo/j3481aac389b57152d20380150d0abd4a.png)
转自：http://maider.blog.sohu.com/306392863.html


