{{indexmenu_n>4}}

# Spark开发指南

## 1\. Spark简介

Spark是一个基于内存计算的开源的集群计算系统，相对于MapReduce，Spark使用了更为快速的计算引擎，可以更有效地支持多种类型的计算，如交互式查询和流处理。Spark被设计的高度易访问，并提供了丰富的内建库，可以使用Python、Java、Scala或SQL设计Spark任务。

### 1.1 Spark运行模式

Spark可以有以下几种运行方式：

\- Local:

以本地单线程的方式运行Spark，一般适用于本地开发、测试。

\- Spark Standalone:

以运行一个主节点Master和多个工作节点Worker的方式运行Spark，自带完整的服务，可单独部署到一个集群中，无需依赖任何其他资源管理系统。

\- Spark on yarn（UHadoop采用的方式）

基于Hadoop的资源管理系统Yarn，Spark作为提交任务的客户端，所有任务都提交到Yarn上，由Yarn来分配任务执行。Spark on
yarn也分为yarn-cluster与yarn-client模式。区别如下：

``` 
  - yarn-cluster：Driver运行在Appliaction Master（AM）上。AM进程同时负责驱动Application和资源申请等，它运行在Container内，客户端提交完任务可关闭。一般适用于生产环境，但不适合运行交互类任务。
  - yarn-client：Driver运行在本地。任务提交后，客户端需要和Container通信进行作业的调度。适用于交互类任务和调试，可更加方便的看到任务的结果。
```

## 2\. Spark使用方式

### 2.1 Spark-submit

最通用的spark任务提交方式，通过spark-submit可以提交spark任务。spark-submit具体使用可以通过spark-submit
--help查看。

**示例**

\- Example：spark-submit提交示例程序中计算pi的任务

\`\`\` spark-submit --master yarn --deploy-mode client --num-executors 2
--executor-cores 1 --executor-memory 1G
$SPARK\_HOME/examples/src/main/python/pi.py 100 \`\`\`

更多关于提交任务的操作请参考：
<https://spark.apache.org/docs/1.6.0/submitting-applications.html>

**在集群外部机器提交任务**

1.  配置环境请参考第1节。
2.  hdfs只能使用本地用户名（whoami显示的这个名字）来当做hdfs的用户名，所以需要在hdfs上面加入本地用户名对目标文件的权限。Spark客户端提交的任务会默认使用
    /user/\[username\]这个目录。所以需要在hdfs上面建立相应的用户根目录。

![](/images/developer/sparksubmit.jpg)

测试命令：

    [hadoop@10-10-116-236 bin]$ pwd
    /root/testsparkclient/spark/bin
    [hadoop@10-10-116-236 bin]$ ./spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster  ../lib/spark-examples*.jar 10

查看运行结果:

> 屏幕打印final status: SUCCEEDED 代表执行成功。
> 
> 这个例子的输出结果是使用标准输出打印的:System.out.println("Pi is roughly " + 4.0 \* count
> / n)

\>

> 所以只有client模式会打印到屏幕上，yarn模式需要去log中查看
> hdfs://Ucluster/var/log/hadoop-yarn/apps/hadoop/logs/applicationid

### 2.2 Spark-shell

spark-shell是Spark提供的可通过scala语言快速实现任务执行的方式。

**示例**

\- 启动spark-shell客户端

    spark-shell

\- 构造一个HiveContext

    scala> val sqlContext = new org.apache.spark.sql.hive.HiveContext(sc);

\- 创建表格src

    scala> sqlContext.sql("CREATE TABLE IF NOT EXISTS src (key INT,value STRING)")

\- 从本地文件加载数据

    scala> sqlContext.sql("LOAD DATA LOCAL INPATH '/home/hadoop/spark/examples/src/main/resources/kv1.txt' INTO TABLE src")

\- 表格操作，显示表src数据

    scala> sqlContext.sql("FROM src SELECT key, value").collect().foreach(println);

### 2.3 Spark-sql

spark-sql是Spark提供的一种用SQL的方式处理结构化数据的组件，它提供了一个叫做DataFrames的可编程抽象数据模型，并且可被视为一个分布式的SQL查询引擎，它支持大部分常用的Hive
SQL

**示例**

\- 启动spark-sql客户端

    spark-sql

\- 执行sql查询

    spark-sql> select * from src;

### 2.4 Spark-Hive

使用Spark Hive的时候需要在SPARK\_HOME/conf下面配置hive-site.xml。通过spark-shell操作Hive
table。

我们可以通过spark-shell -\\-help查看spark-shell的相关用法。

For-example:

> spark-shell --master yarn --deploy-mode client --num-executors 3

我们采用yarn client模式启动spark-shell，并且设置executor的个数为3

spark-shell启动后我们即可进行相关的操作:

\- 构造一个HiveContext

![](/images/developer/sparkhive.jpg)

\- 创建表格src

![](/images/developer/sparkbiaoge.jpg)

\- 加载数据

![](/images/developer/sparkjiazaibiao.jpg)

\- 表格操作

![](/images/developer/sparkbiaocaozuo.jpg)

**通过spark-sql操作Hive table**

我们可以通过spark-sql -\\-help查看spark-sql的相关用法。

For-example：

> spark-sql --master yarn --deploy-mode client --num-executors 3

我们采用yarn client模式启动spark-sql，并且设置executor的个数为3

spark-sql启动之后我们就可以使用Hive的表格进行相关操作。

### 2.5 Spark-ThriftServer

通过Thrift JDBC/ODBC server的方式操作hive表

**示例**

\- 启动spark-thriftserver Master1节点上hadoop用户下执行

    /home/hadoop/spark/sbin/start-thriftserver.sh --hiveconf hive.server2.thrift.port=10000 --hiveconf hive.server2.thrift.bind.host=`hostname`  --supervise

\- beeline的方式连接thrift接口

    beeline> !connect jdbc:hive2://uhadoop-******-master1:10000/default;

> 注解：此处用户名密码传空即可

\- 执行sql

    0: jdbc:hive2://uhadoop-*****-master1:10000/> show tables;
    0: jdbc:hive2://uhadoop-*****-master1:10000/> select * from src;

## 3\. Spark应用开发

### 3.1 JAVA - WordCount示例

在examples中spark提供了这样几个java例子：

![](/images/developer/sparkkaifa1.jpg)

仅以wordcount为例展示运行方法：

```
  spark-submit --class org.apache.spark.examples.JavaWordCount --master yarn --deploy-mode cluster /home/hadoop/spark/lib/spark-examples*.jar /input/kv.txt /output/wordcount
```

> 注解：

1.  在hdfs上面准备好/input/kv.txt数据，随便一个文本什么的都行。
2.  /input/kv.txt /output/wordcount为运行参数，如果是其他程序，需要替换为对应的参数。
3.  这里没有指定分配的资源，在用户实际测试过程中，需要根据测试时使用的运算量，合理的设定资源。

下面将展示完整的过程

利用eclipse开发过程spark java程序过程：

1.  创建Java Project

![](/images/developer/sparkkaifa2.jpg)

![](/images/developer/sparkkaifa3.jpg)

1.  创建lib目录把工程依赖包spark-assembly-1.4.1-hadoop2.6.0-cdh5.4.4.jar加入lib包内，并add
    to build path

![](/images/developer/sparkkaifa4.jpg)

![](/images/developer/sparkkaifa5.jpg)

1.  完成功能代码

代码内容如下：

``` java
  package test.java.spark.job;
  
  import java.util.Arrays;
  import java.util.List;
  import java.util.regex.Pattern;
  import org.apache.spark.SparkConf;
  import org.apache.spark.api.java.JavaPairRDD;
  import org.apache.spark.api.java.JavaRDD;
  import org.apache.spark.api.java.JavaSparkContext;
  import org.apache.spark.api.java.function.FlatMapFunction;
  import org.apache.spark.api.java.function.Function2;
  import org.apache.spark.api.java.function.PairFunction;
  
  import scala.Tuple2;
  
  public final class JavaWordCount {
      private static final Pattern SPACE = Pattern.compile(" ");
  
      public static void main(String[] args) throws Exception {
  
        if (args.length < 2) {
          System.err.println("Usage: JavaWordCount <input> <output>");
          System.exit(1);
        }
  
        SparkConf sparkConf = new SparkConf().setAppName("JavaWordCount");
        JavaSparkContext ctx = new JavaSparkContext(sparkConf);
        JavaRDD<String> lines = ctx.textFile(args[0], 1);
  
        JavaRDD<String> words = lines.flatMap(new FlatMapFunction<String, String>() {
          @Override
          public Iterable<String> call(String s) {
            return Arrays.asList(SPACE.split(s));
          }
        });
  
        JavaPairRDD<String, Integer> ones = words.mapToPair(new PairFunction<String, String, Integer>() {
          @Override
          public Tuple2<String, Integer> call(String s) {
            return new Tuple2<String, Integer>(s, 1);
          }
        });
  
        JavaPairRDD<String, Integer> counts = ones.reduceByKey(new Function2<Integer, Integer, Integer>() {
          @Override
          public Integer call(Integer i1, Integer i2) {
            return i1 + i2;
          }
        });
  
        List<Tuple2<String, Integer>> output = counts.collect();
        counts.saveAsTextFile(args[1]);
        for (Tuple2<?,?> tuple : output) {
          System.out.println(tuple._1() + ": " + tuple._2());
        }
        ctx.stop();
     } 
  }
```

1.  导出工程文件

![](/images/developer/sparkkaifa6.jpg)

![](/images/developer/sparkkaifa7.jpg)

![](/images/developer/sparkkaifa8.jpg)

1.  数据准备

将一段文本上传到 hdfs的这个位置 /input/kv1.txt

```
hdfs dfs –put kv1.txt /input/kv1.txt
```

1.  提交任务运行



```
  spark-submit --class test.java.spark.JavaWordCount --master yarn --deploy-mode client --num-executors 1 --driver-memory 1g --executor-memory 1g --executor-cores 1  /home/hadoop/sparkJavaExample.jar /input/kv1.txt /output/wordcount
```

> 注解：
> 
> 资源配置不要超过当前机器的配额。

### 3.2 Scala - HiveFromSpark示例

\- 安装sbt

```
curl https://bintray.com/sbt/rpm/rpm > bintray-sbt-rpm.repo
sudo mv bintray-sbt-rpm.repo /etc/yum.repos.d/
sudo yum install sbt -y
```

\- 构建代码

以Spark example的HiveFromSpark为例：

```
mkdir -p /data/HiveFromSpark/src/main/scala/com/ucloud/spark/examples
cd /data/HiveFromSpark/src/main/scala/com/ucloud/spark/examples
touch HiveFromSpark.scala; 
```

以Spark1.6.0、scala-2.10.5为例，可将下述代码添加进HiveFromSpark.scala

``` java
package com.ucloud.spark.examples

import com.google.common.io.{ByteStreams, Files}
import java.io.File
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.sql._
import org.apache.spark.sql.hive.HiveContext

object HiveFromSpark {
  case class Record(key: Int, value: String)

  def main(args: Array[String]) {
    val sparkConf = new SparkConf().setAppName("HiveFromSpark")
    val sc = new SparkContext(sparkConf)

    val hiveContext = new HiveContext(sc)
    import hiveContext.implicits._
    import hiveContext.sql

    // Queries are expressed in HiveQL
    println("Result of 'SELECT *': ")
    sql("SELECT * FROM src").collect().foreach(println)

    // Aggregation queries are also supported.
    val count = sql("SELECT COUNT(*) FROM src").collect().head.getLong(0)
    println(s"COUNT(*): $count")

    // The results of SQL queries are themselves RDDs and support all normal RDD functions.  The
    // items in the RDD are of type Row, which allows you to access each column by ordinal.
    val rddFromSql = sql("SELECT key, value FROM src WHERE key < 10 ORDER BY key")

    println("Result of RDD.map:")
    val rddAsStrings = rddFromSql.map {
      case Row(key: Int, value: String) => s"Key: $key, Value: $value"
    }

    // You can also register RDDs as temporary tables within a HiveContext.
    val rdd = sc.parallelize((1 to 100).map(i => Record(i, s"val_$i")))
     // You can also register RDDs as temporary tables within a HiveContext.
    val rdd = sc.parallelize((1 to 100).map(i => Record(i, s"val_$i")))
    rdd.toDF().registerTempTable("records")

    // Queries can then join RDD data with data stored in Hive.
    println("Result of SELECT *:")
    sql("SELECT * FROM records r JOIN src s ON r.key = s.key").collect().foreach(println)

    sc.stop()
  }
}
```

\- 构建sbt文件

```
cd /data/HiveFromSpark/
touch  HiveFromSpark.sbt; 
```

将下述内容添加到文件:

```
name := "HiveFromSpark"
version := "1.0"
scalaVersion := "2.10.5"

libraryDependencies += "org.apache.spark" %% "spark-core" % "1.6.0"
libraryDependencies += "org.apache.spark" %% "spark-sql" % "1.6.0"
libraryDependencies += "org.apache.spark" % "spark-hive_2.10" % "1.6.0"
```

\- 编译

```
cd /data/HiveFromSpark/
sbt package
```

> 由于需要连接maven下载相关依赖包，编译时间受网络环境限制，请耐心等待， 编译后文件位于
> /data/HiveFromSpark/target/scala-2.10/hivefromspark\_2.10-1.0.jar

\- 执行

client模式:

```
spark-submit --class com.ucloud.spark.examples.HiveFromSpark --master yarn --deploy-mode client --num-executors 4 --executor-cores 1 /data/HiveFromSpark/target/scala-2.10/hivefromspark_2.10-1.0.jar
```

cluster模式:

```
spark-submit --class com.ucloud.spark.examples.HiveFromSpark --master yarn --deploy-mode cluster --num-executors 4 --executor-cores 1 --files /home/hadoop/hive/conf/hive-site.xml --jars /home/hadoop/spark/lib/datanucleus-api-jdo-3.2.6.jar,/home/hadoop/spark/lib/datanucleus-rdbms-3.2.9.jar,/home/hadoop/spark/lib/datanucleus-core-3.2.10.jar /data/HiveFromSpark/target/scala-2.10/hivefromspark_2.10-1.0.jar
```

### 3.3 Python - PI示例

> 注解：请参考spark安装目录下examples/src/main/python目录下的实例程序。

**示例代码**

``` python
from __future__ import print_function

import sys
from random import random
from operator import add

from pyspark.sql import SparkSession

if __name__ == "__main__":
    """
        Usage: pi [partitions]
    """
    spark = SparkSession\
        .builder\
        .appName("PythonPi")\
        .getOrCreate()

    partitions = int(sys.argv[1]) if len(sys.argv) > 1 else 2
    n = 100000 * partitions

    def f(_):
        x = random() * 2 - 1
        y = random() * 2 - 1
        return 1 if x ** 2 + y ** 2 < 1 else 0

    count = spark.sparkContext.parallelize(range(1, n + 1), partitions).map(f).reduce(add)
    print("Pi is roughly %f" % (4.0 * count / n))

    spark.stop()
```

可通过以下命令提交Spark PI任务

```
spark-submit  --master yarn --deploy-mode client --num-executors 4 --executor-cores 1 --executor-memory 2G $SPARK_HOME/examples/src/main/python/pi.py 100
```

最终在console的日志中会出现类似“Pi is roughly 3.141039”的结果

## 4\. Spark Streaming

请参考 <http://static.ucloud.cn/6799401b027e12e2206591051a107507.pdf> 1.6章节

更多关于Spark的知识请参考官方文档：<http://spark.apache.org/>
