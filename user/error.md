{{indexmenu_n>9}}

# 常见任务ERROR

## java.lang.OutOfMemoryError: Java heap space

原因：单个任务所分配mem较低，或者任务数据量教导，导致任务OOM

解决方法：

a. Executor端 OOM：提交任务时，尝试增大任务参数--executor-memory

b. Driver端OOM：a. 尝试增大任务参数--driver-memory;
b.降低任务并行度，修改/home/hadoop/spark/conf/spark-defaults.conf，添加spark.default.parallelism
40

## java.lang.ClassNotFoundException

原因：提交任务时缺少相关jar包，具体可根据java.lang.ClassNotFoundException后面提示分析缺少哪个包

解决方法：

a. spark-submit提交任务时候指定--jars，多个包之间用逗号分开

b.
包较多时可以降包放入一个目录下，并在/home/hadoop/spark/conf/spark-defaults.conf中指定spark.executor.extraClassPath或者spark.driver.extraClassPath指向此目录

## User root cannot submit applications to queue root.root

原因：任务提交者没有此队列或者默认队列提交权限

解决方法：提交任务时增加--queue指定有权限队列
