title: MAC docker私有镜像安装配置
date: 2015-12-4 10:19:56
tags: docker
category: dev
---

MAC docker私有镜像安装配置：
1. 先安装brew install swig
2. 在安装docker-registry先安装M2Crypto
```bash
git clone git@github.com:martinpaljak/M2Crypto.git
cd M2Crypto
sudo python setup.py install
```
3. 安装gevent
```bash
sudo CFLAGS='-std=c99' pip install gevent==1.0.1
4. sudo pip install docker-registry
```

5. 重启
http://stackoverflow.com/questions/31990757/network-timed-out-while-trying-to-connect-to-https-index-docker-io
```bash
I had the same problem this morning and the following fixed it for me:

$ docker-machine restart default      # Restart the environment
$ eval $(docker-machine env default)  # Refresh your environment settings
It appears that this is due to the Docker virtual machine getting itself into a strange state.
```

