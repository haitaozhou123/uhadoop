{{indexmenu_n>6}}

# Hue开发指南

Hue是面向 Hadoop 的开源用户界面，可以让您更轻松地运行和开发 Hive 查询、管理 HDFS 中的文件、运行和开发 Pig
脚本以及管理表。服务默认已经启动，用户只需要配置外网IP，在防火墙中配置开放端口就可以了。如果没有安装hue，可以在集群的"服务管理"页面开启Hue。

访问地址: <http://外网ip:8888>

默认用户名/密码：hadoop/hadoop或者hue/hue， 用户登陆后可以自行更改。

如果需要为Hue配置ldap或者sentry服务，请安装以下步骤安装：

## 1. 为Hue配置ldap

下载[安装包](http://uhadoop-new.ufile.ucloud.com.cn/hue/openldap-2.4.44.tar.gz)，并拷贝到集群的master1节点上，并以root用户执行以下命令：

```
tar zxf openldap-2.4.44.tar.gz -C /home/hadoop/.versions/
sh /home/hadoop/.versions/openldap-2.4.44/dependentpackages/install-openldap.sh
```

安装完成后，会自动启动openldap服务。如果需要启动、停止或重启服务，请以root用户在master1节点上执行以下命令：

```
service ldap start|stop|restart
```

## 2. 为Hue配置Sentry

下载[安装包](http://uhadoop-new.ufile.ucloud.com.cn/hue/sentry-1.7.0.tar.gz)，分发到集群所有安装hive服务的节点上，以root用户执行以下命令：

```
tar zxf sentry-1.7.0.tar.gz -C /home/hadoop/.versions/
sh /home/hadoop/.versions/sentry-1.7.0/dependentpackages/install-sentry.sh
```

安装完成后，会自动启动sentry服务。如果需要启动、停止或重启服务，请以root用户在master1节点上执行以下命令：

```
service sentry start|stop|restart
```

> 可以通过集群的“服务管理-\>Hive”页面查看集群中有哪些角色安装了Hive服务：

![hadoop-hue-find-hive.png](/images/developer/hadoop-hue-find-hive.png)

## 3. 配置工作流

启用Hue的工作流功能，需要在集群上安装。可以在集群的"服务管理"页面启用Oozie。

### 3.1 创建新的工作流

在浏览器中依次点击【Workflows】-\>【Editors】-\>【Wokflows】，进入Workflow
Editor。然后点击页面右侧的【Create】按钮。

![3.1-create-workflow.png](/images/developer/3.1-create-workflow.png)

### 3.2 创建Spark任务

从action中拖动spark的标签到工作流中。点击右上角的【Settings】按钮，出现一个弹出窗口，我们可以在这里为Workflow设置变量名，并设置Workspace。

现在，添加input和output这两个变量，并将Workspace设置为HDFS的目录/user/admin/workspaces，如下：

![](/images/developer/3.2-spark.png)

设置好后，点击右上角的叉关闭这个弹出窗口，回到之前的页面（现在还需要自己将Spark Job所需的Jar包放入相应的HDFS目录中）。

我们将【Spark】图标拖到相应的位置，然后继续进行设置。我们设置了Jar包的路径
lib/oozie-examples.jar。还要设置main class。

> 注意这是HDFS路径，且是相对于Workspace的路径（所以实际路径就是/user/admin/workspaces/lib/oozie-examples.jar）

由于这个main class的作用的是复制HDFS的文件，它在运行时需要给main方法传入两个参数，分别是src path和dest
path，所以这里我们继续添加参数，如下图：

![](/images/developer/3.2-spark-2.png)

这里的${input}和${output}就是之前我们在【settings】中设置的变量名。

好了，一切都设置好了之后，点击右上角的【Save】按钮。

### 3.3 创建Hive任务

uhadoop上使用的是hive-server2，所以这里选择hive-server2标签拖动到action中。

然后，将准备好的sql脚本上传到hdfs上，并配置提交任务的参数

> 如果定义了ufd可以通过文件的参数来指定

最后，保存工作流。

![](/images/developer/3.3-hive1.png)

### 3.4 创建Sqoop任务

选择sqoop1这个标签拖动到action中。然后，添加需要执行的Sqoop命令。

> 注意：
>
> 1.密码不能加额外的引号，hue会把-p 参数后面的所有内容都解释为密码；
>
> 2.Sqoop 要把任务分发到所有的集群节点，要保证集群所有节点对目标数据库的读写权限。

最后，保存工作流。

![](/images/developer/3.4-sqoop.png)

## 4. Hue页面权限控制

- 点击【管理用户】-\>【组】-\> 选择要修改的组名称，设置相应权限并保存

![800](/images/developer/hadoop-hue-permissions.jpg)
