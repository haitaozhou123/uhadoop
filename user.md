{{indexmenu_n>45}}

# FAQs

## 如何登陆集群

1.使用相同region的uhost登陆，
[faq\#我的主机只有内网IP，可以通过哪些方式访问这台机器呢？](/network/unet/faq#我的主机只有内网IP，可以通过哪些方式访问这台机器呢？)

2.master节点可以绑定外网IP，直接使用外网IP登陆。

## 使用web服务无法查看其它core/task节点log信息

配置了的外网IP后，需要配置代理访问的方式才可以查看重定向后的url信息。在绑定外网的主机上面执行下面的操作：

1.  安装nginx

yum install nginx -y

1.  修改配置

新建/etc/nginx/conf.d/proxy.conf 文件中添加如下配置

``` 
server {
    listen       8889;
    client_body_timeout 60000;
    client_max_body_size 1024m;
    send_timeout       60000;
    client_header_buffer_size 16k;
    large_client_header_buffers 4 64k;

    proxy_headers_hash_bucket_size 1024;
    proxy_headers_hash_max_size 4096;
    proxy_read_timeout 60000;
    proxy_send_timeout 60000;

    location / {
        resolver 127.0.0.1;
        proxy_pass http://$http_host$request_uri;
    }
} 
```

1.  启动nginx服务

service nginx restart

1.  启动域名服务

service dnsmasq restart ； 集群节点发生变化时，需要重新启动这个服务。

1.  在访问的网页端配置代理

以windows为例：在命令行下执行 set http\_proxy=<http://proxy_ip:port> proxy\_ip
为上面启动nginx的ip，port为上面监听的端口8889.

1.  配置hosts

需要在本地的hosts中添加集群的/etc/hosts 文件中节点的host信息。

## hue中执行hive sql提示Fetching results ran into the following error(s): Couldn't find log associated with operation handle: OperationHandle

1\.
确认hive-site.xml里是否配置了hive.server2.logging.operation.log.location参数，如果配置了，检查是否hadoop用户有权限访问这个路径

2\.
确认/tmp/hadoop/operation\_logs/这个目录是否存在，如果没有重启hiveserver2服务：master2$service
hive-server2 restart

## Namenode一直GC

这种问题是因为hdfs 小文件太多，而master配置不够，导致namenode加载元数据时内存被耗尽且未加载完，致使namenode一直在gc

解决方法：

1.  升级 master
2.  删除hdfs上的小文件
3.  调整Namenode的/home/hadoop/conf/hadoop-env.sh中的HADOOP\_NAMENODE\_HEAPSIZE

## 查看日志

### 任务日志

1\. 使用hue查看 2. 使用web yarn 查看 3. 当日志文件过于庞大时，上面两个服务没有很好的支持，只能通过下载hdfs日志查看。

hdfs://Ucluster/var/log/hadoop-yarn/apps/\[submintuser\]/logs
(submituser 是交用户的名)

### 服务日志

服务日志位于/var/log/{servicename}目录下

## 使用Hive UDF 中文结果无法识别

这种情况是由于udf产生的结果和hive设置的中文字符集不一致，使用udf产生的中文结果作为hive语法的输入时，会被当做NULL字符。可以通过将udf的结果强制转换为UTF-8解决。

一段java代码示例

    public class UDF2UTF8 extends UDF{  
        public String evaluate(final String s){  
            if (null == s){  
                return null;  
            }
            String ret = "";
            try {
                ret = new String(s.toString().getBytes(), "UTF-8");
            } catch (UnsupportedEncodingException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
    
            return ret;  
        }  
    }

## ResourceManager 启动失败

Caused by: java.net.BindException: Cannot assign requested address

1\. 原ResourceManager未关闭

2.拷贝了另一台机器的的yarn-site.xml，导致两个master的yarn.resourcemanager.ha.id参数配置冲突，修改yarn.resourcemanager.ha.id为对应标识(rm1或者rm2)

## 启动服务时报xx端口被占用了

UHadoop 会在后台自动拉起主要服务，用户在手动重启时可能使用了错误的脚本。

启动服务方法如下：

```
# service hadoop-hdfs-datanode restart
# service hive-metastore restart
# service hive-server2 restart
# service hbase-master restart
# service hbase-regionserver restart
# service hadoop-mapreduce-historyserver restart
# service hadoop-hdfs-journalnode restart
# service hadoop-hdfs-namenode restart
# service hadoop-yarn-nodemanager restart
# service hadoop-yarn-resourcemanager restart
# service hadoop-hdfs-zkfc restart
# service zookeeper-server restart
# service AgentMonitor restart
# service mysqld restart
```

## hive-metastore写mysql失败

在任务后面，需要修改hive元数据时，遇到hive-metastore写mysql失败。

hive-metastore日志报下面的错误

Caused by:
com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: No
operations allowed after connection closed.

is blocked because of many connection errors; unblock with 'mysqladmin
flush-hosts'"

Failed to acquire connection to jdbc:mysql://.... Sleeping for 7000 ms.
Attempts left

问题原因：mysql默认有一个wait\_timeout限制，默认为8小时，当连接没有活跃时，8小时后自动断开。

解决方法：

1.  检查/home/hadoop/hive-site.xml文件中javax.jdo.option.ConnectionURL参数是否是下面内容:
    



``` java
jdbc:mysql://<instance_id>-master1:3306/hive?createDatabaseIfNotExist=true&amp;autoReconnect=true&amp;characterEncoding=UTF-8>
```

1.  修改mysql my.cnf 修改wait\_timeout参数，增大。

## HBase导入数据时连接不上

出现：thrift.transport.TTransport.TTransportException: Could not connect to
10.10.10.10:9090

解决方法：

1.  使用batch put方法

java示例代码片段：

``` java
List<Row> batch = new ArrayList<Row>();
Put put = new Put(ROW1);
put.addColumn(COLFAM1, QUAL1, 2L, Bytes.toBytes("val2"));
batch.add(put);
```

完整代码

``` java
package client;

// cc BatchSameRowExample Example application using batch operations, modifying the same row
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Get;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Row;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.util.Bytes;

import util.HBaseHelper;

public class BatchSameRowExample {

private final static byte[] ROW1 = Bytes.toBytes("row1");
private final static byte[] COLFAM1 = Bytes.toBytes("colfam1");
private final static byte[] QUAL1 = Bytes.toBytes("qual1");

public static void main(String[] args) throws IOException, InterruptedException {
    Configuration conf = HBaseConfiguration.create();

    HBaseHelper helper = HBaseHelper.getHelper(conf);
    helper.dropTable("testtable");
    helper.createTable("testtable", "colfam1");
    helper.put("testtable", "row1", "colfam1", "qual1", 1L, "val1");
    System.out.println("Before batch call...");
    helper.dump("testtable", new String[] { "row1" }, null, null);

    Connection connection = ConnectionFactory.createConnection(conf);
    Table table = connection.getTable(TableName.valueOf("testtable"));

    // vv BatchSameRowExample
    List<Row> batch = new ArrayList<Row>();

    Put put = new Put(ROW1);
    put.addColumn(COLFAM1, QUAL1, 2L, Bytes.toBytes("val2"));
    batch.add(put);

    Get get1 = new Get(ROW1);
    get1.addColumn(COLFAM1, QUAL1);
    batch.add(get1);

    Delete delete = new Delete(ROW1);
    delete.addColumns(COLFAM1, QUAL1, 3L); // co BatchSameRowExample-1-AddDelete Delete the row that was just put above.
    batch.add(delete);

    Get get2 = new Get(ROW1);
    get1.addColumn(COLFAM1, QUAL1);
    batch.add(get2);

    Object[] results = new Object[batch.size()];
    try {
    table.batch(batch, results);
    } catch (Exception e) {
    System.err.println("Error: " + e);
    }
    // ^^ BatchSameRowExample

    for (int i = 0; i < results.length; i++) {
    System.out.println("Result[" + i + "]: type = " +
        results[i].getClass().getSimpleName() + "; " + results[i]);
    }
    table.close();
    connection.close();
    System.out.println("After batch call...");
    helper.dump("testtable", new String[]{"row1"}, null, null);
    helper.close();
}
}
```

1.  每一个put操作实际上都是一个RPC操作，它将客户端数据传送到服务器然后返回，这只适合小数据量操作，如果有个程序需要每秒存储上千行数据到HBase表中，这样的处理就不太合适了。（减少独立RPC调用的关键是限制往返时间，往返时间就是客户端发送一个请求到服务器，然后服务器通过网络进行响应的时间。）HBase的API提供了写缓存区，它负责收集put操作，然后调用RPC一次性将put送往服务器。

默认客户端缓冲区是禁止的，可以通过设置自动刷写为false来激活写缓冲区。table.setAutoFlush(false)
;table.flushCommits();
一旦超过缓冲指定大小，客户端会隐式地调用刷写命令。用户可以配置客户端缓冲区大小：setWriteBufferSize(long
size),默认是2MB（即2097152字节） 也可以在hbase-site.xml中设置：

``` java
<property>
    <name>hbase.client.write.buffer</name> 
    <value>20971520</value>
</property>
```

## HBase访问压力集中在一个节点上面

通过HBase Web UI 观察确认，访问地址marter\_ip:60010

![image](/images/pic_hbase.png)

点击相应的regionserver链接，可以查看表的访问情况。

Hbase-site.xml里设置下面两个参数可以对现有的分区做均衡。

hbase.hregion.max.filesize单个ColumnFamily的region大小

hbase.regionserver.region.split.policy 配置为ConstantSizeRegionSplitPolicy

## Jdbc 链接hive执行sql卡主

使用 jdbc:hive2://10.10.10.10:10001/default
这种方式链接hive时，如果链接没有在使用完后关闭连接，会导致zookeeper连接异常。

避免在一个程序中重复连接jdbc端口，使用全局变量保存连接；或者每次执行语句时重新连接，执行完毕后关闭jdbc连接。

## UnkownHostException

提交任务的客户端未配置好host信息，把UHadoop master1机器上的hosts中相关信息拷贝到云主机的/etc/hosts

## 系统盘被不明文件占用了

系统盘显示使用率满了，但是找不到对应文件。

数据盘失效后，目录会在重启系统过后挂载到系统盘下面，这个时候由于服务还在运转，有部分数据是写到/data下的，此时占用的上系统盘空间，后来有把数据盘挂载到/data下，继续往/data里写数据，此时占用的是数据盘的空间。

解决方法：关闭使用/data的服务,根据所在节点不同，选择不同的服务停止。禁止自动拉起服务：/etc/default/static\_conf

1.  禁止自动拉起服务：/etc/default/static\_conf.json文件中将startmonitor 值修改为0
2.  停止服务

service hadoop-hdfs-journalnode stop

service hadoop-yarn-nodemanager stop

service hadoop-yarn-resourcemanager stop

service hadoop-hdfs-datanode stop

service hadoop-hdfs-namenode stop

service zookeeper-server stop

1.  备份/data\*/ 的内容 
2.  umount /data\* 卸载曾经出问题的目录。 
3.  删除/data\* 中的内容 
4.  重新挂载mount /dev/vd\* /data\* 
5.  使能自动拉起服务：/etc/default/static\_conf.json文件中将startmonitor 值修改为1

## 数据迁移

### HDFS迁移

1.  拷贝单个目录或文件

hadoop distcp -i <hdfs://nn1:8020/a.dir hdfs://nn2:8020/b.dir>

将nn1所在集群上目录A拷贝到nn2所在集群目的b上,注意，要保证nn1集群对nn2所有节点直接网络互通。

新集群各应用HDFS存储目录

hbase /hbase

hive /user/hive/warehouse

任务日志 /var/log

详情：<http://hadoop.apache.org/docs/r1.0.4/cn/distcp.html>

### Hive迁移

1.  设置默认需要导出的hive数据库为defaultDatabase



```
vi ~/.hiverc
use defaultDatabase;
```

1.  创建数据临时目录



```
hdfs dfs -mkdir /tmp/hive-export
```

1.  生成导出数据脚本



```
hive -e "show tables" | awk '{printf "export table %s to @</tmp/hive-export/%s@;>\n",$1,$1}' | sed <"s/@/'/g>" > export.sql
```

1.  手工导出数据到HDFS



```
hive -f export.sql
```

1.  下载HDFS数据到本地并传送到目标集群的/tmp/hive-export目录



```
hdfs dfs -get /tmp/hive-export/
scp -r hive-export/ export.sql <root@targetDir>
hdfs dfs -put hive-export/ /tmp/hive-export
```

1.  构造导入语句



```
cp export.sql import.sql
sed -i 's/export table/import table/g' import.sql
sed -i 's/ to / from /g' import.sql
```

1.  导入数据



```
hive -f import.sql
```

更多内容见：
<https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ImportExport#LanguageManualImportExport-Examples>

### HBase迁移

可参考 <http://www.tuicool.com/articles/QJFn22E>

设置主从备份，只拷贝新增数据

1.  开启replication：主从集群hbase-site.xml添加（待确认是否默认就为true，如果默认true，可省略）



    <property>
        <name>hbase.replication</name>
        <value>true</value>
    </property>

1.  new cluster和old cluster所有节点加上对方的hosts
2.  new Cluster 手动建表和簇名
3.  修改表定义，开启复制功能



    disable 'your_table'
    alter 'your_table', {NAME => 'family_name', REPLICATION_SCOPE => '1'}
    enable 'your_table'

1.  添加peer



    add_peer 'ID', 'zk1,zk2,zk3:2181:/hbase'

**zk1，需要使用ip，host名字会被replication的zookeeper拒绝连接**

此ID应和上述REPLICATION\_SCOPE相同，add\_peer后自动开启复制

使用disable\_peer 'peer\_id'停止复制

1.  迁移原HDFS的HBase数据

hbase org.apache.hadoop.hbase.mapreduce.CopyTable --peer.adr=
zk1,zk2,zk3:2181:/hbase table\_name 逐个表拷贝。

1.  恢复元数据与数据

在新集群执行，hbase hbck， 如果发现数据有异常进行下面操作：

    hbase hbck -fixMeta
    hbase hbck -repair

## 调整spark任务日志级别

1.  修改提交任务机器的这个文件/home/hadoop/spark/conf/log4j.properties

log4j.rootCategory=OFF, console

日志级别有以下5种

OFF、FATAL、ERROR、WARN、INFO、DEBUG、TRACE、ALL

1.  在程序中指定



    val sc = new SparkContext(config)
    sc.setLogLevel("WARN")

## spark连接mysql问题

1.java.lang.ClassNotFoundException: com.mysql.jdbc.Driver

原因，默认集群没有连接mysql的jar包。

2.java.sql.SQLException: No suitable driver found for
jdbc:mysql//localhost:3306/mysql

连接的语法错误会报这个错误，仔细检查，连接的url是否存在问题。

3.access denied for user….

需要为每个nodemanager所在的机器(task/core节点)配置访问mysql的权限。

4.旧的集群没有配置mysql的jar包，
需要手动从http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.0.8.tar.gz下载。

测试命令：

启动命令行模式：

    $spark-shell --jars mysql-connector-java-5.0.8/mysql-connector-java-5.0.8-bin.jar

测试命令：

    sqlContext.read.format("jdbc").options(Map("url" -> "jdbc:mysql:%%//%%localhost:3306/mysql", "driver" -> "com.mysql.jdbc.Driver", "dbtable" -> "user", "user" -> "root", "password" -> "")).load();
