{{indexmenu_n>2}}

# 常用操作

## 应用的Web接口

Hadoop 提供了基于 Web 的用户界面，可通过它查看您的 Hadoop 集群。Web 服务会在主节点上运行（Active
NameNode或者Active ResourceManager），绑定外网IP,开放对应应用防火墙端口后可以查看。

**web hdfs**

可以通过在浏览器地址栏中输入 http://外网ip:50070 来查看hdfs的基本信息。

**web yarn**

可以通过 http://外网IP:23188/cluster 查看yarn resource manager 信息。

> 注解：由于resource manager可以动态切换，当节点转换为StandBy节点时，webyarn服务无法切换到active节点。

**web hbase**

可以通过 http://外网IP:60010/master-status 查看hbase的基本信息。


## 查看日志

### 1.查看节点上的日志

可在产品界面选择服务管理，查看相应服务运行的节点位置，登陆对应节点后进入查看所启动服务的服务日志。

各服务日志所在位置如下所示：

| 位置                   | 说明            |
| -------------------- | ------------- |
| /var/log/hadoop-hdfs | hdfs服务相关日志    |
| /var/log/hadoop-yarn | yarn服务日志      |
| /var/log/hadoop-yarn | yarn服务日志      |
| /var/log/hbase       | hbase服务日志     |
| /var/log/hive        | hive服务日志      |
| /var/log/hue         | hue服务日志       |
| /var/log/zookeeper   | zookeeper服务日志 |

### 2.提交到yarn的任务日志

用户可以在web
yarn页面查看提交到yarn上的任务日志详情。由于任务日志界面需支持能访问集群各个节点，故可在UCloud云平台中的任意一台绑定外网IP的云主机或UHadoop的Master节点上，配置代理或者VPN，以便能够访问到集群中的每个节点。

#### a.配置vpn

可参考
[配置openvpn](http://cms.docs.ucloudadmin.com/analysis/uhadoop/operate/openvpn)文档配置vpn。

> 注解：配置完成后，可在访问端加上集群节点中最新的host文件即可（登陆集群中任意节点，查看/etc/hosts）。

#### b.配置Nginx反向代理

- 服务端配置

1. 安装Nginx

    yum install nginx -y

2. 修改配置

新建/etc/nginx/conf.d/proxy.conf 文件中添加如下配置

``` json
         server {
             listen   8889;
             client_body_timeout 60000;
             client_max_body_size 1024m;
             send_timeout   60000;
             client_header_buffer_size 16k;
             large_client_header_buffers 4 64k;
             proxy_headers_hash_bucket_size 1024;
             proxy_headers_hash_max_size 4096;
             proxy_read_timeout 60000;
             proxy_send_timeout 60000;
             location / {
             resolver 127.0.0.1;
             proxy_pass http://$http_host$request_uri;
             }
         } 
```

3.启动nginx服务

    service nginx restart

4.启动域名服务

``` 
service dnsmasq restart

```

> 集群节点发生变化时，需要重新启动这个服务。

- 访问端配置

1.在访问的网页端配置代理

2.配置hosts

需要在hosts中添加代理服务器的/etc/hosts文件中节点的host信息。

例如：

    10.19.43.21 uhadoop-wpmitd-master1
    10.19.20.134 uhadoop-wpmitd-core1
    10.19.133.58 uhadoop-wpmitd-master2

用户登录web yarn页面可通过任务id 来搜索对应任务，查看任务状态，并点击任务，获取任务日志，查看详情进行分析。

![](/images/uhadoop-38.png)

### 3.查看hdfs上的历史日志

yarn任务的日志在任务运行结束之后会上传到hdfs上，当日志文件过大无法通过web来查看时，可以通过将日志文件从hdfs上下载下来查看。

日志文件的目录是：\`hdfs://Ucluster/var/log/hadoop-yarn/apps/$SUBMITUSER/logs\`

> 注解：$SUBMITUSER是当前提交用户的名。

## 配置NFS挂载hdfs到本地

- 1.修改配置

修改master节点下下面两个配置。

core-site.xml

``` xml
    <property>
        <name>hadoop.proxyuser.nfsserver.groups</name>
        <value>*</value>
        <description>nfsserver有哪些group的权限</description>
    </property>
    <property>
        <name>hadoop.proxyuser.nfsserver.hosts</name>
        <value>hostname</value>
        <description>允许启动nfsserver的主机名</description>
    </property>
```

修改hdfs-site.xml

``` 
    <property>
        <name>nfs.dump.dir</name>
        <value>/tmp/.hdfs-nfs</value>
    </property>
    <property>
        <name>nfs.rtmax</name>
        <value>1048576</value>
        <description>单次读请求最大字节数</description>
    </property>
    <property>
        <name>nfs.wtmax</name>
        <value>65536</value>
        <description>单次写访问最大字节数 </description>
    </property>
    <property>
        <name>nfs.exports.allowed.hosts</name>
        <value>* rw</value>
        <description>配置挂载主机对文件的访问权限，例如”192.168.0.0/22 rw ; host.*\.example\.com ; host1.test.org ro;”</description>
    </property>
```

- 2.启动nfs， 在1中配置的允许启动nfs的主机上执行下面操作

``` 
    ${HADOOP_HOME}/sbin/hadoop-daemon.sh start portmap
    *一定是root用户才有权限绑定端口。
    ${HADOOP_HOME}/sbin/hadoop-daemon.sh start nfs3
    *一定是hadoop用户启动，才有所有文件的访问权限。
    showmount -e hostname
    Export list for hostname:
    / *
```

- 3.挂载

在nfs.exports.allowed.hosts允许的主机上执行

```
    mkdir -p /data/hdfsnfs
    mount -t nfs -o vers=3,proto=tcp,nolock,noacl hostname:/ /data/hdfsnfs
```
