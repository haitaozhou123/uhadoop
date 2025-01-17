{{indexmenu_n>1}}

# 故障排查

## 任务执行失败

### 1.查看console输出日志

查看任务执行时控制台输出的log，检查是否有ERROR

### 2.查看任务执行日志

若任务为后台执行或定时任务，首先需要知道失败的任务ID，可通过获取日志详情分析日志报错（查询日志方法可见[日志查看](https://docs.ucloud.cn/analysis/uhadoop/operate/general)）

Tips:
常见任务失败ERROR可参考[常见任务ERROR](https://docs.ucloud.cn/analysis/uhadoop/user/error)

## 排查工具

### 1.查看监控

\- 在集群的“监控视图”页面查看集群或者节点的监控数据，判断是否有异常。

### 2.查看服务日志

\- 各个节点上的/var/log下面有各个服务的日志 - 通过web-yarn的页面或者hue可以查看到任务运行的日志情况

## 故障描述

### 1.问题描述

在向技术支持提交故障时，可以在提交信息中附带以下内容以便快速定位故障：

\- 群集的标识符 - 启动群集的区域和可用区 - 如何操作会出现这个异常 - 异常的现象的具体描述

### 2.检查集群的配置修改

\- 上一次正确运行的配置和环境变量是否有做修改。

### 3.检查日志

通常提交的任务可以在hadoop-yarn的界面可以看到，如无法查看任务通常有以下几种情况：

\- spark任务用本地模式提交 - hive任务用本地提交（hive-server2默认会将一些小任务用本地模式跑）

## 集群运行速度慢

### 1.检查集群配置修改

### 2.检查日志

\- 检查任务日志，如果一个或多个失败任务，请调查对应的任务尝试的日志，以了解更详细的错误信息。 -
检查服务日志，在每个节点的/var/log目录下，每个服务都有各自的存档目录。集群运行慢时，通常会在日志中可以查找到明显的异常，或者花费时间长的操作。

### 3\. 检查集群节点的运行状态

\- master：管理群集上部署的各种服务。如果主节点遇到性能问题，整个群集都会受到影响。 - core：处理 map-reduce
任务,保持 Hadoop 分布式文件系统 (HDFS),hbase的regionserver。 - task：处理
map-reduce 任务。这些纯粹是计算资源，并不存储数据。您可以向群集添加任务节点，提高性能速度，或移除不需要的任务节点。

> 注解：在task节点运行的任务会通过网络从core节点上获取数据，所以在某些情况下增加task节点并不能够缩短任务的运行时间。

### 4\. 检查输入数据

\- 请查看您的输入数据。它是否在键值之间均匀分配？
如果您的数据严重偏向一个或几个键值，那么可能将处理负载映射到少量节点，而其他节点则闲置。工作的不均衡分配可能会导致处理速度较慢。
-
不平衡数据集的示例是，依据按字母顺序排列的词运行群集，但有一个数据集仅包含以字母“a”开始的词。当工作被映射时，以“a”开始的节点处理值会过量，而以其他字母开始的节点处理词会处于闲置状态。
