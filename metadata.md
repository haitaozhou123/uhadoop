{{indexmenu_n>15}}

# 元数据管理

## 介绍

UHadoop 支持将 Hive-Metastore 的数据库独立于 Hadoop 集群部署，也支持多个集群访问同一个 Hive
元数据库，可在控制台对其做管理。

## 产品架构

Hive 元数据存储于 UCloud UDB MySQL 中。

![](/images/架构.png)

## 元数据管理

  - 创建集群时可在控制台开启元数据独立管理。

![](/images/控制台开启.png)

同一项目下，所有集群的 Hive 元数据均存储在同一数据库中。

若首次开启该功能，则 UHadoop 会为用户创建一个新的 UCloud UDB MySQL 。

若项目中已开启过元数据独立管理，则新集群开启该功能时，不再创建新的 UDB，&#8;而是将新集群的 Hive 元数据存储于已有的 UDB 中。

  - 设置完成后，可在【元数据管理】页面查看数据库状态，UDB 名称固定为【uhadoop-metastore】。

![](/images/元数据状态.png)

数据库的监控信息可以在 UDB 控制台查看。为了保证 UHadoop 集群中任务的正常执行，请谨慎对此 UDB 做操作。

![](/images/udb页面.png)

  - 若集群开启了【元数据独立管理】，可在集群详情中查看到关联的 UDB MySQL 资源ID。

![](/images/集群udb.png)
