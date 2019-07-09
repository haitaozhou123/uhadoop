{{indexmenu_n>8}}

# Sqoop2开发指南

Sqoop1一般使用shell脚本方式进行操作，Sqoop2则引入了Sqoop
server，对Connector实现了集中的管理。其访问方式也变得多样化，可以通过
REST API、JAVA API、WEB UI以及CLI控制台方式进行访问，与Sqoop的架构和使用方式完全不同。

如果您创建集群勾选了Sqoop2，Sqoop2将被安装在uhadoop-\*\*\*\*\*\*-master2节点上。

## 1\. 从MySQL导出到HDFS

### 1.1 创建mysql数据

Mysql数据源以UDB为例（创建UDB方式可以参考https://docs.ucloud.cn/database/udb-mysql/quick）

连接Mysql,并创建数据库sqoop和表sqoop

```
[root@uhadoop-******-master2 ~]# mysql -h10.13.124.35 -uroot -p'ucloud' sqoop

mysql> create database sqoop;
Query OK, 1 row affected (0.00 sec)

mysql> use sqoop
Database changed
mysql> CREATE TABLE `sqoop` (
    ->   `id` varchar(64) DEFAULT NULL,
    ->   `value` varchar(64) NOT NULL DEFAULT '',
    ->     PRIMARY KEY (`id`)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.01 sec)

mysql> insert into sqoop values ('1', 'hha'),('2', 'zhang'),('3','hehe');
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

### 1.2 启动Sqoop2 CLI

uhadoop-\*\*\*\*\*\*-master2节点上执行

```
/home/hadoop/sqoop2/bin/sqoop.sh client
```

help可查到基本使用命令

```
sqoop:000> help
For information about Sqoop, visit: http://sqoop.apache.org/

Available commands:
  exit    (\x  ) Exit the shell
  history (\H  ) Display, manage and recall edit-line history
  help    (\h  ) Display this help message
  set     (\st ) Configure various client options and settings
  show    (\sh ) Display various objects and configuration options
  create  (\cr ) Create new object in Sqoop repository
  delete  (\d  ) Delete existing object in Sqoop repository
  update  (\up ) Update objects in Sqoop repository
  clone   (\cl ) Create new object based on existing one
  start   (\sta) Start job
  stop    (\stp) Stop job
  status  (\stu) Display status of a job
  enable  (\en ) Enable object in Sqoop repository
  disable (\di ) Disable object in Sqoop repository

For help on a specific command type: help command
```

### 1.3 查看connector

```
sqoop:000> show connector
```

![800](/images/developer/sqoop-connector.jpg)

### 1.4 创建mysql-link

```
sqoop:000> create link -c 4
Creating link for connector with id 4
Please fill following values to create new link object
Name: mysql-link

Link configuration

JDBC Driver Class: com.mysql.jdbc.Driver
JDBC Connection String: jdbc:mysql://10.13.124.35/sqoop
Username: root
Password: **********
JDBC Connection Properties:
There are currently 0 values in the map:
entry# protocol=tcp
There are currently 1 values in the map:
protocol = tcp
entry#
New link was successfully created with validation status OK and persistent id 1
```

### 1.5 创建hdfs-link

```
sqoop:000> create link -c 3
Creating link for connector with id 3
Please fill following values to create new link object
Name: hdfs-link

Link configuration

HDFS URI: hdfs://Ucluster/
New link was successfully created with validation status OK and persistent id 2
```

### 1.6 显示link

```
sqoop:000> show link
```

![800](/images/developer/sqoop2-show-link.jpg)

### 1.7 创建job

```
sqoop:000> create job -f 1 -t 2
Creating job for links with from id 1 and to id 2
Please fill following values to create new job object
Name: job-mysql-to-hdfs

From database configuration

Schema name: sqoop
Table name: sqoop
Table SQL statement:
Table column names:
Partition column name: id
Null value allowed for the partition column:
Boundary query:

ToJob configuration

Override null value:
Null value:
Output format:
  0 : TEXT_FILE
  1 : SEQUENCE_FILE
Choose: 0
Compression format:
  0 : NONE
  1 : DEFAULT
  2 : DEFLATE
  3 : GZIP
  4 : BZIP2
  5 : LZO
  6 : LZ4
  7 : SNAPPY
  8 : CUSTOM
Choose: 0
Custom compression format:
Output directory: /tmp/sqoop

Throttling resources

Extractors: 1
Loaders: 1
New job was successfully created with validation status OK  and persistent id 1
```

创建完job如下

```
sqoop:000> show job
```

![800](/images/developer/sqoop2-show-job.jpg)

### 1.8 启动job

```
sqoop:000> start job -j 1
Submission details
Job ID: 1
Server URL: http://localhost:12000/sqoop/
Created by: root
Creation date: 2016-12-22 14:40:10 CST
Lastly updated by: root
External ID: job_1481968387780_0011
    http://uhadoop-penomi-master1:23188/proxy/application_1481968387780_0011/
2016-12-22 14:40:10 CST: BOOTING  - Progress is not available
```

## 2\. 从HDFS导出到MySQL

这里利用MySQL导入HDFS中创建的hdsf-link和mysql-link

### 2.1 创建job

```
sqoop:000> create job -f 2 -t 1
Creating job for links with from id 2 and to id 1
Please fill following values to create new job object
Name: job-hdfs-to-mysql

From Job configuration

Input directory: /tmp/sqoop-input
Override null value:
Null value:

To database configuration

Schema name: sqoop
Table name: sqoop
Table SQL statement:
Table column names:
Stage table name:
Should clear stage table:

Throttling resources

Extractors: 1
Loaders: 1
New job was successfully created with validation status OK  and persistent id 2
```

添加完job如下

![800](/images/developer/sqoop2-show-job2.jpg)

### 2.2 执行job

```
sqoop:000> start job -j 2
Submission details
Job ID: 2
Server URL: http://localhost:12000/sqoop/
Created by: root
Creation date: 2016-12-22 15:38:00 CST
Lastly updated by: root
External ID: job_1481968387780_0012
    http://uhadoop-penomi-master1:23188/proxy/application_1481968387780_0012/
2016-12-22 15:38:00 CST: BOOTING  - Progress is not available
```

### 2.3 查看MySQL数据

```
mysql> select * from sqoop;
+----+--------+
| id | value  |
+----+--------+
| 1  | hha    |
| 2  | zhang  |
| 3  | hehe   |
| 4  | zhen   |
| 5  | u      |
| 6  | ucloud |
+----+--------+
6 rows in set (0.00 sec)
```
