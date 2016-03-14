title: Python日志输出——logging模块
date: 2016-3-10 10:19:56
tags: python
category: dev
---

### 1. logging介绍
Python的logging模块提供了通用的日志系统，可以方便第三方模块或者是应用使用。这个模块提供不同的日志级别，并可以采用不同的方式记录日志，比如文件，HTTP GET/POST，SMTP，Socket等，甚至可以自己实现具体的日志记录方式。
logging模块与log4j的机制是一样的，只是具体的实现细节不同。模块提供logger，handler，filter，formatter。
logger：提供日志接口，供应用代码使用。logger最长用的操作有两类：配置和发送日志消息。可以通过logging.getLogger(name)获取logger对象，如果不指定name则返回root对象，多次使用相同的name调用getLogger方法返回同一个logger对象。
handler：将日志记录（log record）发送到合适的目的地（destination），比如文件，socket等。一个logger对象可以通过addHandler方法添加0到多个handler，每个handler又可以定义不同日志级别，以实现日志分级过滤显示。
filter：提供一种优雅的方式决定一个日志记录是否发送到handler。
formatter：指定日志记录输出的具体格式。formatter的构造方法需要两个参数：消息的格式字符串和日期字符串，这两个参数都是可选的。
与log4j类似，logger，handler和日志消息的调用可以有具体的日志级别（Level），只有在日志消息的级别大于logger和handler的级别。

```python
import logging  
import logging.handlers  
  
LOG_FILE = 'tst.log'  
  
handler = logging.handlers.RotatingFileHandler(LOG_FILE, maxBytes = 1024*1024, backupCount = 5) # 实例化handler   
fmt = '%(asctime)s - %(filename)s:%(lineno)s - %(name)s - %(message)s'  
  
formatter = logging.Formatter(fmt)   # 实例化formatter  
handler.setFormatter(formatter)      # 为handler添加formatter  
  
logger = logging.getLogger('tst')    # 获取名为tst的logger  
logger.addHandler(handler)           # 为logger添加handler  
logger.setLevel(logging.DEBUG)  
  
logger.info('first info message')  
logger.debug('first debug message')  
```
输出：
```
2012-03-04 23:21:59,682 - log_test.py:16 - tst - first info message   
2012-03-04 23:21:59,682 - log_test.py:17 - tst - first debug message 
```

### 2. logging的配置
logging的配置可以采用python代码或是配置文件。python代码的方式就是在应用的主模块中，构建handler，handler，formatter等对象。而配置文件的方式是将这些对象的依赖关系分离出来放在文件中。比如前面的例子就类似于python代码的配置方式。这里看一下采用配置文件的方式。
```python
import logging  
import logging.config  
  
logging.config.fileConfig("logging.conf")    # 采用配置文件  
  
# create logger  
logger = logging.getLogger("simpleExample")  
  
# "application" code  
logger.debug("debug message")  
logger.info("info message")  
logger.warn("warn message")  
logger.error("error message")  
logger.critical("critical message")  
```

loggin.conf采用了模式匹配的方式进行配置，正则表达式是r'^[(.*)]$'，从而匹配出所有的组件。对于同一个组件具有多个实例的情况使用逗号‘，’进行分隔。对于一个实例的配置采用componentName_instanceName配置块。使用这种方式还是蛮简单的。
```
[loggers]  
keys=root,simpleExample  
  
[handlers]  
keys=consoleHandler  
  
[formatters]  
keys=simpleFormatter  
  
[logger_root]  
level=DEBUG  
handlers=consoleHandler  
  
[logger_simpleExample]  
level=DEBUG  
handlers=consoleHandler  
qualname=simpleExample  
propagate=0  
  
[handler_consoleHandler]  
class=StreamHandler  
level=DEBUG  
formatter=simpleFormatter  
args=(sys.stdout,)  
  
[formatter_simpleFormatter]  
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s  
datefmt=  
```

在指定handler的配置时，class是具体的handler类的类名，可以是相对logging模块或是全路径类名，比如需要RotatingFileHandler，则class的值可以为：RotatingFileHandler或者logging.handlers.RotatingFileHandler。args就是要传给这个类的构造方法的参数，就是一个元组，按照构造方法声明的参数的顺序。

输出：
```
2012-03-06 00:09:35,713 - simpleExample - DEBUG - debug message  
2012-03-06 00:09:35,713 - simpleExample - INFO - info message  
2012-03-06 00:09:35,714 - simpleExample - WARNING - warn message  
2012-03-06 00:09:35,714 - simpleExample - ERROR - error message  
2012-03-06 00:09:35,714 - simpleExample - CRITICAL - critical message  
```
这里还要明确一点，logger对象是有继承关系的，比如名为a.b和a.c的logger都是名为a的子logger，并且所有的logger对象都继承于root。如果子对象没有添加handler等一些配置，会从父对象那继承。这样就可以通过这种继承关系来复用配置。
### 3. 多模块使用logging
logging模块保证在同一个python解释器内，多次调用logging.getLogger('log_name')都会返回同一个logger实例，即使是在多个模块的情况下。所以典型的多模块场景下使用logging的方式是在main模块中配置logging，这个配置会作用于多个的子模块，然后在其他模块中直接通过getLogger获取Logger对象即可。
        
这里使用上面配置文件：

```
[loggers]  
keys=root,main  
  
[handlers]  
keys=consoleHandler,fileHandler  
  
[formatters]  
keys=fmt  
  
[logger_root]  
level=DEBUG  
handlers=consoleHandler  
  
[logger_main]  
level=DEBUG  
qualname=main  
handlers=fileHandler  
  
[handler_consoleHandler]  
class=StreamHandler  
level=DEBUG  
formatter=fmt  
args=(sys.stdout,)  
  
[handler_fileHandler]  
class=logging.handlers.RotatingFileHandler  
level=DEBUG  
formatter=fmt  
args=('tst.log','a',20000,5,)  
  
[formatter_fmt]  
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s  
datefmt=  
```

主模块main.py：
```python
import logging  
import logging.config  
  
logging.config.fileConfig('logging.conf')  
root_logger = logging.getLogger('root')  
root_logger.debug('test root logger...')  
  
logger = logging.getLogger('main')  
logger.info('test main logger')  
logger.info('start import module \'mod\'...')  
import mod  
  
logger.debug('let\'s test mod.testLogger()')  
mod.testLogger()  
  
root_logger.info('finish test...')  
```

子模块mod.py：
```python
import logging  
import submod  
  
logger = logging.getLogger('main.mod')  
logger.info('logger of mod say something...')  
  
def testLogger():  
    logger.debug('this is mod.testLogger...')  
    submod.tst()  
```

子子模块submod.py：
```python
import logging  
  
logger = logging.getLogger('main.mod.submod')  
logger.info('logger of submod say something...')  
  
def tst():  
    logger.info('this is submod.tst()...')  
```
然后运行python main.py，控制台输出：
```
2012-03-09 18:22:22,793 - root - DEBUG - test root logger...  
2012-03-09 18:22:22,793 - main - INFO - test main logger  
2012-03-09 18:22:22,809 - main - INFO - start import module 'mod'...  
2012-03-09 18:22:22,809 - main.mod.submod - INFO - logger of submod say something...  
2012-03-09 18:22:22,809 - main.mod - INFO - logger say something...  
2012-03-09 18:22:22,809 - main - DEBUG - let's test mod.testLogger()  
2012-03-09 18:22:22,825 - main.mod - DEBUG - this is mod.testLogger...  
2012-03-09 18:22:22,825 - main.mod.submod - INFO - this is submod.tst()...  
2012-03-09 18:22:22,841 - root - INFO - finish test...  
```
可以看出，和预想的一样，然后在看一下tst.log，logger配置中的输出的目的地：
```
2012-03-09 18:22:22,793 - main - INFO - test main logger  
2012-03-09 18:22:22,809 - main - INFO - start import module 'mod'...  
2012-03-09 18:22:22,809 - main.mod.submod - INFO - logger of submod say something...  
2012-03-09 18:22:22,809 - main.mod - INFO - logger say something...  
2012-03-09 18:22:22,809 - main - DEBUG - let's test mod.testLogger()  
2012-03-09 18:22:22,825 - main.mod - DEBUG - this is mod.testLogger...  
2012-03-09 18:22:22,825 - main.mod.submod - INFO - this is submod.tst()... 
```
tst.log中没有root logger输出的信息，因为logging.conf中配置了只有main logger及其子logger使用RotatingFileHandler，而root logger是输出到标准输出。

FROM： http://blog.csdn.net/chosen0ne/article/details/7319306