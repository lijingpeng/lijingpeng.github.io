title: Python mysql 编码问题
date: 2016-3-18 12:19:56
tags: Mysql
category: dev
---

```
'latin-1' codec can't encode characters in position 0-1: ordinal not in range(256)
```

解决方案：
```python
db.set_character_set('utf8')
dbc.execute('SET NAMES utf8;')
dbc.execute('SET CHARACTER SET utf8;')
dbc.execute('SET character_set_connection=utf8;')
```