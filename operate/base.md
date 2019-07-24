{{indexmenu_n>20}}

# 基本操作

## 集群管理

### 1、进入集群管理页面

通过UHadoop集群列表页面进入集群管理页面：

![](/images/operate/集群管理入口.png)

### 2、获取当前节点配置信息

![](/images/operate/节点概览.png)

本例中，Master 节点数量 2，机型为 C1-large；Core 节点数量为 2，机型为 F1-large。

### 3、添加节点

点击添加节点后弹窗如下：

![](/images/operate/添加节点.png)

输入节点初始密码，点击确定，等待节点创建。

### 4、删除节点

选中要删除的 Task 节点，点击删除。

目前暂不支持用户主动对主节点（master节点）、数据节点（core节点）的删除，如有需求，请提工单。

### 5、集群登录

a) 通过控制台登录。如果节点机型是物理机，那么由于不同服务器厂商标准不同，暂不能通过控制台登录集群。

b) 绑定外网eip，本地可通过外网ssh连接登录。目前仅master节点支持绑定。Eip使用详情请见 EIP说明文档 本例中可通过”ssh
root@106.75.135.10 -p22”进行登录。

c) 通过云主机（uhost）内网ssh进行登录。本例中可在云主机上通过”ssh root@10.13.186.23 -p22”进行登录。
登录密码为集群创建时设置的密码

### 6、节点密码重置

可在【节点管理】页面，对节点密码重置。

### 7、节点重启

目前节点的重启是通过【关机】【开机】实现的。

如有重启需求，请在【节点管理】页面中，先对节点【关机】，再对节点做【开机】。

## 服务管理

### 1、进入集群管理页面

通过右侧弹出框点击进入

![](/images/uhadoop-24.png)

### 2、应用开启

若创建时未选择部署该服务，可在控制台点击开启并选择所需版本。

![](/images/uhadoop-25.png)

![](/images/uhadoop-26.png)

### 3、服务关闭、重启

页面可进行相应服务关闭，以及服务重启

### 4、参数调整

4.1 点击参数配置可进行参数调整

![](/images/uhadoop-28.png)

4.2 在界面修改已有参数，修改后确认重启生效

![](/images/uhadoop-29.png)

![](/images/uhadoop-30.png)

4.3 新参数的配置添加

![](/images/uhadoop-31.png)

![](/images/uhadoop-32.png)

**参数名称：** 您需要添加的参数名称。

**使用范围：**
该参数所要应用的范围。目前支持cluster（整个集群）、master（所有master节点）、core（所有core节点）、task（所有task节点）。

**文件：** 参数应存在位置。

**参数值：** 您设置的参数值。

4.4 参数删除

一些非必要参数可在控制台进行删除操作。

![](/images/canshuxinzeng01.png)

### 5、配置队列

5.1 选择 应用-hadoop-队列配置 进行队列配置

![](/images/duilie01.png)

5.2 选择 创建队列

![](/images/duilie02.png)

5.3 输入队列名称，点击确定

![](/images/duilie03.png)

5.4 修改队列配置

![](/images/duilie04.png)

5.5 队列历史配置回退

若用户对队列配置进行修改后想要回退到历史配置，可使用此功能

![](/images/duiliexinzeng01.png)

![](/images/canshuxinzeng02.png)

更多关于队列配置内容可参考[队列配置](https://docs.ucloud.cn/analysis/uhadoop/operate/config)及[Apache官网介绍](https://hadoop.apache.org/docs/r2.7.1/hadoop-yarn/hadoop-yarn-site/FairScheduler.html)。

## 告警与监控

### 1、告警模板设置

监控视图中修改相应集群和集群节点的告警模板。

![](/images/uhadoop-34.png)

![](/images/uhadoop-35.png)

也可到监控页面进行对应产品的告警设置与告警模板修改。

![](/images/uhadoop-36.png)

### 2、监控数据查看

用户可于产品界面右侧弹框中查看集群监控数据，也可进入监控视图中进行详细查看集群及各节点监控数据信息。

![](/images/uhadoop-37.png)

===== 外网访问配置 =====

绑定外网IP:

目前只支持对master节点配置外网IP，可以配置防火墙限制访问IP。

1.申请弹性IP

2.为Master节点绑定IP

3.设置防火墙策略

请参考：<https://doc.ucloud.cn/network/firewall/firewall>

4.为绑定外网IP的主机配置防火墙

![](/images/operate/绑定防火墙.png)

## 数据均衡

选中数集群，在右侧的管理界面选中均衡数据按钮，均衡数据的过程会对集群的性能产生一定影响，持续时间根据集群内数据量，以及均衡程度而不同。

![](/images/operate/55.png)

## Yarn Application跟踪

控制台【Yarn Application跟踪】页面的信息来自集群中的 Yarn 服务。可查最近15天内提交的 Application 情况。

对于 MapReduce、Spark 类型 Application，可查子任务日志、以及 Application Master 日志详情。

暂不支持查看其他类型的 Application 详情。

![](/images/operate/yarn_application跟踪.png)

Yarn Application 状态说明：

| 状态         | 说明                                                          |
| ---------- | ----------------------------------------------------------- |
| NEW        | Application 请求到 ID 和可用资源后，状态为 NEW                           |
| NEW SAVING | Resource Manager 接收到 Application 的详细资源请求后，状态为 NEW SAVING    |
| SUBMITTED  | Resource Manager 存储 Job 信息后，状态为 SUBMITTED                   |
| ACCEPTED   | Resource Manager 的 Scheduler 做资源、用户权限等验证，验证通过后，状态为 ACCEPTED |
| RUNNING    | 资源就绪，开始执行 Application                                       |
| FINISHED   | Job 如期执行完毕，状态为 FINISHED                                     |
| FAILED     | 执行失败                                                        |
| KILLED     | 由用户终止了 Job                                                  |
