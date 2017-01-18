title: Elasticsearch-索引优化
date: 2016-10-10 10:19:56
tags:
category: dev
---

ES索引优化:

ES主要是用tranlog进行各节点之间的数据平衡。所以从上我可以通过索引的settings进行第一优化：
```
"index.translog.flush_threshold_ops": "100000"
"index.refresh_interval": "-1"
```
这两个参数第一是到tranlog数据达到多少条进行平衡，默认为5000，而这个过程相对而言是比较浪费时间和资源的。所以我们可以将这个值调大一些还是设为-1关闭，进而手动进行tranlog平衡。
第二参数是刷新频率，默认为120s是指索引在生命周期内定时刷新，一但有数据进来能refresh像lucene里面commit,我们知道当数据addDoucment会，还不能检索到要commit之后才能行数据的检索所以可以将其关闭，在最初索引完后手动refresh一之，然后将索引setting里面的index.refresh_interval参数按需求进行修改，从而可以提高索引过程效率。

另外的知道ES索引过程中如果有副本存在，数据也会马上同步到副本中去。我个人建议在索引过程中将副本数设为0，待索引完成后将副本数按需量改回来，这样也可以提高索引效率。
```
"number_of_replicas": 0
```

接下来还可以尝试修改以下参数：

增大 threadpool.index.queue_size
增大 indices.memory.index_buffer_size
增大 index.translog.flush_threshold_ops
增大 index.translog.sync_interval
增大 index.engine.robin.refresh_interval

[官方WIKI](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html)
