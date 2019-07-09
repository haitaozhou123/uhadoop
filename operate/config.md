{{indexmenu_n>3}}

# 队列管理

## Hadoop配置队列

\- 用户可于 控制台-服务管理
进行队列配置，具体可参考[基本操作](https://docs.ucloud.cn/analysis/uhadoop/operate/base)。

由于Fair Scheduler比Capacity Scheduler 支持的功能丰富，这里只介绍前者。

\- 修改/home/hadoop/conf/yarn-site.xml

增加配置：

``` xml
    <property>
      <name>yarn.scheduler.fair.allocation.file</name>
      <value>fair-scheduler.xml</value>
      <description>默认会从HADOOOP_CONF_DIR路径下寻找</description>
    </property>
    <property>
      <name>yarn.scheduler.fair.allow-undeclared-pools</name>
      <value>false</value>
      <description>禁止创建为未指定队列名的任务创建新的队列，未指定队列的任务会使用默认队列资源</description>
    </property>
```

\- 配置队列配额

cat fair-scheduler.xml

``` xml
    <?xml version="1.0"?>
    <allocations>
            <queue name="root">             
                    <queue name="testqueue">
                    <minResources>1024mb,1vcores</minResources>
                    <maxResources>2048mb,2vcores</maxResources>
                    <maxRunningApps>100</maxRunningApps>
                    <schedulingPolicy>fair</schedulingPolicy>
                    <weight>3.0</weight>
                    <aclSubmitApps>* </aclSubmitApps>
                    </queue>
            </queue>
    </allocations>
```

\- 队列属性说明：

    1.minResources ：最少资源保证量，设置格式为“X mb, Y vcores”，当一个队列的最少资源保证量未满足时，它将优先于其他同级队列获得资源，对于不同的调度策略（后面会详细介绍），最少资源保证量的含义不同，对于fair策略，则只考虑内存资源，即如果一个队列使用的内存资源超过了它的最少资源量，则认为它已得到了满足；对于drf策略，则考虑主资源使用的资源量，即如果一个队列的主资源量超过它的最少资源量，则认为它已得到了满足。

    2.maxResources：最多可以使用的资源量，fair scheduler会保证每个队列使用的资源量不会超过该队列的最多可使用资源量。

    3.maxRunningApps：最多同时运行的应用程序数目。通过限制该数目，可防止超量Map Task同时运行时产生的中间输出结果撑爆磁盘。

    4.minSharePreemptionTimeout：最小共享量抢占时间。如果一个资源池在该时间内使用的资源量一直低于最小资源量，则开始抢占资源。

    5.schedulingMode/schedulingPolicy：队列采用的调度模式，可以是fifo、fair或者drf。

    6.aclSubmitApps：可向队列中提交应用程序的Linux用户或用户组列表，默认情况下为“*”，表示任何用户均可以向该队列提交应用程序。需要注意的是，该属性具有继承性，即子队列的列表会继承父队列的列表。配置该属性时，用户之间或用户组之间用“，”分割，用户和用户组之间用空格分割，比如“user1, user2 group1,group2”。

    7.aclAdministerApps：该队列的管理员列表。一个队列的管理员可管理该队列中的资源和应用程序，比如可杀死任意应用程序。

    8.weight主要用在资源共享之时，weight越大，拿到的资源越多。比如一个pool中有20GB内存用不了，这时候可以共享给其他pool，其他每个pool拿多少，就是由权重决定的。

    9.maxAMShare： application masters使用的内存比例限制取值范围是[0.0f, 1.0f], 默认值是0.5f， -1.0f 表示不进行检查。

    10.fairSharePreemptionTimeout：公平共享量抢占时间。如果一个资源池在该时间内使用资源量一直低于公平共享量的fairSharePreemptionThreshold * 公平共享量，则开始抢占资源。

    11.fairSharePreemptionThreshold： 配合上面参数使用的。

\- 也可通过以下参数设置上面部分属性的默认值

``` xml
    defaultFairSharePreemptionTimeout
    userMaxAppsDefault
    defaultMinSharePreemptionTimeout
    defaultFairSharePreemptionThreshold
    queueMaxAppsDefault
    queueMaxAMShareDefault
    defaultQueueSchedulingPolicy
    queuePlacementPolicy
```

\- 用户属性：

maxRunningApps: 目前只能限制用户提交任务数量

\- 应用队列配置

修改fair-scheduler.xml文件，后推送到master节点上。然后在master1上以hadoop用户执行

```
yarn rmadmin -refreshQueues
```

> 注解：这个命令只能增加fair-scheduler.xml中的配置，
> 不能删除掉原有的配置队列，如果要删除原来的配置要重启ressourcemanager

更详细的内容请参考[官网说明](https://hadoop.apache.org/docs/r2.7.1/hadoop-yarn/hadoop-yarn-site/FairScheduler.html)
