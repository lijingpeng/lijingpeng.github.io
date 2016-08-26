date: 2016-8-25
title: 一张关于贝叶斯、HMM、CRF的图
tags:
category: machine learning
---

贝叶斯是生成模型，并且假设变量之间相互独立。那么对于像NLP中NER这样的任务是肯定不行的。HMM的出现，拓展了贝叶斯的图关系，model了观测变量的markov性，但是，始终表达能力不够，没有context。CRF不model输出与单个变量之间的关系了，而是与LR类似，model y 与 向量x之间的关系，加上图的local function的表达能力使得context信息和feature扩展能力变强。于是CRF得到了很不错的应用。

![](/images/条件随机场.png)
