title: pandas reset_index
date: 2016-9-8 10:19:56
tags: python
category: dev
---

```python
df1 = pandas.DataFrame( {
    "Name" : ["Alice", "Bob", "Mallory", "Mallory", "Bob" , "Mallory"] ,
    "City" : ["Seattle", "Seattle", "Portland", "Seattle", "Seattle", "Portland"] } )
```

```python
g1.add_suffix('_Count').reset_index()
```

```python
Out[21]:
      Name      City  City_Count  Name_Count
0    Alice   Seattle           1           1
1      Bob   Seattle           2           2
2  Mallory  Portland           2           2
3  Mallory   Seattle           1           1
```

```python
DataFrame({'count' : df1.groupby( [ "Name", "City"] ).size()}).reset_index()
```

```
      Name      City  count
0    Alice   Seattle      1
1      Bob   Seattle      2
2  Mallory  Portland      2
3  Mallory   Seattle      1
```
