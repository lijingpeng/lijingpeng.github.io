date: 2015-1-19
title: Apache htpasswd命令
tags: apache
category: system
---

在Linux主机中，为了保护站点的目录安全，通常会通过htaccess文件对目录添加密码来进行保护，如果忘记密码则可以通过htpasswd命令修改密码。

apache htpasswd选项参数:
```bash
htpasswd [-cmdpsD] passwordfile username
htpasswd -b[cmdpsD] passwordfile username password
htpasswd -n[mdps] username
htpasswd -nb[mdps] username password
```

apache htpasswd命令选项参数说明:
```bash
-c 创建一个加密文件
-n 不更新加密文件，只将apache htpasswd命令加密后的用户名密码显示在屏幕上
-m 默认apache htpassswd命令采用MD5算法对密码进行加密
-d apache htpassswd命令采用CRYPT算法对密码进行加密
-p apache htpassswd命令不对密码进行进行加密，即明文密码
-s apache htpassswd命令采用SHA算法对密码进行加密
-b 在apache htpassswd命令行中一并输入用户名和密码而不是根据提示输入密码
-D 删除指定的用户
```

## apache htpasswd例子

1、如何利用htpasswd命令添加用户？
```bash
htpasswd -bc .passwd tonyzhang pass
```
在bin目录下生成一个.passwd文件，用户名tonyzhang ，密码：pass，默认采用MD5加密方式

2、如何在原有密码文件中增加下一个用户？
```bash
htpasswd -b .passwd onlyzq pass
```
去掉c选项，即可在第一个用户之后添加第二个用户，依此类推

3、如何不更新密码文件，只显示加密后的用户名和密码？
```bash
htpasswd -nb tonyzhang pass
```
不更新.passwd文件，只在屏幕上输出用户名和经过加密后的密码

4、如何利用htpasswd命令删除用户名和密码？
```bash
htpasswd -D .passwd tonyzhang
```

5、如何利用htpasswd命令修改密码？
```bash
htpasswd -D .passwd tonyzhang
htpasswd -b .passwd tonyzhang pass
```
即先使用htpasswd删除命令删除指定用户，再利用htpasswd添加用户命令创建用户即可实现修改密码的功能。

文章链接：http://onlyzq.blog.51cto.com/1228/557593