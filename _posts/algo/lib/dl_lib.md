title: 深度学习框架的评估与比较
date: 2016-02-16 16:07:12
tags: 
category: machine learning
---

人工智能无疑是计算机世界的前沿领域，而深度学习无疑又是人工智能的研究热点，那么现在都有哪些开源的深度学习工具，他们各自的优缺点又是什么呢？最近zer0n和bamos在GitHub上发表了一篇文章，对Caffe、CNTK、TensorFlow、Theano和Torch等深度学习工具从网络、模型能力、接口、部署、性能、架构、生态系统、跨平台等方面做了比较。
## 网络和模型能力
Caffe可能是第一个主流的工业级深度学习工具，它开始于2013年底,具有出色的卷积神经网络实现。在计算机视觉领域Caffe依然是最流行的工具包，它有很多扩展，但是由于一些遗留的架构问题，它对递归网络和语言建模的支持很差。此外，在Caffe中图层需要使用C++定义，而网络则使用Protobuf定义。

CNTK由深度学习热潮的发起演讲人创建,目前已经发展成一个通用的、平台独立的深度学习系统。在CNTK中，网络会被指定为向量运算的符号图，运算的组合会形成层。CNTK通过细粒度的构件块让用户不需要使用低层次的语言就能创建新的、复杂的层类型。

TensorFlow是一个理想的RNN（递归神经网络） API和实现，TensorFlow使用了向量运算的符号图方法，使得新网络的指定变得相当容易，但TensorFlow并不支持双向RNN和3D卷积，同时公共版本的图定义也不支持循环和条件控制，这使得RNN的实现并不理想，因为必须要使用Python循环且无法进行图编译优化。

Theano支持大部分先进的网络，现在的很多研究想法都来源于Theano，它引领了符号图在编程网络中使用的趋势。Theano的符号API支持循环控制，让RNN的实现更加容易且高效。

Torch对卷积网络的支持非常好。在TensorFlow和Theano中时域卷积可以通过conv2d来实现，但这样做有点取巧；Torch通过时域卷积的本地接口使得它的使用非常直观。Torch通过很多非官方的扩展支持大量的RNN，同时网络的定义方法也有很多种。但Torch本质上是以图层的方式定义网络的，这种粗粒度的方式使得它对新图层类型的扩展缺乏足够的支持。与Caffe相比，在Torch中定义新图层非常容易，不需要使用C++编程，图层和网络定义方式之间的区别最小。

## 接口
Caffe支持pycaffe接口，但这仅仅是用来辅助命令行接口的，而即便是使用pycaffe也必须使用protobuf定义模型。

CNTK的使用方式与Caffe相似，也是通过指定配置文件并运行命令行，但CNTK没有Python或者任何其他高级语言的接口。

TensorFlow支持Python和C++两种类型的接口。用户可以在一个相对丰富的高层环境中做实验并在需要本地代码或低延迟的环境中部署模型。

Theano支持Python接口。

Torch运行在LuaJIT上，与C++、C#以及Java等工业语言相比速度非常快，用户能够编写任意类型的计算，不需要担心性能，唯一的问题就是Lua并不是主流的语言。

## 模型部署
Caffe是基于C++的，因此可以在多种设备上编译，具有跨平台性，在部署方面是最佳选择。

CNTK与Caffe一样也是基于C++并且跨平台的，大部分情况下部署非常简单。但是它不支持ARM架构，这限制了它在移动设备上的能力。

TensorFlow支持C++接口，同时由于它使用了Eigen而不是BLAS类库，所以能够基于ARM架构编译和优化。TensorFlow的用户能够将训练好的模型部署到多种设备上，不需要实现单独的模型解码器或者加载Python/LuaJIT解释器。但是TensorFlow并不支持Windows，因此其模型无法部署到Windows设备上。

Theano缺少底层的接口，并且其Python解释器也很低效，对工业用户而言缺少吸引力。虽然对大的模型其Python开销并不 大，但它的限制摆在那，唯一的亮点就是它跨平台，模型能够部署到Windows环境上。

Torch的模型运行需要LuaJIT的支持，虽然这样做对性能的影响并不大，但却对集成造成了很大的障碍，使得它的吸引力不如Caffe/CNTK/TensorFlow等直接支持C++的框架。

## 性能
在单GPU的场景下，所有这些工具集都调用了cuDNN，因此只要外层的计算或者内存分配差异不大其表现都差不多。本文的性能测试是基于Soumith@FB的ConvNets基准测试来做的。
Caffe 简单快速。

CNTK 简单快速。

TensorFlow仅使用了cuDNN v2，但即使如此它的性能依然要比同样使用cuDNN v2的Torch要慢1.5倍，并且在批大小为128时训练GoogleNet还出现了内存溢出的问题。

Theano在大型网络上的性能与Torch7不相上下。但它的主要问题是启动时间特别长，因为它需要将C/CUDA代码编译成二进制，而TensorFlow并没有这个问题。此外，Theano的导入也会消耗时间，并且在导入之后无法摆脱预配置的设备（例如GPU0）。

Torch非常好，没有TensorFlow和Theano的问题。
另外，在多GPU方面，CNTK相较于其他的深度学习工具包表现更好，它实现了1-bit SGD和自适应的minibatching。

## 架构

Caffe的架构在现在看来算是平均水准，它的主要痛点是图层需要使用C++定义，而模型需要使用protobuf定义。另外，如果想要支持CPU和GPU，用户还必须实现额外的函数，例如Forward_gpu和Backward_gpu；对于自定义的层类型，还必须为其分配一个int类型的id，并将其添加到proto文件中。

TensorFlow的架构清晰，采用了模块化设计，支持多种前端和执行平台。

Theano 的架构比较变态，它的整个代码库都是Python的，就连C/
CUDA代码也要被打包为Python字符串，这使得它难以导航、调试、重构和维护。

Torch7和nn类库拥有清晰的设计和模块化的接口。
## 跨平台
Caffe、CNTK和Theano都能在所有的系统上运行，而TensorFlow和Torch则不支持Windows。

转自：http://www.linuxeden.com/html/news/20160128/164564.html