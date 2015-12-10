date: 2014-06-08
title: Swift and LLVM
tags: swift, llvm

------------
2014年6月2号，苹果在一年一度的WWDC上发布了新的编程语言Swift，根据苹果的官方介绍，Swift从开始研发到最终发布用了仅不足4年的时间，这应该算是一个比较短的时间周期了，另外WWDC上苹果还介绍了Swift的一些关键特性，例如：
- Swift从一些脚本语言如Python、Ruby、Javascript上吸取了一些好的特性
- 提供实时预览Playgrounds
- 性能比Objective-C 提升了大约40%～50%  

当然还有一些其他的特性，在这里就不列举了，不过从性能指标上来看，这个提升度是相当惊人的，这背后“必有蹊跷”，看看Swift的研发团队——苹果开发者工具部门总监克里斯·拉特纳（Chris Lattner）及其所带领的团队，我们可能恍然大悟，Lattner时LLVM项目的发起人，也是主要作者，在此，我们也简要介绍一下LLVM项目

1. LVM，命名最早源自于底层虚拟机（Low Level Virtual Machine）的缩写。它是一个编译器的基础建设，以C++写成。它是为了任意一种编程语言写成的程序，利用虚拟技术，创造出编译时期，链结时期，运行时期以及“闲置时期”的优化。它最早是以C/C++为实现对象，目前它支持了包括ActionScript、Ada、D语言、Fortran、GLSL、Haskell、Java bytecode、Objective-C、Swift、Python、Ruby、Rust、Scala以及C♯。[1]
2. LLVM项目起源于2000年伊利诺伊大学厄巴纳-香槟分校维克拉姆·艾夫（Vikram Adve）与克里斯·拉特纳（Chris Lattner）的研究发展而成，他们想要为所有静态及动态语言创造出动态的编译技术。LLVM是以BSD授权来发展的开源码软件。在2005年，苹果计算机雇用了克里斯·拉特纳及他的团队，为了苹果计算机开发应用程序系统，LLVM为现今Mac OS X及iOS开发工具的一部分。[1]
3. LLVM的起名为Low Level Virtual Machine的首字字母缩写，由于这个项目的范围并不局限于创建一个虚拟机，所以这个缩写导致了广泛的疑惑。之后，LLVM开始成长，他成为众多编译工具及低级工具技术的统称，这使得这个名字变得更不贴切，所以这个项目放弃了这个缩写的意涵，现今LLVM已经单纯成为一个品牌，适用于LLVM底下的所有项目，包含LLVM中介码（LLVM IR）、LLVM除错工具、LLVM C++标准库...等。[1]

运行时期的性能，平均GCC比LLVM高出10%的性能。2013年的测试结果，LLVM可以编译出接近与GCC接近相同性能的运行码。[1]

LLVM引发一些人来为许多语言开发新的编译器，其中一个最引发注意的就是Clang，它是一个新的编译器，同时支持C、Objective-C以及C++。Clang本身性能优异，其生成的AST所耗用掉的内存仅仅是GCC的20%左右。FreeBSD 10预计使用Clang取代GCC。[2]

由于GCC下面的Objective-C项目很早之前就已经停止了，所以苹果公司有意识的考虑GCC的替代品来作为自家Mac和IOS的开发工具，因此如上文所言，苹果雇佣了Lattner和他的团队，LLVM也取代了GCC作为开发者的编译工具，与此同时，经过4年多的发展，开发工具团队在LLVM的基础上，总结现有的Objective-C的优点和劣势，并结合其他语言的长处，在WWDC上推出了新的Swift语言，虽然官方说的新特性尤其是性能指标还有待检验，但是新语言的发布对众多苹果开发者来说无疑是一大福音，他们再也无需面对OC那些晦涩的语法了，新语言的简洁性也将会吸引更多的开发者来做开发。

可见拥有一个大牛的团队是何等的重要啊，因LLVM对产业的贡献，计算机协会于2012年授与Adve、Lattner及Evan ChengACM软件系统奖。[1]

</br>
#### 参考文献
------------
1. http://zh.wikipedia.org/wiki/LLVM
2. http://zh.wikipedia.org/wiki/Clang

上文引用均来自维基百科，在此向所有的维基人表示感谢！
