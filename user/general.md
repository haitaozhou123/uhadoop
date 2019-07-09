{{indexmenu_n>2}}

# 集群使用

## 集群创建好后，我该怎么使用？

\- 如果已购买同一可用区下带有外网IP的UHost，可登陆UHost，再通过ssh方式连接集群任一节点的内网IP。

详见：[faq\#我的主机只有内网IP，可以通过哪些方式访问这台机器呢？](/network/unet/faq#我的主机只有内网IP，可以通过哪些方式访问这台机器呢？)

\- 将集群的master节点绑定外网IP，ssh连接此外网IP。集群节点上可直接使用hdfs、hive、hbase、spark相关命令。

## 提交的任务跑失败了，我需要怎样查看任务日志？

\- 通过浏览器连接绑定在master1/master2上的外网IP的23188端口查看任务详情（需将所绑定的外网防火墙开放23188端口）

\- 通过Hue页面查看任务详情

\-
任务的日志默认存储在hdfs的/var/log/hadoop-yarn/apps/\[submintuser\]/logs（submituser为提交任务的用户名），可通过web方式查看hdfs文件，或通过hdfs命令将相应Job日志下载到本地查看

## 客户端提交任务时，提示UnkownHostException，如何处理？

集群节点间默认使用Hostname的方式通信，需要将master1的/etc/hosts中关于uhadoop节点的host拷贝到其它节点本地的/etc/hosts中

## 我收到了磁盘告警通知，磁盘使用率大于95%，该怎么处理？

首先确认是系统盘还是数据盘告警。系统盘告警的请在/目录下执行 du --max-depth=1
-h,数据盘告警请在/data目录下执行，可深入一层层查看具体哪个文件占用了空间，做下清除即可

## HDFS满了，我收到了HDFS使用告警通知，HDFS使用率大于80%，该怎么处理？

首先登录master1节点，切换到hadoop用户，执行 hadoop fs -du -h / 查看哪个目录占用较大。

如果是/var目录，可继续执行hadoop fs -du -h /var/log 查看；若查看是 /var/log/hadoop-yarn/,
可以根据自己的需求在控制台修改这两个参数，需要重启集群

``` 
  yarn.log-aggregation.retain-seconds    （保存时长，建议值：2592000, 即30天）
  yarn.log-aggregation.retain-check-interval-seconds （检查周期：86400，即1天）
```

如果是/var/log/spark目录，一般是spark任务日志堆积，暂未能自动清理，可通过以下方式解决

在/etc/cron.daily/下创建clear-spark-logs.sh文件，并给设置权限 chmod 777
/etc/cron.daily/clear-spark-logs.sh

第一次可通过执行 sh /etc/cron.daily/clear-spark-logs.sh手动清理，后面每天会自动

```
#!/bin/bash
# 删除过期数据（默认30天）

days=30
old_file_list=$(hadoop fs -ls /var/log/spark/ | awk 'BEGIN{ days_ago=strftime("%F", systime()-"'$days'"*24*3600) }{ split($8,arr,"/"); if(arr[7]<days_ago){printf "%s\n", $8} }')
arr=(${old_file_list// / })
for file in ${arr[@]}
do
    su -s /bin/bash hadoop -c "hadoop fs -rm -r $file"
done
```

## 我收到了节点内存告警通知，内存使用率大于95%，该怎么处理？

首先确认是Master节点，还是Core节点告警。

1\.
若是Master节点，可以登录到该节点，通过ps命令找到内存占用较高的进程，如果确认该进程为自己业务启动的且不需要的，可以直接kill掉。如果是hadoop集群相关进程，建议升级Master节点来彻底解决。

升级Master可参考[general\#集群单个节点配置不够，如CPU，MEM或者磁盘，需要怎么升级](/analysis/uhadoop/user/general#集群单个节点配置不够，如CPU，MEM或者磁盘，需要怎么升级)

Master节点可选机型列表： <https://docs.ucloud.cn/analysis/uhadoop/price>

2\.
若是Core节点，如果未自行安装服务，一般为DataNode、HBase-Regionserver和Yarn使用内存较多。可通过下列步骤解决

\-- 若不使用HBase，可通过UCloud管理控制台UHadoop下的“集群服务管理”页面关闭不使用服务；

\--
若关闭后未恢复，可通过修改Hadoop的yarn.nodemanager.resource.memory-mb参数来降低节点分配给Yarn的内存（可通过UCloud管理控制台UHadoop下的Hadoop
Tab下的“参数配置”功能修改）；

\-- 若业务上需求更多内存资源，可适当添加Core/Task节点

\--
若是单节点CPU、MEM不足，可参考[general\#集群单个节点配置不够，如CPU，MEM或者磁盘，需要怎么升级](/analysis/uhadoop/user/general#集群单个节点配置不够，如CPU，MEM或者磁盘，需要怎么升级)

## 集群单个节点配置不够，如CPU，MEM或者磁盘，需要怎么升级？

升级Master：请通过工单形式，需提供（集群ID，原Master机型，待升级Master机型，可以执行操作时间），我们目前支持非标升级，且须同时升级2个Master，升级过程中Master节点需要重启，停止服务大概1分钟之内，注意需要保证集群不欠费，且账户内有足够余额满足升级费用。

Master节点可选机型列表： <https://docs.ucloud.cn/analysis/uhadoop/price>

若为Core节点，默认不支持单个节点纵向升级，如果需要扩充资源，可以横向添加节点；如果遇到内存或其它单个节点的资源瓶颈，可以联系我们提供后台升级，升级是通过用高配节点替换低配节点完成，期间需要迁移大量数据，可能影响到集群业务，数据量大集群不建议此操作。

Core节点可选机型列表： <https://docs.ucloud.cn/analysis/uhadoop/price>

## 我想关闭某台机器上的某个服务，或者关闭服务开机启动，怎么办？

\- 停止服务：

通常情况下，可使用 “service 服务名称 stop”
命令停止正在运行的服务进程。如服务无法停止，可在无任务时使用kill命令进行强制停止服务进程。

\- 关闭服务自动拉起：

将/etc/default/static\_conf.json里startmonitor的值修改为0即可。

\- 关闭服务并关闭开机自启：

执行以上动作，并备份/etc/default/services。备份后，在原文件中删除掉相关服务字段内容即可。
