{{indexmenu_n>7}}

# Spark

## 如何使用Spark-sql访问Hive表？

如果Spark-sql不能访问Hive表，请依次检查以下操作是否完成。没有完成的，需要在uhadoop集群和Spark客户端上操作：

\-
将/home/hadoop/hive/conf/hive-site.xml和/home/hadoop/hbase/conf/hbase-site.xml拷贝到/home/hadoop/spark/conf/下；

\-
将/home/hadoop/hive/lib/hive-serde-\*-cdh\*.jar拷贝到/home/hadoop/spark/lib/下；

\- 在/home/hadoop/spark/conf/spark-env.sh中增加以下配置：

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/hadoop/lib/native
export SPARK_LIBRARY_PATH=$SPARK_LIBRARY_PATH:/home/hadoop/lib/native
export SPARK_CLASS PATH=$SPARK_CLASSPATH:/home/hadoop/share/hadoop/common/:/home/hadoop/share/hadoop/common/lib/:/home/hadoop/spark/lib/:/home/hadoop/hive/lib/:/home/hadoop/hbase/lib/*
```

## 使用Spark连接数据库时，提示java.sql.SQLException怎么办？

如果使用Spark连接数据库时，遇到以下错误：

```
Exception in thread "main" java.sql.SQLException: No suitable driver found for jdbc:mysql://
```

是因为集群上使用的mysql包是5.1.17版本的(home/hadoop/hive/lib/mysql-connector-java-5.1.17.jar)，用户连的数据库的版本可能是更高版本。在[mysql官网](https://dev.mysql.com/downloads/connector/j/5.0.html)下载最新的包替换掉就可以了。

> 举例：

```
spark-submit --class users_day_activity --master yarn-client --jars /root/hive/lib/mysql-connector-java-5.6.jar --executor-memory 2g --num-executors 5
```

其他的数据库或者服务也可能出现这个问题， 找到对应的包用应该就可以了。

## 如何查看spark运行任务的日志

少部分存量集群没有配置spark的history，所以看不到spark任务的日志，需要重新配置下。

具体方法如下：

**1.为spark配置jobhistory服务**

修改配置文件/home/hadoop/spark/conf/spark-defaults.conf

```
    spark.history.ui.port 18080
    spark.eventLog.dir hdfs://Ucluster/var/log/spark
    spark.eventLog.enabled  true
    spark.yarn.historyServer.address uhadoop-XXXXXX-master2:18080
    spark.history.fs.logDirectory hdfs://Ucluster/var/log/spark
```

> 注解：

\>

> 1.修改配置的机器为集群所有节点，包括提交任务的客户端；

\>

> 2.uhadoop-XXXXXX需要修改为集群ID。

**2.NodeManager配置log-url**

在所有的nodemanger节点，为/home/hadoop/conf/yarn-site.xml增加以下配置：

```
     <property>
      <name>yarn.log.server.url</name>
      <value>http://uhadoop-XXXXXX-master2:19888/jobhistory/logs</value>
     </property>
```

> 地址配置为集群的master2的主机名。

修改后，重启nodemanager服务。

**3.创建目录**

在所有的nodemanger节点，以root用户执行以下命令：

```
    su -s /bin/bash hadoop -c 'hdfs dfs -mkdir hdfs://Ucluster/var/log/spark'
    su -s /bin/bash hadoop -c 'hdfs dfs -chmod 777  hdfs://Ucluster/var/log/spark'
```

**4.启动服务**

在master2节点上使用hadoop用户，执行/home/hadoop/spark/sbin/start-history-server.sh脚本。
