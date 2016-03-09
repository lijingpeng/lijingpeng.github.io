title: FastCGI 工作原理分析
date: 2016-2-24 13:19:56
tags: 
category: web
---

相对于 CGI/1.1 规范在 Web 服务器在本地 fork 一个子进程执行 CGI 程序，填充 CGI 预定义的环境变量，放入系统环境变量，把 HTTP body 体的 content 通过标准输入传入子进程，处理完毕之后通过标准输出返回给 Web 服务器。FastCGI 的核心则是取缔传统的 fork-and-execute 方式，减少每次启动的巨大开销（后面以 PHP 为例说明），以常驻的方式来处理请求。
FastCGI 工作流程如下：
```
1. FastCGI 进程管理器自身初始化，启动多个 CGI 解释器进程，并等待来自 Web Server 的连接。
2. Web 服务器与 FastCGI 进程管理器进行 Socket 通信，通过 FastCGI 协议发送 CGI 环境变量和标准输入数据给 CGI 解释器进程。
3. CGI 解释器进程完成处理后将标准输出和错误信息从同一连接返回 Web Server。
4. CGI 解释器进程接着等待并处理来自 Web Server 的下一个连接。
```

![](/images/other/fcgi.png)

FastCGI 与传统 CGI 模式的区别之一则是 Web 服务器不是直接执行 CGI 程序了，而是通过 socket 与 FastCGI 响应器（FastCGI 进程管理器）进行交互，Web 服务器需要将 CGI 接口数据封装在遵循 FastCGI 协议包中发送给 FastCGI 响应器程序。正是由于 FastCGI 进程管理器是基于 socket 通信的，所以也是分布式的，Web服务器和CGI响应器服务器分开部署。
再啰嗦一句，FastCGI 是一种协议，它是建立在CGI/1.1基础之上的，把CGI/1.1里面的要传递的数据通过FastCGI协议定义的顺序、格式进行传递。

来自： http://mengkang.net/668.html