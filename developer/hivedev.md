{{indexmenu_n>2}}

## Hive开发指南

Hive是Hadoop生态系统中的数据仓库产品。它可以简单方便的存储、查询和分析存储在HDFS或者HBase的数据，它将sql语句转换成MapReduce任务，进行复杂的海量数据分析。它也提供了一系列工具，可用来多数据进行提取、转化和加载。

## 1. Hive Cli

Hive Cli是Hive服务提供的一个方便操作Hive表的客户端。其基本操作如下：

- 打开Hive Cli

在UHadoop任一master节点，或者安装了部署了Hive的客户端节点上执行hive即可：

```
[root@uhadoop-******-master1 ~]#hive
```

- 创建hive表

```
  hive> create table test_hive (id int, name string);
```

- 插入数据

```
  hive> insert into test_hive values (1,'test_ucloud'),(2,'test_hive');
```

- 读取数据

```
  hive> select * from test_hive;
```

- 统计数据个数

```
  hive> select count(*) from test_hive;
```

- 命令行直接执行sql命令

```
  hive -e "select * from test_hive"
```

## 2. Beeline

Hive提供了一个可通过JDBC方式调用的服务Hive-server2（UHadoop中默认安装在master2节点上，服务端口10000）。

利用beeline客户端可以远程连接Hive-server2服务，进而对hive数据进行操作。

- 启动beeline客户端

```
[root@uhadoop-******-master1 ~]# beeline
```

- 连接hive-server2

```
beeline> !connect jdbc:hive2://uhadoop-******-master2:10000/default;
```

> 注解：

1.  用户名、密码默认可填空值
2.  uhadoop-\*\*\*\*\*\*-master2须改成您集群master2节点的主机名称或者IP

- 数据操作

同Hive Cli

- 在命令行直接提交sql命令：

```
beeline -ujdbc:hive2://uhadoop-******-master2:10000  -e "select * from test_hive"
```

## 3. Hive应用开发

### 3.1 使用JAVA连接HiveServer2(实现创建表格、加载数据，展示数据操作)

此示例需要您先登陆UHadoop集群master1节点，以下操作默认在master1节点执行

org.apache.hive.jdbc.HiveDriver是hiveserver2的dirvername，hiveserver2的访问地址是"jdbc:hive2://ip:10000/default"

- 编写示例代码

示例代码Hive2JdbcClient.java如下：

``` java
import java.sql.SQLException;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
import java.sql.DriverManager;

public class Hive2JdbcClient {
    private static String driverName = "org.apache.hive.jdbc.HiveDriver";

    /**
     * @param args
     * @throws SQLException
     */
    public static void main(String[] args) throws SQLException {
        try {
            Class.forName(driverName);
        } catch (ClassNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            System.exit(1);
        }
        //replace "hive" here with the name of the user the queries should run as
        Connection con = DriverManager.getConnection("jdbc:hive2://uhadoop-******-master2:10000/default", "", "");
        Statement stmt = con.createStatement();
        String tableName = "testHive2DriverTable";
        stmt.execute("drop table if exists " + tableName);
        stmt.execute("create table " + tableName + " (key int, value string)");
        // show tables
        String sql = "show tables '" + tableName + "'";
        System.out.println("Running: " + sql);
        ResultSet res = stmt.executeQuery(sql);
        if (res.next()) {
            System.out.println(res.getString(1));
        }
        // describe table
        sql = "describe " + tableName;
        System.out.println("Running: " + sql);
        res = stmt.executeQuery(sql);
        while (res.next()) {
            System.out.println(res.getString(1) + "\t" + res.getString(2));
        }

        // load data into table
        // NOTE: filepath has to be local to the hive server
        // NOTE: /tmp/a.txt is a ctrl-A separated file with two fields per line
        String filepath = "/user/hive/warehouse/b.txt";
        sql = "load data inpath '" + filepath + "' into table " + tableName;
        System.out.println("Running: " + sql);
        stmt.execute(sql);

        // select * query
        sql = "select * from " + tableName;
        System.out.println("Running: " + sql);
        res = stmt.executeQuery(sql);
        while (res.next()) {
                System.out.println(String.valueOf(res.getInt(1)) + "\t" + res.getString(2));
        }

        // regular hive query
        sql = "select count(1) from " + tableName;
        System.out.println("Running: " + sql);
        res = stmt.executeQuery(sql);
        while (res.next()) {
            System.out.println(res.getString(1));
        }
    }
}
```

> 注解：
> 1. Connection con = DriverManager.getConnection("jdbc:hive2://uhadoop-\*\*\*\*\*\*-master2:10000/default", "", ""); 
> 2. uhadoop-\*\*\*\*\*\*-master2须改成您集群master2节点的主机名称或者IP

- 编译

```
  javac Hive2JdbcClient.java
```

- 执行程序

test.sh代码如下

```
#!/bin/bash
hdfs dfs -rm /user/hive/warehouse/b.txt
echo -e '1\x01foo' > /tmp/b.txt
echo -e '2\x01bar' >> /tmp/b.txt
hdfs dfs -put /tmp/b.txt /user/hive/warehouse/
HADOOP_HOME=/home/hadoop/
CLASSPATH=.:$HIVE_HOME/conf
for i in ${HADOOP_HOME}/share/hadoop/mapreduce/lib/hadoop-*.jar ; do
    CLASSPATH=$CLASSPATH:$i
done
for i in ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-*.jar ; do
    CLASSPATH=$CLASSPATH:$i
done
for i in ${HADOOP_HOME}/share/hadoop/common/lib/hadoop-*.jar ; do
    CLASSPATH=$CLASSPATH:$i
done
for i in ${HADOOP_HOME}/share/hadoop/common/hadoop-*.jar ; do
    CLASSPATH=$CLASSPATH:$i
done

for i in ${HIVE_HOME}/lib/*.jar ; do
    CLASSPATH=$CLASSPATH:$i
done
java -cp $CLASSPATH Hive2JdbcClient
```

### 3.2 使用Python连接HiveServer2(实现创建表格、加载数据，展示数据操作)

Hiveserver2使用python客户端的过程如下：

- 下载pyhs2 git clone <https://github.com/BradRuderman/pyhs2.git>

- 安装依赖 yum install gcc-c++ cyrus-sasl-\* python-devel

- 安装setuptools wget -q <http://peak.telecommunity.com/dist/ez_setup.py>
./python ez\_setup.py

> 如果上面方式安装失败需要手动下载setuptools-0.6c11.tar.gz安装包安装

- 编译安装pyhs2

进入pyhs2目录并安装

```
cd pyhs2
python setup.py build
python setup.py install
```

- 编写示例代码

示例代码，即pyhs2下example.py

``` python
import pyhs2

with pyhs2.connect(host='uhadoop-******-master2',
                   port=10000,
                   authMechanism="PLAIN",
                   user='root',
                   password='test',
                   database='default') as conn:
    with conn.cursor() as cur:
        #Show databases
        print cur.getDatabases()

        #Execute query
        cur.execute("select * from test_hive")

        #Return column info from query
        print cur.getSchema()

        #Fetch table results
        for i in cur.fetch():
            print i
```

> uhadoop-\*\*\*\*\*\*-master2须改成您集群master2节点的主机名称或者IP,
> test\_hive须修改为hive中已经有的table名称

### 3.3 Hive外表读取HBase数据

通过在Hive中创建HBase外表，可利用简单的sql语句分析HBase的非结构化数据

- 打开HBase shell，创建t1表

```
create 't1',{NAME => 'f1',VERSIONS => 2}
put 't1','rowkey001','f1:col1','value01' 
put 't1','rowkey001','f1:col2','value02' 
put 't1','rowkey001','f1:colf','value03' 
scan 't1' 
```

得到的t1表结构如下

```
hbase(main):013:0> scan 't1'
ROW                                            COLUMN+CELL
 rowkey001                                     column=f1:col1, timestamp=1481075364575, value=value01
 rowkey001                                     column=f1:col2, timestamp=1481075364607, value=value02
 rowkey001                                     column=f1:colf, timestamp=1481075364641, value=value03
```

- 打开Hive Cli，创建外表

```
hive> CREATE EXTERNAL TABLE t_hive_hbase(
    > rowkey string,
    > cf map<STRING,STRING>
    > )
    > STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    > WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f1:")
    > TBLPROPERTIES ("hbase.table.name" = "t1");
```

- 使用sql语句读取hbase数据，结果如下

```
hive> select * from  t_hive_hbase;
OK
rowkey001   {"col1":"value01","col2":"value02","colf":"value03"}
```

## 4\. 常用增强特性

### 4.1 支持自定义UDF函数

下面提供2种方式用于编写udf函数，第一种是通过Eclipse，第二种是直接在UHadoop集群的master节点上利用shell脚本编译

#### 在Eclipse中编写自定义Hive udf函数

1 在eclipse中新建java project: hiveudf

2 在hiveudf工作中添加一个文件夹lib，lib包需要拷贝相关jar包，添加完毕后如下图所示

![](/images/hive-develop-01.jpg)

这四个包在可在uhadoop-\*\*\*\*\*\*-master1的以下位置找到

    /home/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.6.0-cdh5.4.9.jar
    /home/hadoop/share/hadoop/hdfs/hadoop-hdfs-2.6.0-cdh5.4.9.jar
    /home/hadoop/share/hadoop/common/hadoop-common-2.6.0-cdh5.4.9.jar
    /home/hadoop/hive/lib/hive-exec-1.2.1.jar

3 把lib目录下的所有jar包 Add to build Path

![](/images/hive-develop-02.jpg)

4 创建一个UDFLower 类并且继承hive的UDF类，新建的UDFLower类的package请填空值

``` java
import org.apache.hadoop.hive.ql.exec.UDF;  
import org.apache.hadoop.io.Text;  

public class UDFLower extends UDF{  
     public Text evaluate(final Text s){  
          if (null == s){  
              return null;  
          }  
          return new Text(s.toString().toLowerCase());  
      }  
} 
```

最终的结构如图所示

![](/images/hive-develop-03.jpg)

5 打包UDFLower.jar

-- 执行导出

![](/images/hive-develop-04.jpg)

-- 选择JAR file

![](/images/hive-develop-05.jpg)

-- 修改导出目录名称，并点击Finish，UDFLower.jar便制作完成了

![](/images/hive-develop-06.jpg)

-- 通过ssh命令上传至uhadoop master1节点

如果master1已绑定外网，可直接通过外网IP上传，如果没有，请通过有外网IP的机器跳转

#### 在Linux下编写自定义Hive udf函数

- 创建代码

登录uhadoop-\*\*\*\*\*\*-master1节点，进入/data目录，创建udf代码

```
  cd /data
  mkdir hive_udf
  touch UDFLower.java
```

UDFLower.java代码如下

``` java
import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;

public class UDFLower extends UDF{
     public Text evaluate(final Text s){
          if (null == s){
              return null;
          }
          return new Text(s.toString().toLowerCase());
      }
}
```

- 同目录下创建编译文件compile.sh

```
#!/bin/bash

rm -f UDFLower.jar
rm -f UDFLower*.class
export HADOOP_MAPRED_HOME=/home/hadoop
CLASSPATH=.:$HIVE_HOME/conf

for i in /home/hadoop/share/hadoop/common/*.jar ; do
    CLASSPATH=$CLASSPATH:$i
done

for i in /home/hadoop/hive/lib/*.jar ; do
    CLASSPATH=$CLASSPATH:$i
done
javac -cp $CLASSPATH UDFLower.java
jar cvf UDFLower.jar UDFLower.class
```

执行sh compile.sh后将在同目录下产生UDFLower.jar

#### 利用上述生成的UDFLower.jar进行Hive示例

- 进入UDFLower.jar所在目录，上传至HDFS目录

    hadoop fs -put UDFLower.jar /tmp/

- 测试数据准备

测试文件test.txt，内容如下

``` 
HELLO WORLD HEHE

```

上传至HDFS

    hadoop fs -put test.txt /tmp/

- Hive Cli中创建相关数据表，并load数据

```
hive> create table test_hive_udf (name string);
hive> load data inpath  '/test/1/test.txt' into table test_hive_udf;
```

- 创建temporary function

```
hive> create temporary function my_lower as 'UDFLower';
```

- 使用自定义UDF

```
hive> select name from test_hive_udf;
hive> select my_lower(name) from test_hive_udf;
```

### 4.2 支持json格式数据

- 数据准备

上传数据test.json数据到hdfs

test.json数据如下

    {"id":1234544353,"text":"test_hive_json"}

上传数据到HDFS

    hdfs dfs -put test.json /tmp/test.json

- 加载依赖包

hive解析json格式的数据依赖hive-json-serde.jar，如果通过beeline无需再通过add
jar加载对应的jar包。如果你使用的hive cli接口需要add jar，做如下操作：

```
hive> add jar $HIVE_HOME/lib/hive-json-serde.jar;
```

> hive1.2.1版本尚未提供此包

- 创建hive表格

```
hive> CREATE TABLE test_json(id BIGINT,text STRING)ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.JsonSerde' STORED AS TEXTFILE;
```

- 加载数据

```
hive> LOAD DATA  INPATH "/tmp/test.json" OVERWRITE INTO TABLE test_json;
```

- 执行查询

返回如下说明使用json文件解析成功

```
hive> select * from test_json;
OK
1234544353  test_hive_json
```

### 4.3 使用正则匹配

- 数据准备

上传测试数据nginx\_log到hdfs

nginx\_log数据如下

    180.173.250.74 168.34.34

上传至HDFS

``` 
hdfs dfs -put nginx_log /tmp/nginx_log

```

- 加载依赖包

hive正则匹配使用依赖hive-contrib.jar，如果通过beeline无需再通过add jar加载对应的jar包。如果你使用的hive
cli接口需要add jar，做如下操作：

```
hive> add jar $HIVE_HOME/lib/hive-contrib.jar;
```

- 创建测试表格

```
hive> CREATE TABLE logs(
       host STRING,
       identity STRING)
         ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.RegexSerDe'
         WITH SERDEPROPERTIES (
         "input.regex" = "([^ ]*) ([^ ]*)",
         "output.format.string" = "%1$s %2$s"
)STORED AS TEXTFILE;
```

> 注解：创建表格的时候需要指定ROW FORMAT SERDE，SERDEPROPERTIES中的
> input.regex和output.format.string

- 加载数据

```
hive> LOAD DATA INPATH "/tmp/nginx_log" INTO TABLE logs;
```

- 测试数据

返回如下说明正则表达式使用成功

```
hive> select * from logs;
OK
180.173.250.74  168.34.34
```
