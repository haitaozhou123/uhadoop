{{indexmenu_n>7}}

# Sqoop开发指南

Sqoop是可以将Hadoop和关系型数据库中的数据相互转移的工具，可以将一个关系型数据库（例如：MySQL，Oracle，Postgres等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。

> 注解：

``` 
 - Sqoop导入导出mysql中的数据，需要确保mysql中的数据可以被远程用户访问。否则会报权限错误。
 - 详细使用请参考[[http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html|官方网页]]
```

## 1. 基础操作

### 1.1 Sqoop的安装配置

UHadoop中Sqoop默认与Oozie一起安装，如果您创建集群时勾选了Oozie，Sqoop将会安装在uhadoop-\*\*\*\*\*\*-master2节点上。如果需要单独安装Sqoop，可以参考以下步骤，否则可略过。

#### a. UHadoop中脚本安装

以在master1节点安装为例。

- 在master1节点上以root用户执行以下命令即可，默认安装到/home/hadoop/目录下

```
sh /home/hadoop/.versions/umrAgent/script/install.sh sqoop 1.4.5 cdh5.4.9
```

#### b. tar包安装

以下示例为在UHadoop节点上下载tar包安装过程；若在非UHadoop节点上安装，需变更相关路径，可以适当参考。

1.  在 <http://archive.cloudera.com/cdh5/cdh/5/>
    下载sqoop-1.4.5-cdh5.4.9.tar.gz,并解压到/home/hadoop/.versions/



```
  wget "http://archive.cloudera.com/cdh5/cdh/5/sqoop-1.4.5-cdh5.4.9.tar.gz"
  tar -zvxf sqoop-1.4.5-cdh5.4.9.tar.gz -C /home/hadoop/.versions/
```

2.  hadoop用户下建立软链



```
  ln -s /home/hadoop/.versions/sqoop-1.4.5-cdh5.4.9/ /home/hadoop/sqoop
```

3.  加环境变量vim \~/.bashrc



```
# sqoop
export SQOOP_HOME=$HADOOP_HOME/sqoop
export PATH=$PATH:$SQOOP_HOME/bin
```

4.  配置sqoop环境变量



```
  cp $SQOOP_HOME/conf/sqoop-env-template.sh  $SQOOP_HOME/conf/sqoop-env.sh
```

修改如下参数：

```
#Set path to where bin/hadoop is available
export HADOOP_COMMON_HOME=/home/hadoop

#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/home/hadoop/share/hadoop/mapreduce

#set the path to where bin/hbase is available
export HBASE_HOME=/home/hadoop/hbase

#Set the path to where bin/hive is available
export HIVE_HOME=/home/hadoop/hive

#Set the path for where zookeper config dir is
export ZOOCFGDIR=/home/hadoop/zookeeper/conf
```

5.  拷贝相关依赖



```
  cd /home/hadoop/sqoop
  cp /home/hadoop/hive/lib/mysql-connector-java-*.jar ./lib/
  cp /home/hadoop/share/hadoop/mapreduce/* ./lib/
```

### 1.2 从MySQL导出到HDFS

- mysql信息如下：

以MySQL 为例：

• 10.10.50.79

• database: hehe

• 用户：test 密码：test

表格信息如下：

![](/images/developer/sqoopbiaoge.jpg)

#### 执行语句

``` sql
  sqoop import --connect jdbc:mysql://10.10.50.79/hehe --username test --password test --table t_hadoop_version  --target-dir /tmp/sqoop-import-hdfs
```

> 注解
>1. 10.10.50.79 mysql的ip
>2. --username 访问数据的用户名称
>3. --password 访问数据的密码
>4. --table t_hadoop_version 数据表名称
>5. --target-dir 数据导入到hdfs中的目标目录

#### 查看结果

![](/images/developer/sqoopjieguo.jpg)

### 1.3 从MySQL导出到Hive

1.  mysql信息如下：

![](/images/developer/sqoopbiaogebu.jpg)

#### 执行语句

```
  sqoop import --connect jdbc:mysql://10.10.50.79/hehe --username test --password test --table t_hadoop_version  --warehouse-dir /user/hive/warehouse --hive-import --create-hive-table
```

> 注解：
1.  10.10.50.79是mysql的ip
2.  --username test 访问数据的用户名称
3.  --password test 访问数据的密码
4.  --table t_hadoop_version 数据表名称
5.  --warehouse-dir hive数据库的hdfs目录

#### 查看结果

如图所示，mysql中的表格t_hadoop_version已经导入hive中

![](/images/developer/sqoopjieguo2.jpg)

### 1.4 从Hive导出到MySQL

#### hive表格中数据如下所示：

![](/images/developer/sqoopbiaoge2.jpg)

#### 执行语句

```
  sqoop export --connect jdbc:mysql://10.10.50.79/hehe --username test --password test --table t_hadoop_version  --export-dir /user/hive/warehouse/t_hadoop_version --fields-terminated-by "\t" 
```

> 注解

1.  10.10.50.79是mysql的ip
2.  --username test 访问数据的用户名称
3.  --password test 访问数据的密码
4.  --table t_hadoop_version 数据表名称
5.  --export-dir 要导出的数据目录
6.  --fields-terminated-by 数据分割方式

#### 查看结果

之前mysql数据库中的结果+hive数据创库的结果，显示如下所示：

![](/images/developer/sqoopjieguo3.jpg)

## 2. 在UHadoop中使用Sqoop

本例为在UHadoop集群的Master2节点上，使用sqoop，将内网（UDB或自建数据库）中MySQL数据库中的数据，以增量方式导入到UHadoop集群的Hive中。

> 注解：存量集群中存在sqoop部署在Master1上的情况，请登陆后进行测试。

在进行MySQL到Hive的增量数据导入时，需要原数据中，可以通过某一列的数据数据来判断所需导入的增量数据。

本例操作流程中，主要通过在MySQL中构建测试数据库、表、数据，然后依据原数据表中的标签timestamp，并通过对比已经导入到Hive数据库中数据的timestamp最后值，来确定所需要增量导入的数据。

### 2.1 创建库和表

在MySQL中创建导出数据库(data\_demo)、表(data\_export)

``` sql
create database data_demo;
create table data_demo.data_export (i int , t timestamp(3), cn VARCHAR(64));
```

### 2.2 在Hive中创建数据库

``` sql
create database data_demo1;
create table data_demo1.data_import (i int , t VARCHAR(64), cn VARCHAR(64));
```

### 2.3 生成数据

建立生成测试数据的脚本(gene\_data.sh)，并运行脚本来生成数据，插入到数据库data\_demo

```
touch gene_data.sh
```

脚本内容如下：

```
#!/bin/bash

#update
i=$(($RANDOM % 10))
up="insert into data_export values ('$i' , current_timestamp() , '中文\n') ON DUPLICATE KEY UPDATE t=current_timestamp()"
echo $up
mysql data_demo -e "$up"
i=$RANDOM
insert="insert into data_export values ('$i' , current_timestamp() , '中文\n');"
echo $insert
mysql data_demo -e "$insert"
```

执行脚本插入数据

```
bash gene_data.sh
```

### 2.4 添加权限

MySQL数据库在执行数据导出时，需要增加对目标HIVE数据库的可访问权限，可执行以下脚本为HIVE节点添加权限。

创建脚本文件

```
touch grant.sh
```

编辑脚本文件内容如下：

```
USER_NAME=hello
USER_PASSWD=word
DATA_BASE=data_demo
for h in `grep uhadoop /etc/hosts | awk '{print $2}'`
do
    mysql -e "CREATE USER '$USER_NAME'@'$h' IDENTIFIED BY '$USER_PASSWD'"
    echo "CREATE USER '$USER_NAME'@'$h' IDENTIFIED BY '$USER_PASSWD'"
    mysql -e "GRANT SELECT,INSERT,UPDATE,DELETE,LOCK TABLES,EXECUTE ON $DATA_BASE.* TO '$USER_NAME'@'$h'"
    echo "GRANT SELECT,INSERT,UPDATE,DELETE,LOCK TABLES,EXECUTE ON $DATA_BASE.* TO '$USER_NAME'@'$h'"
done
mysql -e "FLUSH PRIVILEGES"
```

执行脚本修改权限

```
bash grant.sh
```

### 2.5 导入数据

> 注解：本例中MySQL地址为”10.10.115.1”，请根据需要修改$USER与$PASSWORD，此处“--last-value”需以实际在mysql中生成的测试数据时间为准。

```
sqoop import --connect jdbc:mysql://10.10.115.1:3306/data_demo --username $USER --password $PASSWORD --table data_export   --warehouse-dir /user/hive/warehouse/tmp --hive-import --hive-table  data_demo1.data_import --check-column t --incremental lastmodified --last-value '2016-11-15 17:22:24.496'  --merge-key i  --hive-drop-import-delims --hive-overwrite -m 1
```

参数详解参考sqoop的使用手册

<http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html>

测试过程中可能遇到的问题如下

> 1.UHadoop集群中task节点或core节点由于没有mysql的权限导入数据失败

请授权给集群中所有节点对MySQL数据库的select权限

> 2.在进行数据导入时只会有一个mapreduce任务在跑

可通过-m参数指定map任务数，命令中为1。
