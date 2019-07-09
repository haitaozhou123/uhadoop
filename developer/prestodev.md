{{indexmenu_n>0}}

# Presto开发指南

## Presto介绍

Presto是一个开源的分布式SQL查询引擎，适用于GB到PB级别数据量的交互式分析查询。

它支持 Hive, Cassandra, 关系数据库等多种源数据的在线查询和数据整合，是大规模商业数据仓库进行快速交互式分析的利器。

## hive-cli中进行表操作

这里选择在master1节点进行测试

1.打开hive-cli

```
[root@uhadoop-xxxxxxxx-master1 hadoop]# hive
```

2.查看hive中数据库列表

```
hive> show databases;
```

3.创建新的test数据库

```
hive> create schema test;
```

4.向test库中加入新表table\_one

```
hive> create table test.table_one(name string);
```

5.查看test库中的全部表

```
hive> show tables from test;
```

6.向test库的table\_one表中插入数据

```
hive> insert into test.table_one values ('name1'),('name2'),('name3');
```

7.hive-cli中查询table\_one表数据。

```
hive> select * from test.table_one;
OK
name1
name2
name3
```

8.退出hive-cli

```
hive> quit;
```

## presto-cli中进行表操作

1.打开presto-cli

```
[root@uhadoop-xxxxxxxx-core1 hadoop]# presto-cli --server uhadoop-xxxxxx-master1:28080 --catalog hive --schema test
presto:test>
```

2.presto-cli 查看hive下全部库信息

```
presto:test> show schemas from hive;
```

3.presto-cli 查看hive下test库的表清单

```
presto:test> show tables from test;
```

4.presto-cli下进行hive表中数据查询。

```
presto:test> select * from test.table_one;
 name
-------
 name1
 name2
 name3
(3 rows)
```

5.直接在presto-cli中进行hive catalog建表操作。

```
presto:test> create table test.presto_table2(name varchar);
```

6.直接在presto-cli中进行 insert 操作

```
presto:test> insert into test.presto_table2 values('myname1'),('myname2');
```

  - 查询上一步的插入结果：



```
presto:test> select * from test.presto_table2;
  name
---------
 myname1
 myname2
(2 rows)
```

  - presto-cli 退出



```
presto:test> exit;
```

## 调用rest-api查看presto集群状态

```
curl http://uhadoop-xxxxxxxx-master1:28080/v1/cluster
```

返回：

```
{
    "runningQueries":0,
    "blockedQueries":0,
    "queuedQueries":0,
    "activeWorkers":3,
    "runningDrivers":0,
    "reservedMemory":0.0,
    "totalInputRows":88,
    "totalInputBytes":2061,
    "totalCpuTimeSecs":1
}
```
