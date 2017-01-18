title: Node/npm安装基本问题
date: 2016-10-11 20:19:56
tags:
category: dev
---

### 安装报错
---  

手动安装node的时候一不注意就会出现这个错误，装好了之后使用命令 npm 测试一下出来了这个错误：
```
Error: Cannot find module 'npmlog'
```

这一般是放在/bin里的链接不对导致的。

解决方案：
要使用软链接，在ln命令后加上 -s 选项：
```
ln -s node_modules/npm/bin/npm-cli.js /bin/npm
```

### 使用淘宝NPM镜像
---  

[官方链接](https://npm.taobao.org/)

```
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```
