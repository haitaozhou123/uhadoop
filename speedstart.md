{{indexmenu_n>5}}

# 快速上手

本文档将带领您如何创建UHadoop集群，并使用UHadoop集群完成数据处理任务。

## 创建集群

本章简单介绍了用户使用UHadoop服务时如何快速创建集群，如已创建完毕，请跳至第二章查看如何[提交任务](http://docs.ucloud.cn/analysis/uhadoop/speedstart#提交任务)。

\*\* 1、进入产品页面 \*\*

在“全部产品”菜单中点击“托管Hadoop集群 UHadoop”进入产品页面。

也可以将“托管Hadoop集群 UHadoop”设置为快捷方式，通过左侧快捷方式菜单栏点击进入。

\*\* 2、点击【创建集群】按钮 \*\*

\*\* 3、按需配置【基本设置】\*\*

\*\* 限制：\*\*
VPC和子网信息必填。详情参考[私有网络VPC](https://docs.ucloud.cn/network/vpc/vpc)。

\*\* 4、软件设置 \*\*

该模块提供集群软件、集群框架的选择。

**集群框架：**

根据应用场景的不同，可选择不同的集群框架。

Hadoop框架：集群中同时部署HDFS和YARN，适用于存储和计算在同一集群。

HDFS框架：集群中仅部署HDFS。用于做存储集群，有专属的HDFS节点机型。

计算框架：不部署HDFS，仅部署YARN。

HDFS框架和计算框架适用于存储计算分离架构。HDFS集群可作为多个独立计算集群的存储集群。

计算集群和存储集群（Hadoop框架、HDFS框架）的关系是多对一。可以在集群详情页看到已经联通的集群。

\*\* 限制：\*\*

1）创建计算集群前需要您已有HDFS集群或Hadoop框架的集群。

2）选择计算集群后，必须要指定【集群存储】，即指定计算集群读写数据的位置。

**发行版：**

发行版命名方式：uhadoop \[ 版本号 \]

每个发行版中有多个大数据生态软件，如HBase、Spark、Hive等。

**框架版本：**

集群中 Hadoop 的版本，不同发行版的框架版本不同。

**集群种类：**

不同种类代表集群会安装不同的集群软件。未在此处选择的软件，也可在集群创建完成后，通过集群管理添加。

\*\* 5、节点设置 \*\*

**节点配额总量：** 最多可创建的节点数量。如需更大配额，可联系客户经理或技术支持申请开通。

**Master节点：** 管理节点，负责协调整个集群服务。一个集群中有且仅有两个管理节点，一主一备，保证高可用。

除了基础服务（如Hadoop、Hive、HBase）的管理端部署在Master上外，一些插件（如Hue、Oozie、Sqoop2、Airflow）也会安装于Master节点上，因此，如若安装大量插件服务，Master节点配置建议高于C1-2xlarge。

**Core节点：**
核心节点，用于存储数据与运行任务。由于核心节点用于存储数据，因此数量须大于等于2（默认集群文件副本数配置为3），您可以根据业务需求添加更多的核心节点。

1.  **不同磁盘类型配置选择建议**

第一参考是数据量，数据量按照您需求的业务数据量\*3计算（HDFS默认将文件存储3份拷贝，来保证高可用）。

若数据量超过6T后，推荐使用密集存储系列节点（密集存储系列采用SATA硬盘，更适合海量数据的存储）。

若对磁盘性能和存储量都有需求，可使用物理机。

1.  **不同CPU、MEM机型的选择**

CPU、MEM的选择可按照计算复杂度与数据读写的频度，如果计算不是很复杂，小配置即可，如果复杂度较高，建议4核以上机型。

Spark对内存需求较大，建议选择12G MEM以上的机型。

**Task节点：** 任务节点，用于执行任务。任务节点不存储数据，您可以在集群运行期间动态进行添加和删除。

Task节点一般用于对整个集群CPU、MEM资源的补充，适合一些需大量消耗计算资源的任务，如若无法确定业务需求，可先不配置Task节点，后续根据需求再添加。

Task所有节点机型配置需保存一致，如若需要升降级，可先删除完所有Task节点，再次添加Task节点时，Console端允许您重新选择机型。

了解各节点配置详情，请参考[产品价格](https://docs.ucloud.cn/analysis/uhadoop/price)。

\*\* 6、访问设置 \*\*

填充节点root密码。

\*\* 7、等待集群部署 \*\*

根据集群规模不同，所需要的部署时间会有所差异，创建时间基本在15分钟左右。

## 提交任务

### 1、进入集群管理页面

在集群创建成功后，点击集群管理，进入集群节点详情页面。

### 2、登录集群

![](/images/uhadoop-12.png)

a) 通过控制台登录。

b) 绑定外网eip，本地可通过外网ssh连接登录。目前仅master节点支持绑定。Eip使用详情请见
[EIP说明文档](https://docs.ucloud.cn/network/unet/eip)。

本例中可通过\`ssh root@106.75.135.10 -p22\`进行登录。

c) 通过云主机（uhost）内网ssh进行登录。

本例中可在云主机上通过\`ssh root@10.13.186.23 -p22\`进行登录。

    登录密码为集群创建时设置的密码。

### 3、任务提交

3.1 利用hadoop命令查看hdfs目录信息

![](/images/uhadoop-13.png)

3.2 创建目录，并上传测试数据

```
[root@uhadoop-******-master1 ~]# hadoop fs -mkdir /input
[root@uhadoop-******-master1 ~]# hadoop fs -put /home/hadoop/conf/* /input
```

3.3 执行WordCount任务

```
[root@uhadoop-******-master1 ~]# hadoop jar /home/hadoop/hadoop-examples.jar wordcount /input /output
```

> 提示：如果/output目录已存在，请删除该目录或使用其他目录。

3.4 查看wordcount任务的结果

```
[root@uhadoop-******-master1 ~]# hadoop fs -cat /output/part-r-00000

!=  3
""  6
"". 4
"$HADOOP_CLASSPATH" 1
"$JAVA_HOME"    2
"$YARN_HEAPSIZE"    1
"$YARN_LOGFILE" 1
"$YARN_LOG_DIR" 1
"$YARN_POLICYFILE"  1
"*" 17

...
```

3.5 若集群安装了spark服务，可提交spark任务

    spark-submit  --master yarn --deploy-mode client --num-executors 1 --executor-cores 1 --executor-memory 1G $SPARK_HOME/examples/src/main/python/pi.py 100

屏幕信息中会打印任务执行结果：

    Pi is roughly 3.141313

更多使用内容，请参考
[UHadoop开发指南](https://doc.ucloud.cn/analysis/uhadoop/developer)
