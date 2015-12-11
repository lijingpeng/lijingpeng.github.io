title: Hive UDF开发
date: 2015-09-03 12:49:45
tags:
category: Hive
---
Hive进行UDAF开发，相对要比UDF复杂一些，不过也不是很难。请看一个例子:
```java
package org.hrj.hive.udf;

import org.apache.hadoop.hive.ql.exec.UDAFEvaluator;
import org.apache.hadoop.hive.serde2.io.DoubleWritable;

public class UDAFSum_Sample extends NumericUDAF {
    public static class Evaluator implements UDAFEvaluator {
        private boolean mEmpty;
        private double mSum;
        public Evaluator() {
            super();
            init();
        }

        public void init() {
            mSum = 0;
            mEmpty = true;
        }

        public boolean iterate(DoubleWritable o) {
            if (o != null) {
                mSum += o.get();
                mEmpty = false;
            }
            return true;
        }

        public DoubleWritable terminatePartial() {
            // This is SQL standard - sum of zero items should be null.
            return mEmpty ? null : new DoubleWritable(mSum);
        }

        public boolean merge(DoubleWritable o) {
            if (o != null) {
                mSum += o.get();
                mEmpty = false;
            }
            return true;
        }

        public DoubleWritable terminate() {
            // This is SQL standard - sum of zero items should be null.
            return mEmpty ? null : new DoubleWritable(mSum);
        }
    }
}
```
1.将java文件编译成Sum_Sample.jar

2.进入hive
```bash

hive> add jar Sum_sample.jar;

hive> create temporary function sum_test as 'com.hrj.hive.udf.UDAFSum_Sample';

hive> select sum_test(t.num) from t;

hive> drop temporary function sum_test;

hive> quit;
```

关于UDAF开发注意点：

1.需要import org.apache.hadoop.hive.ql.exec.UDAF以及org.apache.hadoop.hive.ql.exec.UDAFEvaluator,这两个包都是必须的

2.函数类需要继承UDAF类，内部类Evaluator实现UDAFEvaluator接口

3.Evaluator需要实现 init、iterate、terminatePartial、merge、terminate这几个函数

    1）init函数类似于构造函数，用于UDAF的初始化

    2）iterate接收传入的参数，并进行内部的轮转。其返回类型为boolean

    3）terminatePartial无参数，其为iterate函数轮转结束后，返回乱转数据，iterate和terminatePartial类似于hadoop的Combiner

    4）merge接收terminatePartial的返回结果，进行数据merge操作，其返回类型为boolean

    5）terminate返回最终的聚集函数结果
