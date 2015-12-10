date: 2015-5-14
title: Shadowsocks 安装配置使用
tags: Shadowsocks
category: system
---

Debian/Ubuntu安装
```bash
apt-get install python-pip
pip install shadowsocks
```
配置:
```bash
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false,
    "workers": 1
}
```
服务器端运行:
```bash
ssserver -c /etc/shadowsocks/config.json
```
后台运行:
```bash
nohup ssserver -c /etc/shadowsocks/config.json > /dev/null &
```
客户端运行:
```bash
sslocal -c /etc/shadowsocks/config.json
```
后台运行:
```bash
nohup sslocal -c /etc/shadowsocks/config.json > /dev/null &
```
浏览器配置，推荐使用SwithySharp插件
```bash
protocol: socks5
hostname: 127.0.0.1
port:     your local_port
```