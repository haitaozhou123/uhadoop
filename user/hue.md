{{indexmenu_n>7}}

# Hue

## Hue中执行“hive sql”时提示Fetching results ran into the following error(s): Couldn't find log associated with operation handle: OperationHandler

首先确认master2节点上的 “/tmp/hadoop/operation\_logs/” 目录是否存在，是否有过删除或修改了权限；
此地址是存hive客户端的log地址，上述操作会导致hiveserver2写log失败，可通过在master2上执行
“service hive-server2 restart” 重启hive-server2服务进行解决。

## Hue密码忘记了怎么办？

Hue默认密码是hadoop/hadoop或者hue/hue，一般第一次登录会强制设置密码。

Hue页面默认不支持密码重置，可以通过到master1节点登录hue所在数据库，修改数据库中加密后字符串

这里举例将hadoop用户密码变更为“hadoop”

a）登录集群master1节点

b）shell中执行

``` 
  mysql
```

c）执行

``` 
  update hue.auth_user set password="pbkdf2_sha256$12000$FfWacKgK93h2$J4uP5UU5aR/JXvNydvPxFlquyrbWWF2FmtXkrBCOUUA=" where username='hadoop';
```

（上面sql中password是“hadoop”加密后的字符串）

## Hue怎么连接Spark；Hue首页显示“The app won't work without a running Livy Spark Server”，怎么处理？

Hue3.8.1有独立的Spark模块，而在Hue3.10.0中集成到notebook模块里面

Hue上执行Spark任务依赖livyserver，默认已配置，但需要手动开启

开启方法：

登录master1节点执行

```
cp /home/hadoop/hue/dependentpackages/livyserver /etc/init.d/
chmod u+x /etc/init.d/livyserver
echo "LivyServer##livyserver" >> /etc/default/process
sed -i "s/$/#LivyServer#livy/g" /etc/default/services   
```

## 在Hue页面提交任务后，master节点上有多个LivyServer进程不退出怎么办？

LivyServer在用spark集群模式的时候，提交任务后新开窗口不会自行终止。目前只能手动kill。
可以将下面的脚本放到/etc/cron.hourly下面并赋予可执行权限，自动清理一小时前启动的spark提交进程。

```
#!/bin/bash
ppid=`ps axu | grep livy.server.port | grep -v "grep" | awk '{ print  $2}'`
pids=`ps -elf | grep -v "grep" | grep $ppid | awk '{print \$4}'`
NOW=`date +"%s"`
echo now $NOW
for i in $pids
do
if [ $ppid -ne $i ]
then
        echo $i
  JIFFIES=`cat /proc/${i}/stat | cut -d" " -f22`
  UPTIME=`grep btime /proc/stat | cut -d" " -f2`
  START_SEC=$(( $UPTIME + $JIFFIES / $(getconf CLK_TCK)))
  LAST=$(( $NOW - $START_SEC))
  if [ $LAST -gt 3600 ]
  then
        echo $i last $LAST
        kill -9 $i
  fi
fi
done
```
