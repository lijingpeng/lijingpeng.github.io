title: Python print 中文乱码问题
date: 2016-8-31 10:19:56
tags: python
category: dev
---

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

file = open('~/file.csv')
lines = file.readlines()
for line in lines:
    print line.decode('gbk')
```
