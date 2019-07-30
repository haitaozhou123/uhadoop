{{indexmenu_n>35}}

# 端口配置

| 配置名                                               | UHadoop默认配置    |
| ------------------------------------------------- | -------------- |
| yarn.resourcemanager.zk-address                   | localhost:2181 |
| yarn.resourcemanager.address.rm1                  | master1:23140  |
| yarn.resourcemanager.address.rm2                  | master2:23140  |
| yarn.resourcemanager.scheduler.address.rm1        | master1:23130  |
| yarn.resourcemanager.scheduler.address.rm2        | master2:23130  |
| yarn.resourcemanager.webapp.https.address.rm1     | master1:23189  |
| yarn.resourcemanager.webapp.https.address.rm2     | master2:23189  |
| yarn.resourcemanager.webapp.address.rm1           | master1:23188  |
| yarn.resourcemanager.webapp.address.rm2           | master2:23188  |
| yarn.resourcemanager.admin.address.rm1            | master1:23141  |
| yarn.resourcemanager.admin.address.rm2            | master2:23141  |
| yarn.resourcemanager.resource-tracker.address.rm1 | master1:23125  |
| yarn.resourcemanager.resource-tracker.address.rm2 | master2:23125  |
| yarn.nodemanager.localizer.address                | 0.0.0.0:23344  |
| NM Webapp address                                 | 0.0.0.0:23999  |
| zeppelin.server.port                              | master1:29090  |
|presto coordinator http-server.http.port           | master1:28080  |
| Presto worker http-server.http.por( core,task节点）| 28080          |
| mapreduce.shuffle.port                            | 23080          |
| mapreduce.jobhistory.address                      | 10020          |
| dfs.datanode.address                              | 50010          |
| dfs.datanode.http.address                         | 50075          |
| dfs.datanode.https.address                        | 50475          |
| dfs.datanode.ipc.address                          | 50020          |
| fs.defaultFS                                      | 8020           |
| dfs.namenode.servicerpc-address                   | 8022           |
| dfs.namenode.http-address                         | 50070          |
| dfs.namenode.https-address                        | 50470          |
| dfs.namenode.secondary.http-address               | 50090          |
| dfs.secondary.https.address                       | 50495          |
| dfs.namenode.shared.edits.dir                     | 8485           |
| dfs.journalnode.http-address                      | 8480           |
| dfs.journalnode.https-address                     | 8481           |
| DFSZKFailoverController                           | 8019           |
| hbase.master.port                                 | 60000          |
| hbase.master.info.port                            | 60010          |
| hbase.regionserver.port                           | 60020          |
| hbase.zookeeper.property.clientPort               | 2181           |
| hbase.zookeeper.peerport                          | 2888           |
| hbase.zookeeper.leaderport                        | 3888           |
| hbase.rest.port                                   | 60050          |
| hive.server2.thrift.port                          | 10000          |
| Hive metastore                                    | 9083           |
| Zookeeper ClientPort                              | 2181           |
| Zookeeper Peer                                    | 2888、3888      |
