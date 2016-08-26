title: Python编码问题
date: 2016-5-23 10:19:56
tags: python
category: dev
---

解决Python2.7的UnicodeEncodeError: ‘ascii’ codec can’t encode异常错误

```python
import sys
reload(sys)
print sys.getdefaultencoding()
sys.setdefaultencoding('utf-8')
```