date: 2015-9-23
title: 机器学习资料大汇总
tags: PPTP, vpn
category: machine learning
---

![](/images/other/ml.jpg)

注：本页面主要针对想快速上手机器学习而又不想深入研究的同学，对于专门的researcher，建议直接啃PRML，ESL，MLAPP以及你相应方向的书（比如Numerical Optimization，Graphic Model等），另外就是Follow牛会牛paper，如果谁有兴趣也可以一起来整理个专业的汇总页。本页面将持续更新，敬请关注，如有推荐的文章请留言，谢谢！

###  开源工具
----
[机器学习的开源工具](http://www.52ml.net/12043.html)
[Python机器学习库](http://www.52ml.net/13547.html)
[C++矩阵运算库推荐](http://www.52ml.net/13002.html)

###  公开课
----

- Machine Learning | Coursera Andrew NG在coursera上的课，难度比公开课略低，适合入门
- 斯坦福大学公开课 ：机器学习课程 Andrew NG在学校里面的课程，网易公开课有中英文字幕，可以配合笔记来看
- CMU机器学习系主任Tom Mitchell院士机器学习课程视频及课件（英文）
- 机器学习|加州理工，老师是Yaser Abu-Mostafa，会从最基本的理论开始，为你构建机器学习的基础。
- 机器学习基石 如果想听中文课程，台湾大学的这门就很合适，友情提示，台大的课程基本上都可以加快语速来听，原因你懂的
- 神经网络|多伦多大学 鼎鼎大名的Geoffrey Hinton ，这门课着实不容错过
- 凸优化课程|斯坦福 授课老师是凸优化经典教材的作者Stephen Boyd！有难度有挑战！
- 概率图模型  coursera的另外一个创始人，Daphne Koller的课程，值得一提的是，Koller因提出了Probabilistic Relational Models拿到了2001年的IJCAI Computers and Thought Award
- 统计学习|斯坦福 授课老师是ESL作者 ，还有同学把视频放在了百度网盘上～ 这个更快一些

### 1. 机器学习入门篇

#### 1.1 机器学习介绍

- 机器学习-维基百科  Machine Learning-Wikipedia
- 机器学习简史
- 规则与机器学习 不建议为了机器学习而机器学习，对于初学者应该是先规则再机器学习，规则直观，可以深入理解领域知识和特征，要记住一个机器学习的专家必须首先是该领域知识的专家。
- 贝叶斯思想 MLAPP 第5章 Bayesian statistics 第6章 Frequentist statistics 机器学习第6章 贝叶斯学习
- 监督学习 ESL 第2章 Overview of Supervised Learning

#### 1.2 书籍

- 《统计学习方法》 第1章 统计学习方法概论
- 《机器学习》（Mitchell） 第1章 引言
- PRML 第1章 Introduction
- MLAPP 第1章 Introduction 第2章 Probability
- ESL 第1章 Introduction
- Some Notes on Applied Mathematics for Machine (选修)
- Machine Learning Textbook minireviews
- List of Cool Machine Learning Books

#### 1.3 数学基础

- 线性代数：公开课： 线性代数；推荐文章 ： 线性代数的本质，
- 概率论：公开课： 概率课|台大 叶老师为人风趣幽默，课程也比较简单，容易听进去
- 书籍：MLAPP第二章
- 微积分：公开课：单变量微积分|MIT 多变量微积分|MIT

——————————————-

#### 1.4 LDA

- LDA最佳学习资料汇总

#### 1.4 Spectral Clustering

- Spectral Clustering最佳学习资料汇总

#### 1.5 图像处理

- 图像处理和计算机视觉中的经典论文

#### 2 线性回归模型

- PRML 第3章 Linear Models for Regression
- MLAPP 第7章 Linear Regression 第13章 Sparse Linear Models
- ESL 第3章 Linear Method for Regression

#### 3 线性分类模型

- PRML 第4章 Linear Models for Classification
- MLAPP 第8章 Logistic Regression 第9章 Generalized Linear Models and the exponential family
- ESL 第4章 Linear Method for Classification
- 统计机器学习 第6章 逻辑斯谛回归与最大熵模型

#### 4 神经网络

- PRML 第5章 Neural Networks
- ESL 第11章 Neural Networks
- 统计学习方法 第2章 感知机
- 机器学习 第4章 人工神经网络

#### 5 支持向量机

- 统计学习方法 第7章 支持向量机 (强烈推荐)
- PRML 第6章 Kernel Methods 第7章 Sparse Kernel Machine
- ESL 第12章 Support Vector Machines and Flexible Discriminants
- MLAPP 第14章 Kernels

#### 6 图模型

- PRML 第8章 Graphical Models
- MLAPP 第10章 Directed graphical models（Bayes nets） 第19章 Undirected Graphical Models（Marcov random fields）第20章 Exact inference for graphical models 第26章 Graphical model structure learning
- 统计学习方法 第10章 隐马尔可夫模型 第11章 条件随机场
- 机器学习 6.11 贝叶斯信念网
- ESL 第17章 Undirected Graphical Models
- Koller 的书
- Jordan 的书

#### 7 混合模型和EM

- PRML 第9章 Mixture Models and EM
- MLAPP 第11章 Mixture models and the EM algorithm
- ESL 8.5 The EM Algorithm
- 统计学习方法 第9章 EM算法及其推广

#### 8 近似推理

- PRML 第10章 Approximate Inference
- MLAPP 第21章 Variational Inference 第22章 More Variational Inference

#### 9 采样方法

- PRML 第11章 Sampling Methods
- MLAPP 第23章 Monte Carlo inference 第24章 Markov Chain Monte Carlo (MCMC) inference
- ESL 8.6 MCMC for Sampling from Posterior

#### 10 PCA

- PRML 第12章 Continuous Latent Variables
- MLAPP 第12章 Latent Linear Models
- ESL 14.5 Principal Componens， Curves and Surfaces

#### 11 HMM

- PRML 13.1 13.2 Hidden Marcov Models
- MLAPP 第17章 Marcov and Hidden Marcov Models

#### 12 组合模型

- (投票，boosting，bagging，树模型，model averaging)
- PRML 第14章 Combining Models
- 统计学习方法 第5章 决策树 第8章 提升方法
- MLAPP 第16章 Adaptive basis function models
- ESL 第15章 Random Forests 第16章 Ensemble Learning 8.7 Bagging 第9章 Additive Models, Trees, and Related Methods 第10章 Boosting and Additive Trees
- 机器学习 第3章 决策树学习

#### 14 聚类

- ESL 14.3 Cluster Analysis
- MLAPP 25章 Clustering
- PRML 9.1 K-means Clustering

#### 15 近邻

- ELS 第13章 Protype Methods and Nearest-Neighbors

#### 16 Deep Learning

- http://deeplearning.net/
- Deep Learning Tutorial
- MLAPP 第28章 Deep Learning

#### 2.2 Deep Learning教程

- UFLDL-斯坦福大学Andrew Ng教授“Deep Learning”教程

### 3. 自然语言处理入门篇

#### 3.1 斯坦福大学自然语言处理公开课

- NLP | 斯坦福  授课教师是 Dan Jurafsky 以及 Christopher Manning，英文不是很有信心的可以参考《斯坦福大学自然语言处理公开课中文解读》
- NLP | 哥伦比亚 授课老师是Michael Collins大神

#### 3.2 统计机器翻译

- Statistical Machine Translation
- 统计机器翻译开源软件汇总

转自：http://www.52ml.net/star?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io
