{{indexmenu_n>3}}

# HBase开发指南

HBase是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，它支持通过key/value存储来支持实时分析，也支持通过map-reduce支持批处理分析。

## 1. HBase shell

Hbase shell是简单的，通过shell与HBase交互的方式，以下介绍使用方法：

#### 1.1 启动shell

```
[root@uhadoop-******-master1 ~]# hbase shell
2016-12-06 11:00:08,624 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.0-cdh5.8.0, rUnknown, Tue Jul 12 16:11:18 PDT 2016
```

#### 1.2 创建表格，并插入3条数据

```
hbase(main):004:0> create 'test_ucloud', 'cf'
0 row(s) in 1.2280 seconds

=> Hbase::Table - test_ucloud
hbase(main):005:0> list 'test_ucloud'
TABLE
test_ucloud
1 row(s) in 0.0180 seconds

=> ["test_ucloud"]
hbase(main):006:0> put 'test_ucloud', 'row1', 'cf:a', 'value1'
0 row(s) in 0.1780 seconds

hbase(main):007:0> put 'test_ucloud', 'row2', 'cf:b', 'value2'
0 row(s) in 0.0060 seconds

hbase(main):008:0> put 'test_ucloud', 'row3', 'cf:c', 'value3'
0 row(s) in 0.0060 seconds
```

#### 1.3 读取全部数据

```
hbase(main):009:0> scan 'test_ucloud'
ROW                                            COLUMN+CELL
 row1                                          column=cf:a, timestamp=1480993562401, value=value1
 row2                                          column=cf:b, timestamp=1480993575088, value=value2
 row3                                          column=cf:c, timestamp=1480993587152, value=value3
3 row(s) in 0.0610 seconds
```

#### 1.4 get一行数据

```
hbase(main):010:0> get 'test_ucloud', 'row1'
COLUMN                                         CELL
 cf:a                                          timestamp=1480993562401, value=value1
1 row(s) in 0.0090 seconds
```

#### 1.5 删除表

```
hbase(main):011:0> disable 'test_ucloud'
0 row(s) in 2.3550 seconds

hbase(main):012:0> drop 'test_ucloud'
0 row(s) in 1.4980 seconds
```

#### 1.6 退出HBase shell

```
hbase(main):013:0> exit
```

## 2. 为Hbase开启LZO压缩

UHadoop集群中HBase已经默认配置支持LZO压缩，只需修改表COMPRESSION属性即可。

#### 2.1 创建LZO表

```
hbase(main):001:0> create 'test-lzo', {NAME=>'cf', COMPRESSION=>'lzo'}
0 row(s) in 4.5140 seconds
=> Hbase::Table - test-lzo

hbase(main):002:0> desc 'test-lzo'
Table test-lzo is ENABLED
test-lzo
COLUMN FAMILIES DESCRIPTION
{NAME => 'cf', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATION_SCOPE => '0', VERSIONS => '1', COMPRESSION => 'LZO', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELE
TED_CELLS => 'FALSE', BLOCKSIZE => '65536', IN_MEMORY => 'false', BLOCKCACHE => 'true'}
1 row(s) in 0.1470 seconds
```

#### 2.2 为已存在的表增加COMPRESSION=LZO属性

- 假设您已创建test-nolzo表，如没有，可通过下面的语句创建

```
create 'test-nolzo', {NAME=>'cf'} 
```

- 增加COMPRESSION属性需要先distable表，再修改COMPRESSION属性为LZO，最后enable表。

```
hbase(main):002:0> disable 'test-nolzo'
0 row(s) in 2.4030 seconds

hbase(main):003:0> alter 'test-nolzo',NAME => 'cf', COMPRESSION => 'LZO'
Updating all regions with the new schema...
1/1 regions updated.
Done.
0 row(s) in 1.9730 seconds

hbase(main):004:0> enable 'test-nolzo'
0 row(s) in 1.2140 seconds
```

- 修改COMPRESSION后并不会压缩原有数据，可通过执行以下命令强制执行

```
hbase(main):005:0> major_compact 'test-nolzo'
0 row(s) in 0.5060 seconds
```

> 注解：
> 
> 数据量大的表此操作需等待较久时间。

## 3. 使用Hive读取HBase数据

请参考[Hive应用开发](https://docs.ucloud.cn/analysis/uhadoop/developer/hivedev#hive外表读取hbase数据)

## 4. HBase应用开发

### 4.1 使用JAVA读取HBase（实现创建表格、插入数据，展示数据操作）

此示例需要您先登陆UHadoop集群master1节点，以下操作默认在master1节点执行

#### 4.1.1 构建JAVA代码

```
mkdir  -p /data/hbase-example
cd /data/hbase-example
touch HbaseJob.java
```

HbaseJob.java代码如下

``` java
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.KeyValue;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Get;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.util.Bytes;

public class HbaseJob {
    static Configuration conf=null;
    static{
        conf=HBaseConfiguration.create();//hbase的配置信息
    }
    public static void main(String[] args)throws Exception {
        HbaseJob t=new HbaseJob();
        t.createTable("person", new String[]{"name","age"});
        t.insertRow("person", "1", "age", "hehe", "100");
        t.insertRow("person", "2", "age", "haha", "101");
        t.showAll("person");
    }

    /***
     * 创建一张表
     * 并指定列簇
     * */
    public void createTable(String tableName,String cols[])throws Exception{
        HBaseAdmin admin=new HBaseAdmin(conf);//客户端管理工具类
        if(admin.tableExists(tableName)){
            System.out.println("此表已经存在.......");
        }else{
            HTableDescriptor table=new HTableDescriptor(tableName);
            for(String c:cols){
                HColumnDescriptor col=new HColumnDescriptor(c);//列簇名
                table.addFamily(col);//添加到此表中
            }
            admin.createTable(table);//创建一个表
            admin.close();
            System.out.println("创建表成功!");
        }
    }

    public  void insertRow(String tableName, String row,
            String columnFamily, String column, String value) throws Exception {
        HTable table = new HTable(conf, tableName);
        Put put = new Put(Bytes.toBytes(row));
        put.add(Bytes.toBytes(columnFamily), Bytes.toBytes(column),
                Bytes.toBytes(value));
        table.put(put);
        table.close();//关闭
        System.out.println("插入一条数据成功!");
    }

    public void showAll(String tableName)throws Exception{
        HTable h=new HTable(conf, tableName);
        Scan scan=new Scan();
        ResultScanner scanner=h.getScanner(scan);
        for(Result r:scanner){
            System.out.println("====");
            for(KeyValue k:r.raw()){
                System.out.println("行号:  "+Bytes.toStringBinary(k.getRow()));
                System.out.println("时间戳:  "+k.getTimestamp());
                System.out.println("列簇:  "+Bytes.toStringBinary(k.getFamily()));
                System.out.println("列:  "+Bytes.toStringBinary(k.getQualifier()));
                String ss=  Bytes.toString(k.getValue());
                System.out.println("值:  "+ss);
            }
        }
        h.close();
    }
}
```

#### 4.1.2 构建编译程序

- 创建编译目录和文件

```
cd /data/hbase-example
touch hbase-test.sh
```

- hbase-test.sh 代码如下：

```
#!/bin/bash
HBASE_HOME=/home/hadoop/hbase
CLASSPATH=.:$HBASE_HOME/conf/hbase-site.xml

for i in ${HBASE_HOME}/lib/*.jar ;
do
      CLASSPATH=$CLASSPATH:$i
done
#编译程序
javac -cp $CLASSPATH HbaseJob.java
#执行程序
java -cp $CLASSPATH HbaseJob
```

#### 4.1.3 执行HBase程序

编译程序目录下执行

```
sh hbase-test.sh
```

执行结果如下：

```
创建表成功!
插入一条数据成功!
插入一条数据成功!
====
行号:  1
时间戳:  1480991139173
列簇:  age
列:  hehe
值:  100
====
行号:  2
时间戳:  1480991139240
列簇:  age
列:  haha
值:  101
```

## 5. HBase日常运维操作

以下操作需要在UHadoop集群master节点下以hadoop用户下执行，否则会出现权限不足提示

- 查看hbase region状态信息查询

```
hbase hbck
```

- 存在inconsistencies region修复 -

```
hbase hbck -repair
```

- 修复hbase空洞 -

```
hbase hbck -fixHdfsHoles
```

- 修复meta信息（根据meta表，将表上的region分给regionserver）

```
hbase hbck -fixMeta
```

- 重新修复meta表（根据hdfs上的regioninfo文件，生成meta表）

```
hbase hbck -fixAssignments
```

- 开启region自动均衡

需要在hbase shell下开启：

```
[root@uhadoop-******-master1 ~]# hbase shell
2016-12-06 11:00:08,624 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.0-cdh5.8.0, rUnknown, Tue Jul 12 16:11:18 PDT 2016
hbase(main):001:0> balance_switch true
```
