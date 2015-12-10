date: 2014-06-05
title: Github pages 搭建个人博客并绑定域名 
tags: github, pages, blog, 域名

------------

目前，github下pelican搭建的极简主义的博客越来越受到大家的欢迎，本文介绍在github中一个仓库同时保存Markdown博客主内容和HTML网页，并绑定域名。 我们将markdown文件保存在仓库的master分支中，将HTML网页保存在gh-pages分支中，并将域名绑定到gh-pages分支，下面是操作步骤：

#### 1. 首先在github创建仓库
并不局限于username.github.com这种的顶级域名仓库，如果想创建类似blog.example.com的博客，直接创建一个名字为blog的仓库即可。
#### 2. 克隆到本地：

    git clone git@github.com:yourusername/blog.git 

#### 3. 在克隆后的blog目录中初始化pelican工作目录 
```bash
               cd blog  
               pelican-quickstart
```
这样，本地blog目录将会有以下目录结构：
```bash  
    content  
    develop_server.sh*  
    fabfile.py  
    Makefile  
    output  
    pelicanconf.py  
    pelicanconf.pyc   
    publishconf.py  
    .git  
```

其中.git表示本目录被git所管理，因为我们知道markdown所写的内容必须放到content下，**因此我们可以将.git目录拷贝到content目录**

    cp -r .git content/

这样就确保我们的md文件将会被同步到github仓库的master分支

#### 4. 创建分支

进入content目录，并创建gh-pages分支（必须是这个分支名字）
```bash
    git checkout -b gh-pages  
    git push -u origin gh-pages  
    git checkout master
```
**在这一步骤中，只要我们自己创建gh-pages分支并同步到远端的时候，github会自动为我们建立一个可以访问的URL地址，格式是：http://username.github.io/blog**

#### 5. 同步静态网站到分支
pelican处理markdown生成的html文件将会放到output目录中，因此我们将gh-pages的checkout到该目录，以后html更新后，只需要在该目录下push即可。

克隆分支，在blog目录下：

    git clone -b gh-pages git@github.com:username/blog.git output

#### 6. 测试
- 在content目录下随便写一篇文章，例如example.md
```bash
    git status
    git add example.md
    git push origin master
```
这将会把该md文件同步到远端主分支，起到一定的备份作用

- 退出到blog目录下
```bash
    make html
```
生成HTML文件到output目录

- 到output目录下
```bash
    git status
    git add .
    git push origin gh-pages
```
这样就将网页同步到gh-pages分支了，打开浏览器，访问
http://username.github.io/blog
看看是不是能访问了

#### 7. 绑定域名
- 在DNS解析商那设置二级域名，例如本例中的blog，添加CNAME，指向username.github.io.
- 在output目录下创建CNAME文件，内容为要指向的域名，例如：blog.example.com，push到远端gh-pages分支即可。

#### 8. Notice
- 域名解析需要一定的时间才能在全球生效，设置完域名解析之后请稍后哦
- 文中的username和example需要替换成你自己的项目和域名

