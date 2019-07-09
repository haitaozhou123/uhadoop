{{indexmenu_n>5}}

# HBase

## HBase某一个表数据无法写入，也无法读取，从WebUI界面查看到有多个Region状态为region in transaction是因为？

这是由于Region在分裂或者迁移中卡住了，可以找到相应的RegionServer节点，通过 “service
hbase-regionserver restart” 将服务进行重启。

若重启后还存在此状态，可依次重启master1，master2的Hmaster服务，命令为

```
service hbase-master restart
```

## 读取、写入数据时，为什么找不到region？

通过row key查找一条数据的时候，抛出NotServingRegionException异常。可以尝试通过以下几个步骤来解决：

``` 
  - 通过web界面查看是有rit状态的region
  - 通过hbase hbck 命令查看异常状态
  - 使用hbase hbck命令修复
```

> 举例：

\>

> ERROR:Region{meta=\>inventoryIBCF,304801,1465002356884.93d89fdd4546546f0d52041773b495bc.,hdfs=\>hdfs://Ucluster/hbase/data/default/inventoryIBCF/93d89fdd4546546f0d52041773b495bc,
> deployed =\> , repli caId =\> 0 } not deployed on any region server.

\>

> 可以通过hbase hbck -fixAssignments命令来修复。

更多修复命令可以通过 hbase hbck --help来查询，找到对应错误的修复语句执行即可。

如果频繁遇到这个问题可能是 hbase 的参数或者其他方面设置的不合理，需要调整一下。
