### 介绍
Apache Phoenix项目由saleforce开源并贡献给Apache基金会，目前为Apache基金会的顶级项目。它是构建在HBase上的SQL中间层。 Phoenix会将用户编写的sql查询编译为一系列的scan操作，最终产生通用的JDBC结果集返回给客户端，小范围的查询可做到毫秒级响应，千万数据的响应速度为秒级。

### 使用
在完成Phoenix的安装后，进入Phoenix客户端目录，使用命令行工具：

1. 选择集群master或者core节点，进入客户端目录
```
# 这里我们选择master1节点
cd /home/hadoop/phoenix/bin
```
2. 使用Phoenix python命令行工具

```
./sqlline.py uhadoop-xxx-master1:2181


0: jdbc:phoenix:uhadoop-xxxx-master1>

```

3. 使用sql操纵Phoenix表

参考文档：https://phoenix.apache.org/language/index.html#create

**tips**:

Phoenix会自动将表名和字段名转换为大写字母，如果不想转换的话可以使用双引号把字段或者表名括起来

* 使用帮助，查看全部Phoenix命令
```
 0: jdbc:phoenix> !help
```

* 查看table列表
```
 0: jdbc:phoenix> !table
```

* 创建新表

```
 0: jdbc:phoenix:> create table test (
                    mykey integer not null primary key, 
                    mycolumn varchar
                );
```

* 插入数据
``` 
0: jdbc:phoenix:> upsert into test values (1,'uhadoop');
0: jdbc:phoenix:> upsert into test values (2,'phoenix');

```
*  查看表内容

```
0: jdbc:phoenix:> select * from test;
```

* 删除表内容

```
0: jdbc:phoenix:> delete from test where mykey=1;
```
* 为表创建索引

```
0: jdbc:phoenix:> create local index test_idx on test(mycolumn);

```

* 查看表的索引信息

```
0: jdbc:phoenix:> !indexes test
```

* 删除表

```
0: jdbc:phoenix:> drop table if exists test;
```


* 退出客户端

```
0: jdbc:phoenix:>!exit

```


4. 使用Phoenix与已有hbase表建立表映射关系

* 4.1 进入hbase shell命令行创建表并插入数据

```
[root@uhadoop-xxx-master1 bin]# hbase shell

hbase(main):009:0> create 'phoenix','info'

hbase(main):010:0> put 'phoenix', 'row001','info:name','phoenix'

hbase(main):011:0> scan 'phoenix'

```

* 4.2 进入Phoenix python客户端，建立映射关系

tips: 

>1.Phoenix 4.10 及以上的版本,Phoenix对列对编码格式有所改变([官网说明文档](http://phoenix.apache.org/columnencoding.html))，所以在建立与hbase映射关系时，需要设置 COLUMN_ENCODED_BYTES 属性为 0，即不让 Phoenix 对 column family 进行编码。 

> 2.HBase数据表默认主键列名是ROW

> 3.为了防止Phoenix自动将小写column转大写，需要用双引号将column扩起来

> 4.如果映射关系建立成功，建标之后可以看到有多少行数据被影响

* 创建表映射
```
0: jdbc:phoenix:> create table "phoenix"("ROW" varchar primary key, "info"."name" varchar) column_encoded_bytes=0;
1 row affected (6.523 seconds)
```

* 删除表（注意，这里hbase中表也会被同时删除）

```
0: jdbc:phoenix:> drop table "phoenix";
```

5. 使用Phoenix与已有hbase表建立视图映射关系

如果Phoenix表只是做查询操作对话，做表映射是个更好对选择，做表映射，如果在Phoenix客户端删除表，则hbase中该表也将被删除。 如果是用视图映射，则删除视图不会影响原有表的数据。

* 5.1 创建视图映射

```
# 如果hbase中不存在phoenix表，需要按照4.1节的方式在hbase中建表
0: jdbc:phoenix:> create view "phoenix"("ROW" varchar primary key, "info"."name" varchar);
```
* 5.2 查询视图

```
0: jdbc:phoenix:>select * from "phoenix";
```

* 5.3 删除视图

```
0: jdbc:phoenix:> drop view "phoenix";
```

