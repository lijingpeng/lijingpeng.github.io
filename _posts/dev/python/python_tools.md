title: Python 工具
date: 2015-12-28 10:19:56
tags: python
category: dev
---

## 包管理
---

Python世界最棒的地方之一，就是大量的第三方程序包。同样，管理这些包也非常容易。按照惯例，会在 requirements.txt 文件中列出项目所需要的包。每个包占一行，通常还包含版本号。这里有一个例子，本博客使用Pelican：
```
pelican==3.3
Markdown
pelican-extended-sitemap==1.0.0
```

Python 程序包有一个缺陷是，它们默认会进行全局安装。我们将要使用一个工具，使我们每个项目都有一个独立的环境，这个工具叫virtualenv。我们同样要安装一个更高级的包管理工具，叫做pip，他可以和virtualenv配合工作。

首先，我们需要安装pip。大多数python安装程序已经内置了easy_install（python默认的包管理工具），所以我们就使用easy_install pip来安装pip。这应该是你最后一次使用easy_install 了。如果你并没有安装easy_install ，在linux系统中，貌似从python-setuptools 包中可以获得。

如果你使用的Python版本高于等于3.3， 那么Virtualenv 已经是标准库的一部分了，所以没有必要再去安装它了。

下一步，你希望安装virtualenv和virtualenvwrapper。Virtualenv使你能够为每个项目创造一个独立的环境。尤其是当你的不同项目使用不同版本的包时，这一点特别有用。Virtualenv wrapper 提供了一些不错的脚本，可以让一些事情变得容易。

```sh
sudo pip install virtualenvwrapper
```

当virtualenvwrapper安装后，它会把virtualenv列为依赖包，所以会自动安装。

```
打开一个新的shell，输入mkvirtualenv test 。如果你打开另外一个shell，则你就不在这个virtualenv中了，你可以通过workon test 来启动。如果你的工作完成了，可以使用deactivate 来停用。
```

## IPython
---

IPython是标准Python交互式的编程环境的一个替代品，支持自动补全，文档快速访问，以及标准交互式编程环境本应该具备的很多其他功能。

当你处在一个虚拟环境中的时候，可以很简单的使用pip install ipython 来进行安装，在命令行中使用ipython 来启动

## 测试
---

我推荐使用nose或是py.test。我大部分情况下用nose。它们基本上是类似的。我将讲解nose的一些细节。

这里有一个人为创建的可笑的使用nose进行测试的例子。在一个以test_开头的文件中的所有以test_开头的函数，都会被调用：

```python
def test_equality():
    assert True == False
```
不出所料，当运行nose的时候，我们的测试没有通过。

```
(test)jhaddad@jons-mac-pro ~VIRTUAL_ENV/src$ nosetests              
F
======================================================================
FAIL: test_nose_example.test_equality
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/jhaddad/.virtualenvs/test/lib/python2.7/site-packages/nose/case.py", line 197, in runTest
    self.test(*self.arg)
  File "/Users/jhaddad/.virtualenvs/test/src/test_nose_example.py", line 3, in test_equality
    assert True == False
AssertionError
 
----------------------------------------------------------------------
```
nose.tools中同样也有一些便捷的方法可以调用

```python
from nose.tools import assert_true
def test_equality():
    assert_true(False)
```
如果你想使用更加类似JUnit的方法，也是可以的：

```python
from nose.tools import assert_true
from unittest import TestCase
 
class ExampleTest(TestCase):
 
    def setUp(self): 
# setUp & tearDown are both available
        self.blah = False
 
    def test_blah(self):
        self.assertTrue(self.blah)
```
开始测试：

```
(test)jhaddad@jons-mac-pro ~VIRTUAL_ENV/src$ nosetests                            
F
======================================================================
FAIL: test_blah (test_nose_example.ExampleTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/jhaddad/.virtualenvs/test/src/test_nose_example.py", line 11, in test_blah
    self.assertTrue(self.blah)
AssertionError: False is not true
 
----------------------------------------------------------------------
Ran 1 test in 0.003s
 
FAILED (failures=1)
```
卓越的Mock库包含在Python 3 中，但是如果你在使用Python 2，可以使用pypi来获取。这个测试将进行一个远程调用，但是这次调用将耗时10s。这个例子显然是人为捏造的。我们使用mock来返回样本数据而不是真正的进行调用。

```python

import mock
 
from mock import patch
from time import sleep
 
class Sweetness(object):
    def slow_remote_call(self):
        sleep(10)
        return "some_data" 
# lets pretend we get this back from our remote api call
 
def test_long_call():
    s = Sweetness()
    result = s.slow_remote_call()
    assert result == "some_data"
```

当然，我们的测试需要很长的时间。

```
(test)jhaddad@jons-mac-pro ~VIRTUAL_ENV/src$ nosetests test_mock.py            
Ran 1 test in 10.001s
 
OK
```
太慢了！因此我们会问自己，我们在测试什么？我们需要测试远程调用是否有用，还是我们要测试当我们获得数据后要做什么？大多数情况下是后者。让我们摆脱这个愚蠢的远程调用吧：

```python

import mock
 
from mock import patch
from time import sleep
 
class Sweetness(object):
    def slow_remote_call(self):
        sleep(10)
        return "some_data" 
# lets pretend we get this back from our remote api call
 
def test_long_call():
    s = Sweetness()
    with patch.object(s, "slow_remote_call", return_value="some_data"):
        result = s.slow_remote_call()
    assert result == "some_data"
```
好吧，让我们再试一次：

```
(test)jhaddad@jons-mac-pro ~VIRTUAL_ENV/src$ nosetests test_mock.py                     
.
----------------------------------------------------------------------
Ran 1 test in 0.001s
OK
```
好多了。记住，这个例子进行了荒唐的简化。就我个人来讲，我仅仅会忽略从远程系统的调用，而不是我的数据库调用。

nose-progressive是一个很好的模块，它可以改善nose的输出，让错误在发生时就显示出来，而不是留到最后。如果你的测试需要花费一定的时间，那么这是件好事。
pip install nose-progressive 并且在你的nosetests中添加--with-progressive

## 调试
---

iPDB是一个极好的工具，我已经用它查出了很多匪夷所思的bug。pip install ipdb 安装该工具，然后在你的代码中import ipdb; ipdb.set_trace()，然后你会在你的程序运行
时，获得一个很好的交互式提示。它每次执行程序的一行并且检查变量。

## Gevent
---

Gevent 是一个很好的库，封装了Greenlets，使得Python具备了异步调用的功能。是的，非常棒。我最爱的功能是Pool，它抽象了异步调用部分，给我们提供了可以简单使用的途径，一个异步的map()函数

```python
from gevent import monkey
monkey.patch_all()
 
from time import sleep, time
 
def fetch_url(url):
    print "Fetching %s" % url
    sleep(10)
    print "Done fetching %s" % url
 
from gevent.pool import Pool
 
urls = ["http://test.com", "http://bacon.com", "http://eggs.com"]
 
p = Pool(10)
 
start = time()
p.map(fetch_url, urls)
print time() - start
```
非常重要的是，需要注意这段代码顶部对gevent monkey进行的补丁，如果没有它的话，就不能正确的运行。如果我们让Python连续调用 fetch_url 3次，通常我们期望这个过程花费30秒时间。使用gevent：
```
(test)jhaddad@jons-mac-pro ~VIRTUAL_ENV/src$ python g.py                                                                                                                                    
Fetching http://test.com
Fetching http://bacon.com
Fetching http://eggs.com
Done fetching http://test.com
Done fetching http://bacon.com
Done fetching http://eggs.com
10.001791954
```
如果你有很多数据库调用或是从远程URLs获取，这是非常有用的。我并不是很喜欢回调函数，所以这一抽象对我来说效果很好。