date: 2016-9-11
title: CDNJS 库以及 Google Fonts、Ajax 和 Gravatar 反代
tags:
category: other
---

由于某些众所周知的原因，好多开源的 JS 库采用的国外 CDN 托管方式在国内访问速度不如人意。所以我们特意制作了这个公益项目，托管了 CDNJS 的所有开源 JS 库以及反代了 Google Fonts、Ajax 和 Gravatar，并且全部支持 http 和 https

## 一、CDNJS 开源 JS 库

我们采用的方法是每天定时同步 CDNJS 的 Github

所有的 JS 库可以在这儿找到您需要的链接

http://cdn.css.net/libs/

并且支持 https ，比如

http://cdn.css.net/libs/jquery/2.1.4/jquery.min.js
https://cdn.css.net/libs/jquery/2.1.4/jquery.min.js


## 二、Google Fonts

我们采用的方法是万能的 Nginx 反代 + 关键词替换，具体方法可以摸这儿。

使用的时候，您只需要替换 fonts.googleapis.com 为 fonts.css.network 即可，如

<link href='//fonts.googleapis.com/css?family=Open+Sans' rel='stylesheet'>
替换成

<link href='//fonts.css.network/css?family=Open+Sans' rel='stylesheet'>
如果需要 Material icons ，则把

https://fonts.googleapis.com/icon?family=Material+Icons
替换成

https://fonts.css.network/icon?family=Material+Icons
即可

## 三、Google 前端公共库

方法同上，直接替换 ajax.googleapis.com 为 ajax.css.network 即可，如

<script type='text/javascript' src='//ajax.googleapis.com/ajax/libs/jquery/2.1.4/jquery.min.js'></script>
替换成

<script type='text/javascript' src='//ajax.css.network/ajax/libs/jquery/2.1.4/jquery.min.js'></script>

## 四、Gravatar 反代

方法还是同上，直接替换 *.gravatar.com 为 gravatar.css.network 即可，如

https://cn.gravatar.com/avatar/8406d089bc81b664a2610b8d214c1428
替换成

https://gravatar.css.network/avatar/8406d089bc81b664a2610b8d214c1428

From: https://ttt.tt/185/
