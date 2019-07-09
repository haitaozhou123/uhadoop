{{indexmenu_n>20}}

# 操作指南

## 1、节点管理

**扩容节点:**

在集群的管理界面点击添加节点。

![](/images/45.jpg)

输入节点初始密码。

![](/images/46.jpg)

等待创建。

**删除节点:**

在集群的管理节点界面，选中要删除的节点，然后点击删除。

![](/images/47.jpg)

> 注解：目前暂不支持用户主动对数据节点的删除，如有需求，请提工单。

## 2、监控视图

在集群的管理界面可以查看到hadoop集群的运行状态统计信息，以及各个节点的物理信息。

集群运行状态监控视图：

![](/images/48.jpg)

各节点运行状态监控视图：

![](/images/49.jpg)

## 3、外网访问配置

绑定外网IP:

目前只支持对master节点配置外网IP，可以配置防火墙限制访问IP。

1.申请弹性IP

![](/images/uhadoop-guide1.png)

2.为Master节点绑定IP

![](/images/uhadoop-guide2.png)

3.设置防火墙策略

请参考:

[防火墙使用手册](/network/unet/firewall)

![](/images/52.jpg)

4.为绑定外网IP的主机配置防火墙

点击修改外网防火墙

![](/images/53.jpg)

选择步骤3中配置的防火墙策略

![](/images/54.jpg)

## 4、数据均衡

选中数集群，在右侧的管理界面选中均衡数据按钮，均衡数据的过程会对集群的性能产生一定影响，持续时间根据集群内数据量，以及均衡程度而不同。

![](/images/55.jpg)

## 5、JOB展示

展示运行的job信息

![](/images/56.jpg)

设置默认显示列

![](/images/57.jpg)

## 6、服务管理

选中集群，点击右侧的服务管理界面可以进入到集群服务管理的界面

![](/images/58.jpg)

点击右侧的参数配置可以修对应服务的参数，参数设置后，不会立即生效，需要重启集群所有相关服务才能生效。

![](/images/59.jpg)

## 7、应用的Web接口

Hadoop 提供了基于 Web 的用户界面，可通过它查看您的 Hadoop 集群。Web 服务会在主节点上运行（Active
NameNode或者Active ResourceManager），绑定外网IP,开放对应应用防火墙端口后可以查看。

**web hdfs**

可以通过在浏览器地址栏中输入 <http://外网ip:50070> 来查看hdfs的基本信息。

**web yarn**

可以通过 <http://外网IP:23188/cluster> 查看yarn resource manager 信息。

> 注解：由于resource manager可以动态切换，当节点转换为StandBy节点时，webyarn服务无法切换到active节点。

**web hbase**

可以通过 <http://外网IP:60010/master-status> 查看hbase的基本信息。
