date: 2015-3-3
title: MapReduce 计数器简介
tags: MapReduce
category: machine learning
---

## 1、计数器 简介
---

在许多情况下，一个用户需要了解待分析的数据，尽管这并非所要执行的分析任务 的核心内容。以统计数据集中无效记录数目的任务为例，如果发现无效记录的比例 相当高，那么就需要认真思考为何存在如此多无效记录。是所采用的检测程序存在 缺陷，还是数据集质量确实很低，包含大量无效记录？如果确定是数据集的质量问 题，则可能需要扩大数据集的规模，以增大有效记录的比例，从而进行有意义的分析。

计数器是一种收集作业统计信息的有效手段，用于质量控制或应用级统计。计数器 还可辅助诊断系统故障。如果需要将日志信息传输到map或reduce任务，更好的 方法通常是尝试传输计数器值以监测某一特定事件是否发生。对于大型分布式作业 而言，使用计数器更为方便。首先，获取计数器值比输出日志更方便，其次，根据 计数器值统计特定事件的发生次数要比分析一堆日志文件容易得多。

## 2 、内置计数器
---

Hadoop为每个作业维护若干内置计数器, 以描述该作业的各项指标。例如，某些计数器记录已处理的字节数和记录数，使用户可监控已处理的输入数据量和已产生的输出数据量，并以此对 job 做适当的优化。
```
14/06/08 15:13:35 INFO mapreduce.Job: Counters: 46
  File System Counters
  FILE: Number of bytes read=159
  FILE: Number of bytes written=159447
  FILE: Number of read operations=0
  FILE: Number of large read operations=0
  FILE: Number of write operations=0
  HDFS: Number of bytes read=198
  HDFS: Number of bytes written=35
  HDFS: Number of read operations=6
  HDFS: Number of large read operations=0
  HDFS: Number of write operations=2
  Job Counters 
  Launched map tasks=1
  Launched reduce tasks=1
  Rack-local map tasks=1
  Total time spent by all maps in occupied slots (ms)=3896
  Total time spent by all reduces in occupied slots (ms)=9006
  Map-Reduce Framework
  Map input records=3
  Map output records=12
  Map output bytes=129
  Map output materialized bytes=159
  Input split bytes=117
  Combine input records=0
  Combine output records=0
  Reduce input groups=4
  Reduce shuffle bytes=159
  Reduce input records=12
  Reduce output records=4
  Spilled Records=24
  Shuffled Maps =1
  Failed Shuffles=0
  Merged Map outputs=1
  GC time elapsed (ms)=13
  CPU time spent (ms)=3830
  Physical memory (bytes) snapshot=537718784
  Virtual memory (bytes) snapshot=7365263360
  Total committed heap usage (bytes)=2022309888
  Shuffle Errors
  BAD_ID=0
  CONNECTION=0
  IO_ERROR=0
  WRONG_LENGTH=0
  WRONG_MAP=0
  WRONG_REDUCE=0
  File Input Format Counters 
  Bytes Read=81
  File Output Format Counters 
  Bytes Written=35
```
计数器由其关联任务维护，并定期传到tasktracker，再由tasktracker传给 jobtracker.因此，计数器能够被全局地聚集。详见第 hadoop 权威指南第170页的“进度和状态的更新”小节。与其他计数器（包括用户定义的计数器）不同，内置的作业计数器实际上 由jobtracker维护，不必在整个网络中发送。
一个任务的计数器值每次都是完整传输的，而非自上次传输之后再继续数未完成的传输，以避免由于消息丢失而引发的错误。另外，如果一个任务在作业执行期间失 败，则相关计数器值会减小。仅当一个作业执行成功之后，计数器的值才是完整可 靠的。

## 3、用户定义的Java计数器

MapReduce允许用户编写程序来定义计数器，计数器的值可在mapper或reducer 中增加。多个计数器由一个Java枚举(enum)类型来定义，以便对计数器分组。一 个作业可以定义的枚举类型数量不限，各个枚举类型所包含的字段数量也不限。枚 举类型的名称即为组的名称，枚举类型的字段就是计数器名称。计数器是全局的。 换言之，MapReduce框架将跨所有map和reduce聚集这些计数器，并在作业结束 时产生一个最终结果。

Note1： 需要说明的是，不同的 hadoop 版本定义的方式会有些许差异。

（1）在0.20.x版本中使用counter很简单,直接定义即可，如无此counter，hadoop会自动添加此counter.
```java
Counter ct = context.getCounter(“INPUT_WORDS”, “count”);
ct.increment(1);
```
（2）在0.19.x版本中,需要定义enum
```java
enum MyCounter {INPUT_WORDS };
reporter.incrCounter(MyCounter.INPUT_WORDS, 1);
RunningJob job = JobClient.runJob(conf);
Counters c = job.getCounters();
long cnt = c.getCounter(MyCounter.INPUT_WORDS);
```

Notice2： 使用计数器需要清楚的是它们都存储在jobTracker的内存里。 Mapper/Reducer 任务序列化它们，连同更新状态被发送。为了运行正常且jobTracker不会出问题，计数器的数量应该在10-100个，计数器不仅仅只用来聚合MapReduce job的统计值。新版本的hadoop限制了计数器的数量，以防给jobTracker带来损害。你最不想看到的事情就是由于定义上百个计数器而使jobTracker宕机。

下面咱们来看一个计数器的实例（以下代码请运行在 0.20.1 版本以上）：

3.1 测试数据：

hello world 2013 mapreduce
hello world 2013 mapreduce
hello world 2013 mapreduce

3.2 代码：

```java
/**
 * Project Name:CDHJobs
 * File Name:MapredCounter.java
 * Package Name:tmp
 * Date:2014-6-8下午2:12:48
 * Copyright (c) 2014, decli#qq.com All Rights Reserved.
 *
 */

package tmp;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Counter;
import org.apache.hadoop.mapreduce.CounterGroup;
import org.apache.hadoop.mapreduce.Counters;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountWithCounter {

  static enum WordsNature {
    STARTS_WITH_DIGIT, STARTS_WITH_LETTER, ALL
  }

  /**
   * The map class of WordCount.
   */
  public static class TokenCounterMapper extends Mapper<Object, Text, Text, IntWritable> {

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  /**
   * The reducer class of WordCount
   */
  public static class TokenCounterReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException,
        InterruptedException {
      int sum = 0;

      String token = key.toString();
      if (StringUtils.isNumeric(token)) {
        context.getCounter(WordsNature.STARTS_WITH_DIGIT).increment(1);
      } else if (StringUtils.isAlpha(token)) {
        context.getCounter(WordsNature.STARTS_WITH_LETTER).increment(1);
      }
      context.getCounter(WordsNature.ALL).increment(1);

      for (IntWritable value : values) {
        sum += value.get();
      }
      context.write(key, new IntWritable(sum));
    }
  }

  /**
   * The main entry point.
   */
  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = new Job(conf, "WordCountWithCounter");
    job.setJarByClass(WordCountWithCounter.class);
    job.setMapperClass(TokenCounterMapper.class);
    job.setReducerClass(TokenCounterReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path("/tmp/dsap/rawdata/june/a.txt"));
    FileOutputFormat.setOutputPath(job, new Path("/tmp/dsap/rawdata/june/a_result"));
    int exitCode = job.waitForCompletion(true) ? 0 : 1;

    Counters counters = job.getCounters();
    Counter c1 = counters.findCounter(WordsNature.STARTS_WITH_DIGIT);
    System.out.println("-------------->>>>: " + c1.getDisplayName() + ": " + c1.getValue());

    // The below example shows how to get built-in counter groups that Hadoop provides basically.
    for (CounterGroup group : counters) {
      System.out.println("==========================================================");
      System.out.println("* Counter Group: " + group.getDisplayName() + " (" + group.getName() + ")");
      System.out.println("  number of counters in this group: " + group.size());
      for (Counter counter : group) {
        System.out.println("  ++++ " + counter.getDisplayName() + ": " + counter.getName() + ": "
            + counter.getValue());
      }
    }
    System.exit(exitCode);
  }
}
```
3.3 结果与计数器详解

运行结果下面会一并给出。Counter有”组group”的概念，用于表示逻辑上相同范围的所有数值。MapReduce job提供的默认Counter分为7个组，下面逐一介绍。这里也拿上面的测试数据来做详细比对，我将会针对具体的计数器，挑选一些主要的简述一下。
```
... 前面省略 job 运行信息 xx 字 ...
  ALL=4
  STARTS_WITH_DIGIT=1
  STARTS_WITH_LETTER=3
-------------->>>>: STARTS_WITH_DIGIT: 1
==========================================================
#MapReduce job执行所依赖的数据来自于不同的文件系统，这个group表示job与文件系统交互的读写统计 
* Counter Group: File System Counters (org.apache.hadoop.mapreduce.FileSystemCounter)
 number of counters in this group: 10
 #job读取本地文件系统的文件字节数。假定我们当前map的输入数据都来自于HDFS，那么在map阶段，这个数据应该是0。但reduce在执行前，它 的输入数据是经过shuffle的merge后存储在reduce端本地磁盘中，所以这个数据就是所有reduce的总输入字节数。
 ++++ FILE: Number of bytes read: FILE_BYTES_READ: 159
 #map的中间结果都会spill到本地磁盘中，在map执行完后，形成最终的spill文件。所以map端这里的数据就表示map task往本地磁盘中总共写了多少字节。与map端相对应的是，reduce端在shuffle时，会不断地拉取map端的中间结果，然后做merge并 不断spill到自己的本地磁盘中。最终形成一个单独文件，这个文件就是reduce的输入文件。 
 ++++ FILE: Number of bytes written: FILE_BYTES_WRITTEN: 159447
 ++++ FILE: Number of read operations: FILE_READ_OPS: 0
 ++++ FILE: Number of large read operations: FILE_LARGE_READ_OPS: 0
 ++++ FILE: Number of write operations: FILE_WRITE_OPS: 0
 # 整个job执行过程中，只有map端运行时，才从HDFS读取数据，这些数据不限于源文件内容，还包括所有map的split元数据。所以这个值应该比FileInputFormatCounters.BYTES_READ 要略大些。 
 ++++ HDFS: Number of bytes read: HDFS_BYTES_READ: 198
 #Reduce的最终结果都会写入HDFS，就是一个job执行结果的总量。 
 ++++ HDFS: Number of bytes written: HDFS_BYTES_WRITTEN: 35
 ++++ HDFS: Number of read operations: HDFS_READ_OPS: 6
 ++++ HDFS: Number of large read operations: HDFS_LARGE_READ_OPS: 0
 ++++ HDFS: Number of write operations: HDFS_WRITE_OPS: 2
==========================================================
#这个group描述与job调度相关的统计 
* Counter Group: Job Counters (org.apache.hadoop.mapreduce.JobCounter)
 number of counters in this group: 5
 #Job在被调度时，如果启动了一个data-local(源文件的幅本在执行map task的taskTracker本地) 
 ++++ Data-local map tasks 
 #当前job为某些map task的执行保留了slot，总共保留的时间是多少 
 ++++ FALLOW_SLOTS_MILLIS_MAPS/REDUCES
 #所有map task占用slot的总时间，包含执行时间和创建/销毁子JVM的时间
 ++++ SLOTS_MILLIS_MAPS/REDUCES
 # 此job启动了多少个map task 
 ++++ Launched map tasks: TOTAL_LAUNCHED_MAPS: 1
 # 此job启动了多少个reduce task 
 ++++ Launched reduce tasks: TOTAL_LAUNCHED_REDUCES: 1
 ++++ Rack-local map tasks: RACK_LOCAL_MAPS: 1
 ++++ Total time spent by all maps in occupied slots (ms): SLOTS_MILLIS_MAPS: 3896
 ++++ Total time spent by all reduces in occupied slots (ms): SLOTS_MILLIS_REDUCES: 9006
==========================================================
#这个Counter group包含了相当多地job执行细节数据。这里需要有个概念认识是：一般情况下，record就表示一行数据，而相对地byte表示这行数据的大小是 多少，这里的group表示经过reduce merge后像这样的输入形式{"aaa", [5, 8, 2, …]}。 
* Counter Group: Map-Reduce Framework (org.apache.hadoop.mapreduce.TaskCounter)
 number of counters in this group: 20
 #所有map task从HDFS读取的文件总行数 
 ++++ Map input records: MAP_INPUT_RECORDS: 3
 #map task的直接输出record是多少，就是在map方法中调用context.write的次数，也就是未经过Combine时的原生输出条数 
 ++++ Map output records: MAP_OUTPUT_RECORDS: 12
 # Map的输出结果key/value都会被序列化到内存缓冲区中，所以这里的bytes指序列化后的最终字节之和 
 ++++ Map output bytes: MAP_OUTPUT_BYTES: 129
 ++++ Map output materialized bytes: MAP_OUTPUT_MATERIALIZED_BYTES: 159
 # #与map task 的split相关的数据都会保存于HDFS中，而在保存时元数据也相应地存储着数据是以怎样的压缩方式放入的，它的具体类型是什么，这些额外的数据是 MapReduce框架加入的，与job无关，这里记录的大小就是表示额外信息的字节大小
 ++++ Input split bytes: SPLIT_RAW_BYTES: 117
 #Combiner是为了减少尽量减少需要拉取和移动的数据，所以combine输入条数与map的输出条数是一致的。
 ++++ Combine input records: COMBINE_INPUT_RECORDS: 0
 # 经过Combiner后，相同key的数据经过压缩，在map端自己解决了很多重复数据，表示最终在map端中间文件中的所有条目数 
 ++++ Combine output records: COMBINE_OUTPUT_RECORDS: 0
 #Reduce总共读取了多少个这样的groups 
 ++++ Reduce input groups: REDUCE_INPUT_GROUPS: 4
 #Reduce端的copy线程总共从map端抓取了多少的中间数据，表示各个map task最终的中间文件总和 
 ++++ Reduce shuffle bytes: REDUCE_SHUFFLE_BYTES: 159
 #如果有Combiner的话，那么这里的数值就等于map端Combiner运算后的最后条数，如果没有，那么就应该等于map的输出条数 
 ++++ Reduce input records: REDUCE_INPUT_RECORDS: 12
 #所有reduce执行后输出的总条目数 
 ++++ Reduce output records: REDUCE_OUTPUT_RECORDS: 4
 #spill过程在map和reduce端都会发生，这里统计在总共从内存往磁盘中spill了多少条数据 
 ++++ Spilled Records: SPILLED_RECORDS: 24
 #每个reduce几乎都得从所有map端拉取数据，每个copy线程拉取成功一个map的数据，那么增1，所以它的总数基本等于 reduce number * map number 
 ++++ Shuffled Maps : SHUFFLED_MAPS: 1
 # copy线程在抓取map端中间数据时，如果因为网络连接异常或是IO异常，所引起的shuffle错误次数 
 ++++ Failed Shuffles: FAILED_SHUFFLE: 0
 #记录着shuffle过程中总共经历了多少次merge动作 
 ++++ Merged Map outputs: MERGED_MAP_OUTPUTS: 1
 #通过JMX获取到执行map与reduce的子JVM总共的GC时间消耗 
 ++++ GC time elapsed (ms): GC_TIME_MILLIS: 13
 ++++ CPU time spent (ms): CPU_MILLISECONDS: 3830
 ++++ Physical memory (bytes) snapshot: PHYSICAL_MEMORY_BYTES: 537718784
 ++++ Virtual memory (bytes) snapshot: VIRTUAL_MEMORY_BYTES: 7365263360
 ++++ Total committed heap usage (bytes): COMMITTED_HEAP_BYTES: 2022309888
==========================================================
#这组内描述Shuffle过程中的各种错误情况发生次数，基本定位于Shuffle阶段copy线程抓取map端中间数据时的各种错误。
* Counter Group: Shuffle Errors (Shuffle Errors)
 number of counters in this group: 6
 #每个map都有一个ID，如attempt_201109020150_0254_m_000000_0，如果reduce的copy线程抓取过来的元数据中这个ID不是标准格式，那么此Counter增加 
 ++++ BAD_ID: BAD_ID: 0
 #表示copy线程建立到map端的连接有误 
 ++++ CONNECTION: CONNECTION: 0
 #Reduce的copy线程如果在抓取map端数据时出现IOException，那么这个值相应增加 
 ++++ IO_ERROR: IO_ERROR: 0
 #map端的那个中间结果是有压缩好的有格式数据，所有它有两个length信息：源数据大小与压缩后数据大小。如果这两个length信息传输的有误(负值)，那么此Counter增加
 ++++ WRONG_LENGTH: WRONG_LENGTH: 0
 #每个copy线程当然是有目的:为某个reduce抓取某些map的中间结果，如果当前抓取的map数据不是copy线程之前定义好的map，那么就表示把数据拉错了
 ++++ WRONG_MAP: WRONG_MAP: 0
 #与上面描述一致，如果抓取的数据表示它不是为此reduce而准备的，那还是拉错数据了。 
 ++++ WRONG_REDUCE: WRONG_REDUCE: 0
==========================================================
#这个group表示map task读取文件内容(总输入数据)的统计 
* Counter Group: File Input Format Counters (org.apache.hadoop.mapreduce.lib.input.FileInputFormatCounter)
 number of counters in this group: 1
# Map task的所有输入数据(字节)，等于各个map task的map方法传入的所有value值字节之和。 
 ++++ Bytes Read: BYTES_READ: 81
==========================================================
##这个group表示reduce task输出文件内容(总输出数据)的统计 
* Counter Group: File Output Format Counters (org.apache.hadoop.mapreduce.lib.output.FileOutputFormatCounter)
 number of counters in this group: 1
 ++++ Bytes Written: BYTES_WRITTEN: 35
==========================================================
# 自定义计数器的统计
* Counter Group: tmp.WordCountWithCounter$WordsNature (tmp.WordCountWithCounter$WordsNature)
 number of counters in this group: 3
 ++++ ALL: ALL: 4
 ++++ STARTS_WITH_DIGIT: STARTS_WITH_DIGIT: 1
 ++++ STARTS_WITH_LETTER: STARTS_WITH_LETTER: 3
```

4、最后的问题：

如果想要在 MapReduce 中实现一个类似计数器的“全局变量”，可以在 map、reduce 中以任意数据类型、任意修改变量值，并在 main 函数中回调获取该怎么办呢？

5、 Refer:

（1）An Example of Hadoop MapReduce Counter

http://diveintodata.org/2011/03/15/an-example-of-hadoop-mapreduce-counter/

（2）Hadoop Tutorial Series, Issue #3: Counters In Action

http://www.philippeadjiman.com/blog/2010/01/07/hadoop-tutorial-series-issue-3-counters-in-action/

（3）Controlling Hadoop MapReduce Job recursion

http://codingwiththomas.blogspot.com/2011/04/controlling-hadoop-job-recursion.html

（4）MapReduce Design Patterns（chapter 2 （part 3））（四）

http://blog.csdn.net/cuirong1986/article/details/8456923

（5）[hadoop源码阅读][5]-counter的使用和默认counter的含义

http://www.cnblogs.com/xuxm2007/archive/2012/06/15/2551030.html