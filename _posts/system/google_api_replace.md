date: 2014-11-14
title: Wordpress替换fonts.googleapis.com
tags: Google
category: system
---

在wordpress自动调用fonts.googleapis.com, 这对于国内用户访问来说非常慢，这里推荐使用
[360常用前端公共库CDN服务](http://libs.useso.com/)

![](/images/other/googleapi.png)

操作办法：

在wordpress目录/var/www中执行命令，搜索本目录下所有包含googleapis字段的文件。
```bash
grep -H -R "googleapis" .
```
在vim下打开所有相关文件并逐步将所有googleapis换成useso
```bash
:%s/googleapis/useso/cg
```