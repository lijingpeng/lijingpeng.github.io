title: 使用Docker搭建一个隔离环境
date: 2015-12-24 16:19:56
tags: docker
category: dev
---

Docker轻量、隔离运行的特性非常适合运维部署，下面通过一个简单的例子介绍一下如何使用Docker搭建一个隔离的环境。
### 1. 安装Ubuntu镜像
```sh
docker pull ubuntu:14.04
```

等待上面的命令执行完之后，运行:

```sh
docker images
```
可以看到安装的镜像，比如在我的机器上：
```
REPOSITORY    TAG        IMAGE ID        CREATED        VIRTUAL SIZE
ubuntu        14.04      d55e68e6cc9c    2 weeks ago    187.9 MB
```

### 2. 启动并进入容器
执行:
```sh
docker run -i -t ubuntu:14.04
```

接下来就进入了Ubuntu的执行环境了，可以通过apt-get install来安装软件，默认情况下，如果不保存，退出容器时，安装的软件也会消失。例如我们通过apt-get安装了vim：
```sh
apt-get install vim
```

为了防止容器关闭时，安装的vim也消失，我们需要记住当前运行的容器id，执行:
```sh
docker ps
```
查看并记住当前运行的容器的id，在我的机器上，ID为：b14aa8cc9696
### 3. Docker commit
通过commit命令来保存对容器的变更：
```sh
docker commit b14 ubuntu-with-vim
```

注： 这里只需记住容器id的前几个字母就可以了

再次执行 `docker images` 即可看到保存后的新容器 `ubuntu-with-vim`， 相比于最开始的 `ubuntu`, `ubuntu-with-vim`安装了vim
