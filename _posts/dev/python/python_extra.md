title: Python snippets
date: 2016-12-16 10:19:56
tags: 
category: dev
---

## Python extra
---

### python中包含UTF-8编码中文的列表或字典的输出
在python 下面一个包含中文字符串的列表（list）或字典，直接使用print会出现以下的结果：

```python
>>> dict = {"asdf": "我们的python学习"}
>>> print dict
{'asdf': '\xe6\x88\x91\xe4\xbb\xac\xe7\x9a\x84python\xe5\xad\xa6\xe4\xb9\xa0'}
```
在输出处理好的数据结构的时候很不方便，需要使用以下方法进行输出：
```python
>>> import json
>>> print json.dumps(dict, encoding="UTF-8", ensure_ascii=False)
{"asdf": "我们的python学习"}
```
注意上面的两个参数，如果是字符串，直接输出：
```python
print str.encode("UTF-8")
```

### Python 判断字符串是否为数字

```python
# -*- coding: UTF-8 -*-

# Filename : test.py
# author by : www.runoob.com

def is_number(s):
    try:
        float(s)
        return True
    except ValueError:
        pass

    try:
        import unicodedata
        unicodedata.numeric(s)
        return True
    except (TypeError, ValueError):
        pass

    return False

# 测试字符串和数字
print(is_number('foo'))   # False
print(is_number('1'))     # True
print(is_number('1.3'))   # True
print(is_number('-1.37')) # True
print(is_number('1e3'))   # True

# 测试 Unicode
# 阿拉伯语 5
print(is_number('٥'))  # False
# 泰语 2
print(is_number('๒'))  # False
# 中文数字
print(is_number('四')) # False
# 版权号
print(is_number('©'))  # False
```

执行以上代码输出结果为：
```
False
True
True
True
True
False
False
False
False
```

Python isdigit() 方法检测字符串是否只由数字组成。
Python isnumeric() 方法检测字符串是否只由数字组成。这种方法是只针对unicode对象。


### Python中将（字典，列表等）变量格式化成（漂亮的，树形的，带缩进的，JSON方式的）字符串输出

想要JSON输出为带缩进的，树状的，很漂亮的效果，那么可以通过这样的方法：
```python
import json

# demoDictList is the value we want format to output
jsonDumpsIndentStr = json.dumps(demoDictList, indent=1)
print "jsonDumpsIndentStr=", jsonDumpsIndentStr
```
