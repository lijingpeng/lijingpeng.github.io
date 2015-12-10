date: 2015-2-6
title: 云梯表Join的倾斜问题以及解决方法
tags: 
category: SQL
---

### 什么是倾斜问题, 问题症状

写HQL语句的时候常常会遇到表Join的情况，一个简单的Join会被Hive解释成一个MapReduce任务，Map端分别读取两个表的数据，Reduce做真正的Join操作。如果执行的过程中，如果发现有些Reduce任务比其他的Reduce任务慢很多，往往是发生了倾斜问题。

### 问题分析

举个栗子：
```SQL
select
    a.*,
    b.cat_name
from
    dim_auction a
join
    dim_category b
on a.cat_id=b.cat_id
```
Join会被Hive解释成一个MapReduce任务时，Map端输出的记录是以Join的条件为Key的，即这些Map生成的Key都是 cat_id。随后，这些cat_id被Hash到多个Reduce任务中，来完成真正Join。所有拥有相同cat_id的记录一定会被分配到同一个 Reducer中。

但是每个cat_id多对应的记录数是不一样的，连衣裙类目的数据一定很多，钟点工类目的数据就很少。如果某个Reducer比较悲催，分到了连衣裙类目，则其处理的数据量就会很大，最后表现在处理时间拖后腿的情况。

#### 解决方法1 — MapJoin

一个常用的解决手段是使用MapJoin，这种手段适合于关联的两个表有一个较小的情况。其原理是，把Join动作提前到Map端，而不是Reduce端。在Map的时候，对于大表，我们还是每个Map装载这个表的一部分，对于小表，我们把它放到每个Map中。这样每个Map都拥有小表的所有记录，可以在本地进行Join操作了。具体的，在SQL中加入这样一个Hint就OK了：
```SQL
select /*\+ MAPJOIN(b) \*/
   a.*,
   b.cat_name
from
   dim_auction a
join
   dim_category b
on a.cat_id=b.cat_id
```

#### 解决方法2 —分而治之

MapJoin是一个很好用的工具，但是却存在一个致命的弱点，就是其中一个表一定要比较小，能够完全装入单台机器的内存。

我们看下面一个例子：
```SQL
select
   a.*,
   b.property_name
from
   auction_property a
left outer join
(
   select
      *
   from
      dim_base_properties
   where ds='20130620'
) b
on a.property_id = b.property_id
```
auction_property表存储了每一个商品以及该商品的每一个属性。
dim_base_properties存储了每个属性的名称、以及一些其他元数据。
我们这次关联是想在auction_property表的基础上，加上每个属性的名称。和类目一样，属性ID是有倾斜的，即有一些很常用的属性，被很多宝贝都引用了。悲剧的事情是，auction_property和dim_base_properties这两个表都很大。。。

解决这个问题依赖于如下的观察：导致倾斜的Key的个数往往不多，也就是说，常用的属性就那么几个，剩下的大部分属性都不常用。

下面我们采用分治方法来解决这个问题：
第一步，找到的常用的属性。
```SQL
create table lingyun_property_skew
as
select
    a.property_id,
    a.cnt,
    b.property_name
from
    (
        select
           property_id,
           count(*) as cnt
        from
            lingyun_auction_property a
        group by property_id
        order by cnt desc
        limit 1000
    ) a
join
    (
        select
            *
        from
            tbdw.dim_base_properties
        where ds='20130620'
    ) b
on a.property_id = b.property_id;
```
这里我们把auction_property表按照property_id汇总，并且找到最常用的1000个属性ID，并且查到了这些属性ID的名字。这里1000可以扩展到上万个，只要保障能够被装进内存就可以了。

第二步是先解决常用属性的关联
```SQL
create table lingyun_auction_property_name_temp
as
select /*+ MAPJOIN(b) */
    a.*,
    b.property_name
from
    auction_property a
left outer join
    lingyun_property_skew b
on a.property_id = b.property_id;

create table lingyun_auction_property_name_part1
as
select
    *
from
    lingyun_auction_property_name_temp
where property_name is not null;
```
这一步我们可以使用MAPJOIN是因为lingyun_property_skew是一个小表。

第三步是解决非常用属性的关联
```SQL
create table lingyun_auction_property_name_part2
as
select
    a.auction_id,
    a.property_id,
    a.value_id,
    b.property_name
from
    (
        select
            *
        from
            lingyun_auction_property_name_temp
        where property_name is null
    ) a
join
    (
        select
            *
        from
            tbdw.dim_base_properties
        where ds='20130620'
    ) b
on a.property_id=b.property_id;
```
这一步不存在倾斜问题是因为可能导致倾斜的property_id已经从lingyun_auction_property_name_temp里面筛除掉了，剩下的每个property_id对应的商品数不会很多。

最后把两个表Union到一起，得到最终结果。
```SQL
create table lingyun_auction_property_name
as
select
    *
from
(
    select
        auction_id,
        property_id,
        value_id,
        property_name
    from
        lingyun_auction_property_name_part1

    union all

    select
        auction_id,
        property_id,
        value_id,
        property_name
    from
        lingyun_auction_property_name_part2
) a;
```

能不能再给力一点？

到这里问题就解决了，但是写这些代码太复杂了，能不能再给力一点呢？
从上面的代码可以看出，整个流程是有规律可循的，可以被推到幕后的。
具体的，可以把这个逻辑放进Hive的执行计划器中，我们只需要声明需要对哪个表分治，分出来的小表有多大就可以了，例如：
```SQL
select /*+ SPLITJOIN(b, 1000) */
   a.*,
   b.property_name
from
   auction_property a
left outer join
(
   select
      *
   from
      dim_base_properties
   where ds='20130620'
) b
on a.property_id = b.property_id
```