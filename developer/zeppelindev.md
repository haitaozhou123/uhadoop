### 介绍

Zeppelin是一个Web笔记形式的交互式数据查询分析工具，可以在线用多种语言和工具对数据进行查询分析并生成报表。

其默认对数据执行引擎是Spark，同时提供了Hive、HBase、Markdown等多种解释器来支持更多数据源的分析处理，官方支持的解释器如下：

在Zeppelin中以笔记本（notebook）的形式组织和管理交互式数据探索任务，执行引擎的作用就是执行笔记中的与引擎相对应的代码，以进行交互式数据分析和展现。

![](/images/notebook.png)

### 安装及登陆

目前在uhadoop集群中，zeppelin服务安装在master1节点的/home/hadoop/zeppelin目录,可通过uhadoop-xxxxx-master1:29090访问Zeppelin服务界面。

 - 启动： service Zeppelin start 
 - 重启： service Zeppelin restart 
 - 服务关闭： service Zeppelin stop
 - 状态检查： service Zeppelin status

uhadoop zeppelin不允许匿名用户使用，使用Shiro进行用户权限控制。所有可使用用户和密码信息均存放在/home/hadop/zeppelin/conf/shiro.ini中. 默认配置了一个管理员用户和3个普通用户，管理员可以进行全部操作，普通用户只能基于现有解释器创建notebook, 进行数据查询，无法创建或修改解释器。

| 用户名     |  密码      | 角色     |
| ---------- | ---------- | -------- |
| admin      | password1  | 管理员   | 
| user1      | password2  | 普通用户 |
| user2      | password3  | 普通用户 |
| user3      | password4  | 普通用户 |

用户打开zeppelin web页面后，点击右上角button进行登陆，需要先用管理员用户登陆，进行解释器相关配置：

![](/images/zeppelin-index.png)

管理员配置interpreter:

使用admin用户登陆后，点击右上角的下拉框，选择"Interpreter"，可以看的默认的解释器列表，管理员可以根据已有解释器创建新的解释器（后面在测试hive,mysql等数据源时，需要基于jdbc解释器创建新的解释器）。

![](/images/zeppelin-interpreter.png)

### Hive 解释器测试

#### Hive 解释器配置

zeppelin-0.8.1不再提供默认的Hive解释器，只提供了jdbc解释器，用户可以基于jdbc解释器创建专门的Hive解释器。

![](/images/create-hive.png)

并修改option为如下的配置：

![](/images/hive-interperter.png)

主要修改的配置为如下几项：

default.driver： org.apache.hive.jdbc.HiveDriver

default.url： jdbc:hive2://uhadoop-xxxxxx-master2:10000   (为集群hive-server2 jdbc地址）

default.user： hive

zeppelin.jdbc.auth.type:  simple

Dependencies的artifact增加如下两个（这里选择从集群本地文件读取）：

/home/hadoop/hive/jdbc/hive-jdbc-2.3.3-standalone.jar

/home/hadoop/lib/lib/hadoop-common-2.6.0-cdh5.13.3.jar


#### 3.2 Hive notebook创建

![](/images/create-note.png)

![](/images/hive-note.png)

#### 3.3 hive notebooke中创建查询

hive详细使用请参考：https://doc.ucloud.cn/analysis/uhadoop/developer/hivedev

点击打开上一步创建的/test/hive notebook， 在notebook中创建一个paragraph，输入如下：

%hive
show tables

点击右侧的run三角按钮即可执行，并在下方看到输出结果。

![](/images/hive-sql-1.png)

一个notebook中可以创建很多个paragraph，将鼠标悬浮在每个paragraph的下端，点击弹出的“Add Paragraph"就可以创建新的paragraph：

这里依次创建了三个paragraph，演示一个table从创建、插入数据、查询数据的过程。

sql：

     1. create table test_hive (id int, name string)

     2. insert into test_hive values (1,'test_ucloud'),(2,'test_hive')

     3. select * from test_hive

![](/images/hive-note-paragraph.png)

![](/images/hive-sql-2.png)

### 4.hbase解释器测试

参考文档：https://zeppelin.apache.org/docs/latest/interpreter/hbase.html

hbase解释器为Zeppelin自带的解释器，不需要再进行解释器的配置，只需在使用时带上"%hbase"便可以进行hbase语句解析

可以为hbase测试建立新的notebook，方法同上，然后创建如下paragraph并执行：

paragraph1:

```
%hbase

create 'test_hbase', 'cf'
```

paragraph2:

```
%hbase

list
```

paragraph3:

```
%hbase

put 'test_hbase', 'row1', 'cf:a', 'value1'
```

paragraph4:

```
%hbase
scan 'test_hbase'
```

执行结果如下：

![](/images/hbase-sql.png)

### 5. spark解释器使用

#### 概述：

参考文档：https://zeppelin.apache.org/docs/latest/interpreter/spark.html

Zeppelin spark解释器组支持如下几种解释器

| Name       |  Class     | Description     |
| ---------- | ---------- | -------- |
| %spark	| SparkInterpreter	| Creates a SparkContext and provides a Scala environment |
| %spark.pyspark |	PySparkInterpreter | Provides a Python environment |
| %spark.r |	SparkRInterpreter |	Provides an R environment with SparkR support |
| %spark.sql | SparkSQLInterpreter |	Provides a SQL environment |
| %spark.dep | DepInterpreter |	Dependency loader |

#### 5.1 解释器配置修改

spark解释器配置的默认部署类型是local mode, uhadoop中spark 是on yarn模式，修改配置的master属性为yarn-client或yarn-cluster:

![](/images/spark-interpreter.png)

#### 5.2 使用测试

下载测试数据，放入hdfs:

``` sh

wget http://mirrors.ucloud.cn/ucloud/udata/bank.csv

hadoop fs -put bank.csv  /

```

创建名为/test/spark的notebook，并在其中创建如下几个paragraph：

paragraph1:

```

%spark
import org.apache.commons.io.IOUtils
import java.net.URL
import java.nio.charset.Charset

val bankText = sc.textFile("/bank.csv")
case class Bank(age: Integer, job: String, marital: String, education: String, balance: Integer)

val bank = bankText.map(s => s.split(";")).filter(s => s(0) != "\"age\"").map(
s => Bank(s(0).toInt,
s(1).replaceAll("\"", ""),
s(2).replaceAll("\"", ""),
s(3).replaceAll("\"", ""),
s(5).replaceAll("\"", "").toInt
)
).toDF()
bank.registerTempTable("bank")
bank.show(10)

```

paragraph2:

```
%sql
select age, count(1) value
from bank
where age < 30
group by age
order by age
```

paragraph3:

```
%sql
show tables
```

执行预期结果如下图：

![](/images/spark-sql-1.png)



