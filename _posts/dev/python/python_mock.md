title: Python Mock的入门
date: 2015-12-9 10:19:56
tags: Mock
category: dev
---

### Mock是什么

Mock这个词在英语中有模拟的这个意思，因此我们可以猜测出这个库的主要功能是模拟一些东西。准确的说，Mock是Python中一个用于支持的测试的库，它的主要功能是使用mock对象替代掉指定的Python对象，以达到模拟对象的行为。简单的说，mock库用于如下的场景：

假设你开发的项目叫a，里面包含了一个模块b，模块b中的一个函数c（也就是a.b.c）在工作的时候需要调用发送请求给特定的服务器来得到一个JSON返回值，然后根据这个返回值来做处理。如果要为a.b.c函数写一个单元测试，该如何做？
一个简单的办法是搭建一个测试的服务器，在单元测试的时候，让a.b.c函数和这个测试服务器交互。但是这种做法有两个问题：

测试服务器可能很不好搭建，或者搭建效率很低。
你搭建的测试服务器可能无法返回所有可能的值，或者需要大量的工作才能达到这个目的。
那么如何在没有测试服务器的情况下进行上面这种情况的单元测试呢？Mock模块就是答案。上面已经说过了，mock模块可以替换Python对象。我们假设a.b.c的代码如下：

```python
import requests

def c(url):
    resp = requests.get(url)
    # further process with resp
```
如果利用mock模块，那么就可以达到这样的效果：使用一个mock对象替换掉上面的requests.get函数，然后执行函数c时，c调用requests.get的返回值就能够由我们的mock对象来决定，而不需要服务器的参与。简单的说，就是我们用一个mock对象替换掉c函数和服务器交互的过程。你一定很好奇这个功能是如何实现的，这个是mock模块内部的实现机制，不在本文的讨论范围。本文主要讨论如何用mock模块来解决上面提到的这种单元测试场景。

### Mock的安装和导入

在Python 3.3以前的版本中，需要另外安装mock模块，可以使用pip命令来安装：
```bash
$ sudo pip install mock
```
然后在代码中就可以直接import进来：
```python
import mock
```
从Python 3.3开始，mock模块已经被合并到标准库中，被命名为unittest.mock，可以直接import进来使用：
```python
from unittest import mock
```

### Mock对象

#### 基本用法

Mock对象是mock模块中最重要的概念。Mock对象就是mock模块中的一个类的实例，这个类的实例可以用来替换其他的Python对象，来达到模拟的效果。Mock类的定义如下：
```python
class Mock(spec=None, side_effect=None, return_value=DEFAULT, wraps=None, name=None, spec_set=None, **kwargs)
```
这里给出这个定义只是要说明下Mock对象其实就是个Python类而已，当然，它内部的实现是很巧妙的，有兴趣的可以去看mock模块的代码。

Mock对象的一般用法是这样的：

找到你要替换的对象，这个对象可以是一个类，或者是一个函数，或者是一个类实例。
然后实例化Mock类得到一个mock对象，并且设置这个mock对象的行为，比如被调用的时候返回什么值，被访问成员的时候返回什么值等。
使用这个mock对象替换掉我们想替换的对象，也就是步骤1中确定的对象。
之后就可以开始写测试代码，这个时候我们可以保证我们替换掉的对象在测试用例执行的过程中行为和我们预设的一样。
举个例子来说：我们有一个简单的客户端实现，用来访问一个URL，当访问正常时，需要返回状态码200，不正常时，需要返回状态码404。首先，我们的客户端代码实现如下：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import requests

def send_request(url):
    r = requests.get(url)
    return r.status_code

def visit_ustack():
    return send_request('http://www.ustack.com')
```
外部模块调用visit_ustack()来访问UnitedStack的官网。下面我们使用mock对象在单元测试中分别测试访问正常和访问不正常的情况。
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import unittest
import mock
import client

class TestClient(unittest.TestCase):

    def test_success_request(self):
        success_send = mock.Mock(return_value='200')
        client.send_request = success_send
        self.assertEqual(client.visit_ustack(), '200')

    def test_fail_request(self):
        fail_send = mock.Mock(return_value='404')
        client.send_request = fail_send
        self.assertEqual(client.visit_ustack(), '404')
```
找到要替换的对象：我们需要测试的是visit_ustack这个函数，那么我们需要替换掉send_request这个函数。
实例化Mock类得到一个mock对象，并且设置这个mock对象的行为。在成功测试中，我们设置mock对象的返回值为字符串“200”，在失败测试中，我们设置mock对象的返回值为字符串"404"。
使用这个mock对象替换掉我们想替换的对象。我们替换掉了client.send_request
写测试代码。我们调用client.visit_ustack()，并且期望它的返回值和我们预设的一样。
上面这个就是使用mock对象的基本步骤了。在上面的例子中我们替换了自己写的模块的对象，其实也可以替换标准库和第三方模块的对象，方法是一样的：先import进来，然后替换掉指定的对象就可以了。

#### 稍微高级点的用法

class Mock的参数

上面讲的是mock对象最基本的用法。下面来看看mock对象的稍微高级点的用法（并不是很高级啊，最完整最高级的直接去看mock的文档即可，后面给出）。

先来看看Mock这个类的参数，在上面看到的类定义中，我们知道它有好几个参数，这里介绍最主要的几个：

name: 这个是用来命名一个mock对象，只是起到标识作用，当你print一个mock对象的时候，可以看到它的name。
return_value: 这个我们刚才使用过了，这个字段可以指定一个值（或者对象），当mock对象被调用时，如果side_effect函数返回的是DEFAULT，则对mock对象的调用会返回return_value指定的值。
side_effect: 这个参数指向一个可调用对象，一般就是函数。当mock对象被调用时，如果该函数返回值不是DEFAULT时，那么以该函数的返回值作为mock对象调用的返回值。
其他的参数请参考官方文档。

#### mock对象的自动创建

当访问一个mock对象中不存在的属性时，mock会自动建立一个子mock对象，并且把正在访问的属性指向它，这个功能对于实现多级属性的mock很方便。
```python
client = Client()
client.v2_client.get.return_value = '200'
```
这个时候，你就得到了一个mock过的client实例，调用该实例的v2_client.get()方法会得到的返回值是"200"。

从上面的例子中还可以看到，指定mock对象的return_value还可以使用属性赋值的方法。

#### 对方法调用进行检查

mock对象有一些方法可以用来检查该对象是否被调用过、被调用时的参数如何、被调用了几次等。实现这些功能可以调用mock对象的方法，具体的可以查看mock的文档。这里我们举个例子。

还是使用上面的代码，这次我们要检查visit_ustack()函数调用send_request()函数时，传递的参数类型是否正确。我们可以像下面这样使用mock对象。
```python
class TestClient(unittest.TestCase):

    def test_call_send_request_with_right_arguments(self):
        client.send_request = mock.Mock()
        client.visit_ustack()
        self.assertEqual(client.send_request.called, True)
        call_args = client.send_request.call_args
        self.assertIsInstance(call_args[0][0], str)
```
Mock对象的called属性表示该mock对象是否被调用过。

Mock对象的call_args表示该mock对象被调用的tuple，tuple的每个成员都是一个mock.call对象。mock.call这个对象代表了一次对mock对象的调用，其内容是一个tuple，含有两个元素，第一个元素是调用mock对象时的位置参数（*args），第二个元素是调用mock对象时的关键字参数（**kwargs）。

现在来分析下上面的用例，我们要检查的项目有两个：

visit_ustack()调用了send_request()
调用的参数是一个字符串
patch和patch.object

在了解了mock对象之后，我们来看两个方便测试的函数：patch和patch.object。这两个函数都会返回一个mock内部的类实例，这个类是class _patch。返回的这个类实例既可以作为函数的装饰器，也可以作为类的装饰器，也可以作为上下文管理器。使用patch或者patch.object的目的是为了控制mock的范围，意思就是在一个函数范围内，或者一个类的范围内，或者with语句的范围内mock掉一个对象。我们看个代码例子即可：
```python
class TestClient(unittest.TestCase):

    def test_success_request(self):
        status_code = '200'
        success_send = mock.Mock(return_value=status_code)
        with mock.patch('client.send_request', success_send):
            from client import visit_ustack
            self.assertEqual(visit_ustack(), status_code)

    def test_fail_request(self):
        status_code = '404'
        fail_send = mock.Mock(return_value=status_code)
        with mock.patch('client.send_request', fail_send):
            from client import visit_ustack
            self.assertEqual(visit_ustack(), status_code)
```
这个测试类和我们刚才写的第一个测试类一样，包含两个测试，只不过这次不是显示创建一个mock对象并且进行替换，而是使用了patch函数（作为上下文管理器使用）。

patch.object和patch的效果是一样的，只不过用法有点不同。举例来说，同样是上面这个例子，换成patch.object的话是这样的：
```python
    def test_fail_request(self):
        status_code = '404'
        fail_send = mock.Mock(return_value=status_code)
        with mock.patch.object(client, 'send_request', fail_send):
            from client import visit_ustack
            self.assertEqual(visit_ustack(), status_code)
```
就是替换掉一个对象的指定名称的属性，用法和setattr类似。

### 如何学习使用mock？

你肯定很奇怪，本文不就是教人使用mock的么？其实不是的，我发现自己在学习mock的过程中遇到的主要困难是不清楚mock能做什么，而不是mock对象到底有哪些函数。因此写这篇文章的主要目的是为了说明mock能做什么。

当你知道了mock能做什么之后，要如何学习并掌握mock呢？最好的方式就是查看阅读官方文档，并在自己的单元测试中使用。

最后，学习mock技能你应该要能够感受到一种控制的快感，就是你能享受控制外部服务的快乐。当你感受到这种快感的时候，你的mock应该就达到熟练使用的水平了。

### 官方文档

Python 2.7

mock还未加入标准库。

http://www.voidspace.org.uk/python/mock/index.html

Python 3.4

mock已经加入了标准库。

https://docs.python.org/3.4/library/unittest.mock-examples.html
https://docs.python.org/3.4/library/unittest.mock.html