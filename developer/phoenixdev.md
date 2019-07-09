{{indexmenu_n>0}}

# Phoenix开发指南

## Phoenix介绍

Apache Phoenix 项目由 Saleforce 开源并贡献给 Apache 基金会，目前为 Apache 基金会的顶级项目。

它是构建在 HBase 上的 SQL 中间层，Phoenix 会将用户编写的 SQL 查询编译为一系列的 scan 操作，最终产生通用的
JDBC 结果集返回给客户端。

小范围的查询可做到毫秒级响应，千万数据的响应速度为秒级。

## Phoenix使用

在完成 Phoenix 的安装后，进入 Phoenix 客户端目录，使用命令行工具：

1.选择集群 master 或者 core 节点，进入客户端目录

\`\`\` \# 这里我们选择 master1 节点 cd /home/hadoop/phoenix/bin \`\`\`

2.使用 Phoenix python 命令行工具

\`\`\` ./sqlline.py uhadoop-xxx-master1:2181 0:
jdbc:phoenix:uhadoop-xxxx-master1\>

\`\`\`

3.使用 SQL 操纵 Phoenix 表

参考文档：<https://phoenix.apache.org/language/index.html#create>

**tips**:

Phoenix 会自动将表名和字段名转换为大写字母，如果不想转换的话可以使用双引号把字段或者表名括起来

\* 使用帮助，查看全部 Phoenix 命令

\`\`\` 0: jdbc:phoenix\> \!help \`\`\`

\* 查看 table 列表

\`\`\` 0: jdbc:phoenix\> \!table \`\`\`

\* 创建新表

\`\`\` 0: jdbc:phoenix:\> create table test (

``` 
                  mykey integer not null primary key,
                  mycolumn varchar
              );
```

\`\`\`

\* 插入数据

\`\`\` 0: jdbc:phoenix:\> upsert into test values (1,'uhadoop'); 0:
jdbc:phoenix:\> upsert into test values (2,'phoenix');

\`\`\`

\* 查看表内容

\`\`\` 0: jdbc:phoenix:\> select \* from test; \`\`\`

\* 删除表内容

\`\`\` 0: jdbc:phoenix:\> delete from test where mykey=1; \`\`\`

\* 为表创建索引

\`\`\` 0: jdbc:phoenix:\> create local index test\_idx on
test(mycolumn);

\`\`\`

\* 查看表的索引信息

\`\`\` 0: jdbc:phoenix:\> \!indexes test \`\`\`

\* 删除表

\`\`\` 0: jdbc:phoenix:\> drop table if exists test; \`\`\`

\* 退出客户端

\`\`\` 0: jdbc:phoenix:\>\!exit

\`\`\`

4.使用 Phoenix 与已有 HBase 表建立表映射关系

\* 4.1 进入 HBase shell 命令行创建表并插入数据

\`\`\` \[root@uhadoop-xxx-master1 bin\]\# hbase shell

hbase(main):009:0\> create 'phoenix','info' hbase(main):010:0\> put
'phoenix', 'row001','<info:name>','phoenix' hbase(main):011:0\> scan
'phoenix'

\`\`\`

\* 4.2 进入 Phoenix python 客户端，建立映射关系

\*\* tips\*\*:

\* Phoenix 4.10 及以上的版本，Phoenix
对列对编码格式有所改变，官方文档：<http://phoenix.apache.org/columnencoding.html>
。

所以在建立与 HBase 映射关系时，需要设置 COLUMN \_ ENCODED \_ BYTES 属性为 0，即不让 Phoenix 对
column family 进行编码。

\* HBase 数据表默认主键列名是 ROW。

\* 为了防止 Phoenix 自动将小写 column 转大写，需要用双引号将 column 扩起来。

\* 如果映射关系建立成功，建标之后可以看到有多少行数据被影响。

\* 创建表映射

\`\`\` 0: jdbc:phoenix:\> create table "phoenix"("ROW" varchar primary
key, "info"."name" varchar) column\_encoded\_bytes=0; 1 row affected
(6.523 seconds) \`\`\`

\* 删除表（注意，这里hbase中表也会被同时删除）

\`\`\` 0: jdbc:phoenix:\> drop table "phoenix"; \`\`\`

5.使用 Phoenix 与已有 HBase 表建立视图映射关系

如果 Phoenix 表只是做查询操作对话，做表映射是个更好对选择，做表映射，如果在 Phoenix 客户端删除表，则 HBase
中该表也将被删除。

如果是用视图映射，则删除视图不会影响原有表的数据。

\* 5.1 创建视图映射

\`\`\` \# 如果 HBase 中不存在 Phoenix 表，需要按照 4.1 节的方式在 HBase 中建表 0:
jdbc:phoenix:\> create view "phoenix"("ROW" varchar primary key,
"info"."name" varchar); \`\`\`

\* 5.2 查询视图

\`\`\` 0: jdbc:phoenix:\>select \* from "phoenix"; \`\`\`

\* 5.3 删除视图

\`\`\` 0: jdbc:phoenix:\> drop view "phoenix"; \`\`\`
