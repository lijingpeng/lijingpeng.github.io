date: 2016-09-12
title: Linux查找文件内容（grep）
tags: linux
category: system
------------

grep是Linux命令行下常用于查找过滤文本文件内容的命令。最简单的用法是：

grep 'apple' fruitlist.txt

如果想忽略大小写，可以用-i参数：
grep -i 'apple' fruitlist.txt

如果想搜索目录里所有文件，包括子目录的话，并且在结果中显示行号，可以用以下命令：
grep -nr 'apple' folder
