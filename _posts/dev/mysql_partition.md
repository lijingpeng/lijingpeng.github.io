title: MySQL 分区表原理及使用详解
date: 2016-3-20 20:19:56
tags:
category: MySQL
---

## 1. 什么是表分区？

表分区，是指根据一定规则，将数据库中的一张表分解成多个更小的，容易管理的部分。从逻辑上看，只有一张表，但是底层却是由多个物理分区组成。

## 2. 表分区与分表的区别

分表：指的是通过一定规则，将一张表分解成多张不同的表。比如将用户订单记录根据时间成多个表。 分表与分区的区别在于：分区从逻辑上来讲只有一张表，而分表则是将一张表分解成多张表。

## 3. 表分区有什么好处？

1） 分区表的数据可以分布在不同的物理设备上，从而高效地利用多个硬件设备。 
2） 和单个磁盘或者文件系统相比，可以存储更多数据 
3） 优化查询。在where语句中包含分区条件时，可以只扫描一个或多个分区表来提高查询效率；涉及sum和count语句时，也可以在多个分区上并行处理，最后汇总结果。 
4） 分区表更容易维护。例如：想批量删除大量数据可以清除整个分区。 
5） 可以使用分区表来避免某些特殊的瓶颈，例如InnoDB的单个索引的互斥访问，ext3问价你系统的inode锁竞争等。

## 4. 分区表的限制因素

1） 一个表最多只能有1024个分区 
2） MySQL5.1中，分区表达式必须是整数，或者返回整数的表达式。在MySQL5.5中提供了非整数表达式分区的支持。 
3） 如果分区字段中有主键或者唯一索引的列，那么多有主键列和唯一索引列都必须包含进来。即：分区字段要么不包含主键或者索引列，要么包含全部主键和索引列。 
4） 分区表中无法使用外键约束 
5） MySQL的分区适用于一个表的所有数据和索引，不能只对表数据分区而不对索引分区，也不能只对索引分区而不对表分区，也不能只对表的一部分数据分区。

## 5. 如何判断当前MySQL是否支持分区？

命令：`show variables like '%partition%'` 运行结果:

<pre class="brush: sql; gutter: false; first-line: 1">mysql&gt; show variables like '%partition%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| have_partitioning | YES   |
+-------------------+-------+
1 row in set (0.00 sec)</pre>

have_partintioning 的值为YES，表示支持分区。

## 6. MySQL支持的分区类型有哪些？

1） RANGE分区：按照数据的区间范围分区 
2） LIST分区：按照List中的值分区，与RANGE的区别是，range分区的区间范围值是连续的。 
3） HASH分区 
4） KEY分区 **说明** 在MySQL5.1版本中，RANGE,LIST,HASH分区要求分区键必须是INT类型，或者通过表达式返回INT类型。但KEY分区的时候，可以使用其他类型的列（BLOB，TEXT类型除外）作为分区键。

## 7. Range分区

利用取值范围进行分区，区间要连续并且不能互相重叠。 语法：

<pre class="brush: sql; gutter: false; first-line: 1">partition by range(exp)( //exp可以为列名或者表达式，比如to_date(created_date)
    partition p0 values less than(num)
)</pre>

例如：

<pre class="brush: sql; gutter: false; first-line: 1">mysql&gt; create table emp(
    -&gt; id INT NOT null,
    -&gt; store_id int not null
    -&gt; )
    -&gt; partition by range(store_id)(
    -&gt;   partition p0 values less than(10),
    -&gt;   partition p1 values less than(20)
    -&gt; );</pre>

上面的语句创建了emp表，并根据store_id字段进行分区，小于10的值存在分区p0中，大于等于10，小于20的值存在分区p1中。 **注意** 每个分区都是按顺序定义的，从最低到最高。上面的语句，如果将less than(10) 和less than (20)的顺序颠倒过来，那么将报错，如下：

<pre class="brush: bash; gutter: false; first-line: 1">ERROR 1493 (HY000): VALUES LESS THAN value must be strictly increasing for each partition</pre>

**RANGE分区存在的问题**

1. range范围覆盖问题：当插入的记录中对应的分区键的值不在分区定义的范围中的时候，插入语句会失败。 上面的例子，如果我插入一条store_id = 30的记录会怎么样呢？ 我们上面分区的时候，最大值是20，如果插入一条超过20的记录，会报错:
<pre class="brush: bash; gutter: false; first-line: 1">mysql&gt; insert into emp(id,store_id) values(2,30);
ERROR 1526 (HY000): Table has no partition for value 30</pre>

提示30这个值没有对应的分区。 **解决办法** A. 预估分区键的值，及时新增分区。 B. 设置分区的时候，使用`values less than maxvalue` 子句,MAXVALUE表示最大的可能的整数值。 C. 尽量选择能够全部覆盖的字段作为分区键，比如一年的十二个月等。

2. Range分区中，分区键的值如果是NULL，将被作为一个最小值来处理。

## 8. LIST分区

<p>List分区是建立离散的值列表告诉数据库特定的值属于哪个分区。 语法：

<pre class="brush: sql; gutter: false; first-line: 1">   partition by list(exp)( //exp为列名或者表达式
        partition p0 values in (3,5)  //值为3和5的在p0分区
    )</pre>

与Range不同的是，list分区不必生命任何特定的顺序。例如：

<pre class="brush: sql; gutter: false; first-line: 1">mysql&gt; create table emp1(
    -&gt; id int not null,
    -&gt; store_id int not null
    -&gt; )
    -&gt; partition by list(store_id)(
    -&gt;   partition p0 values in (3,5),
    -&gt;   partition p1 values in (2,6,7,9)
    -&gt; );</pre>

**注意** 如果插入的记录对应的分区键的值不在list分区指定的值中，将会插入失败。并且，list不能像range分区那样提供maxvalue。

## 9. Columns分区

MySQL5.5中引入的分区类型，解决了5.5版本之前range分区和list分区只支持整数分区的问题。 Columns分区可以细分为 range columns分区和 list columns分区，他们都支持整数，日期时间，字符串三大数据类型。（不支持text和blob类型作为分区键） columns分区还支持多列分区（这里不详细展开）。

## 10. Hash分区

Hash分区主要用来分散热点读，确保数据在预先确定个数的分区中尽可能平均分布。 MySQL支持两种Hash分区:常规Hash分区和线性Hash分区。 A. 常规Hash分区:使用取模算法 语法：

<pre class="brush: sql; gutter: false; first-line: 1">partition by hash(store_id) partitions 4;</pre>

上面的语句，根据store_id对4取模，决定记录存储位置。 比如store_id = 234的记录，MOD(234,4)=2,所以会被存储在第二个分区。

**常规Hash分区的优点和不足** 优点：能够使数据尽可能的均匀分布。 缺点：不适合分区经常变动的需求。假如我要新增加两个分区，现在有6个分区，那么MOD(234,6)的结果与之前MOD(234,4)的结果就会出现不一致，这样大部分数据就需要重新计算分区。为解决此问题，MySQL提供了线性Hash分区。

B. 线性Hash分区：分区函数是一个线性的2的幂的运算法则。 语法：

<pre class="brush: bash; gutter: false; first-line: 1">partition by LINER hash(store_id) partitions 4;</pre>

与常规Hash的不同在于，“Liner”关键字。 算法介绍: 假设要保存记录的分区编号为N,num为一个非负整数,表示分割成的分区的数量，那么N可以通过以下步骤得到：

Step 1. 找到一个大于等于num的2的幂，这个值为V，V可以通过下面公式得到：

V = Power(2,Ceiling(Log(2,num)))

例如：刚才设置了4个分区，num=4，Log(2,4)=2,Ceiling(2)=2,power(2,2)=4,即V=4

Step 2. 设置N=F(column_list)&amp;(V-1)

例如：刚才V=4，store_id=234对应的N值，N = 234&amp;（4-1） =2

Step 3. 当N&gt;=num,设置V=Ceiling(V/2),N=N&amp;(V-1)

例如：store_id=234,N=2&lt;4,所以N就取值2，即可。

假设上面算出来的N=5，那么V=Ceiling(2.5)=3,N=234&amp;(3-1)=1,即在第一个分区。

**线性Hash的优点和不足** 优点：在分区维护（增加，删除，合并，拆分分区）时，MySQL能够处理得更加迅速。 缺点：与常规Hash分区相比，线性Hash各个分区之间的数据分布不太均衡。

## 11. Key分区

类似Hash分区，Hash分区允许使用用户自定义的表达式，但Key分区不允许使用用户自定义的表达式。Hash仅支持整数分区，而Key分区支持除了Blob和text的其他类型的列作为分区键。 语法:

<pre class="brush: bash; gutter: false; first-line: 1">partition by key(exp) partitions 4;//exp是零个或多个字段名的列表</pre>

key分区的时候，exp可以为空，如果为空，则默认使用主键作为分区键，没有主键的时候，会选择非空惟一键作为分区键。

## 12. 子分区

分区表中对每个分区再次分割，又成为复合分区。

## 13. 分区对于NULL值的处理

MySQ允许分区键值为NULL，分区键可能是一个字段或者一个用户定义的表达式。一般情况下，MySQL在分区的时候会把NULL值当作零值或者一个最小值进行处理。

**注意**

Range分区中：NULL值被当作最小值来处理

List分区中：NULL值必须出现在列表中，否则不被接受

Hash/Key分区中：NULL值会被当作零值来处理

## 14. 分区管理

分区管理包括对于分区的增加，删除，以及查询。

**增加分区：**

对于Range分区和LIst分区来说：

<pre class="brush: sql; gutter: false; first-line: 1">alter table table_name add partition (partition p0 values ...(exp))</pre>

values后面的内容根据分区的类型不同而不同。

对于Hash分区和Key分区来说：

<pre class="brush: sql; gutter: false; first-line: 1">alter table table_name add partition partitions 8;</pre>

上面的语句，指的是新增8个分区 。

**删除分区**

对于Range分区和List分区：

<pre class="brush: sql; gutter: false; first-line: 1">alter table table_name drop partition p0; //p0为要删除的分区名称</pre>

删除了分区，同时也将删除该分区中的所有数据。同时，如果删除了分区导致分区不能覆盖所有值，那么插入数据的时候会报错。

对于Hash和Key分区：

<pre class="brush: sql; gutter: false; first-line: 1">alter table table_name coalesce partition 2; //将分区缩减到2个</pre>

**coalesce [ˌkəʊəˈles] vi. 联合，合并**

分区查询 1）查询某张表一共有多少个分区

<pre class="brush: sql; gutter: false; first-line: 1">mysql&gt; select 
 -&gt;   partition_name,
 -&gt;   partition_expression,
 -&gt;   partition_description,
 -&gt;   table_rows
 -&gt; from 
 -&gt;   INFORMATION_SCHEMA.partitions
 -&gt; where
 -&gt;   table_schema='test'
 -&gt;   and table_name = 'emp';
+----------------+----------------------+-----------------------+------------+
| partition_name | partition_expression | partition_description | table_rows |
+----------------+----------------------+-----------------------+------------+
| p0             | store_id             | 10                    |          0 |
| p1             | store_id             | 20                    |          1 |
+----------------+----------------------+-----------------------+------------+</pre>

即，可以从information_schema.partitions表中查询。

2）查看执行计划，判断查询数据是否进行了分区过滤

<pre class="brush: sql; gutter: false; first-line: 1">mysql&gt; explain partitions select * from emp where store_id=10 \G;
*************************** 1. row ***************************
        id: 1
select_type: SIMPLE
     table: emp
partitions: p1
      type: system
possible_keys: NULL
       key: NULL
   key_len: NULL
       ref: NULL
      rows: 1
     Extra: 
1 row in set (0.00 sec)</pre>

上面的结果：partitions:p1 表示数据在p1分区进行检索。