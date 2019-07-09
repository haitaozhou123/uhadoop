{{indexmenu_n>4}}

# Hive

## Hive执行sql任务太慢，是否可以支持hive on spark？

UHadoop暂时未支持hive on
spark，但可通过使用[Spark-sql](http://docs.ucloud.cn/analysis/uhadoop/developer/sparkdev#spark-sql)代替，或者启用[Spark-thriftserver](http://docs.ucloud.cn/analysis/uhadoop/developer/sparkdev#spark-thriftserver)，通过beeline进行连接。

## 执行SQL语句时，map/reduce任务内存不足怎么办？

如果在日志文件中看到出现running beyond physical memory limits错误，可以通过set
mapreduce.map.memory.mb=XXX或者set
mapreduce.reduce.memory.mb=XXX来增大map或reduce可以使用的内存数。如果多个任务都出现这个情况，可以在/home/hadoop/hive/conf/hive-site.xml里加入配置

## hive-server2 通过jdbc提交任务的时候报文件权限不足

可以在连接hive的时候使用hadoop用户，以java代码为例：

``` java
Connection con = DriverManager.getConnection("jdbc:hive2://ip:10000/default", "hadoop", "");
```

## 执行sql时速度很慢怎么办？

Hive默认的优化策略是对select这种简单操作，并不会启动map任务。可以通过在命令中指定hive.fetch.task.conversion=none这个参数，或者配置在/home/hadoop/hive/conf/hive-site.xml中来强制制启动map任务。

> 举例：

\>

> hive -e "set hive.fetch.task.conversion=none;select \* from
> spark\_testtable;"
