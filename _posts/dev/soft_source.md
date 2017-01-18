title: 常用的软件源
date: 2016-10-26 16:19:56
tags:
category: dev
---

## Ubuntu 软件源
---
以Dockerfile应用为例：

RUN echo deb http://mirrors.aliyun.com/ubuntu trusty universe >> /etc/apt/sources.list
RUN echo deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse >> /etc/apt/sources.list
RUN echo deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse >> /etc/apt/sources.list
RUN apt-get update

## Pip源
---

pip install -i http://pypi.douban.com/simple/ lxml
