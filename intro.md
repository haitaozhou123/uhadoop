{{indexmenu_n>1}}

# 产品简介

UHadoop是运行在ucloud平台上的一种大数据处理系统的托管服务。通过将Hadoop和Spark运行在云平台上，让用户可以非常方便的使用Hadoop和Spark生态系统中的其他周边系统（如Apache
Hive，Apache Pig，HBase等）来分析和处理自己的数据。

## 1、产品架构图

![](/images/jiagou.png)

1.1 HDFS

HDFS默认采用HA方式部署，2个Namenode分别部署于master1与master2，Datanode则分配在所有Core节点上，Task不部署Datanode。

![](/images/developer/hdfs.jpg)

1.2 Yarn

Yarn同样默认采用HA方式部署，2个ResourceManager分别部署于master1与master2，Nodemanager则分配在所有Core、Task节点上。

![](/images/developer/yarn.jpg)

1.3 Hive

Hive目前只支持on
yarn模式，2个Hive-MetaStore分别部署于master1与master2，并连接本地的mysql，避免了单个master节点宕机引起的Hive服务故障

可以通过HiveCli或者Beeline连接Hive服务。

![](/images/developer/hive.jpg)

1.4 HBase

HBase默认采用HA方式部署，2个HMaster分别部署于master1与master2，HRegionServer则分配在所有Core节点上。

![](/images/developer/hbase.jpg)

1.5 Spark

Spark采用On
Yarn模式，具体可参考[Spark开发指南](https://docs.ucloud.cn/analysis/uhadoop/developer/sparkdev)。

## 2、产品特色与优点

#### 方便

几分钟便可创建集群，无需为节点分配、部署、优化操心；借助丰富的示例和场景教程，快速上手达成业务目标。

#### 易用

1 按照所选硬件机型（CPU，内存， 磁盘），所选择软件组合和版本，进行自动化部署；

2 用户可以根据自己或数据源所处的地理位置申请相应将自己的集群部署在哪里。目前UHADOOP支持的地域包括北京二、广州、香港。后续会陆续开放到所有ucloud支持的地域。

#### 弹性

集群可大可小，并支持动态伸缩，有效避免资源浪费；支持计算与存储分离。

#### 开放

完全兼容开源社区版本的Hadoop/Spark，客户可以使用开源标准API编写作业，无需任何修改便可以迁移上云。

#### 安全

用户集群位于独享的虚拟私有网络中，实现资源的彻底隔离。

#### 稳定

集群内的Hadoop/Spark/HBase等关键组件都支持高可用特性，确保服务可用性。

