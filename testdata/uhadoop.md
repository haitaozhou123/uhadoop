{{indexmenu_n>0}}

# UHadoop性能测试

## 1.测试集群配置

分别选择SAS的B2-large、B2-xlarge和SATA的D1-large、D1-xlarge以及普通虚拟机作为core节点，建立A-E共5个集群。Master节点统一采用C1-4xlarge。具体配置见表1.1。其中，由于E集群性能太差，只测试了HBase和磁盘读写性能。

- 表1.1 集群配置

| 集群名称 | Master节点配置 | Core节点配置        | Core节点数量 | 可用区 |
| ---- | ---------- | --------------- | -------- | --- |
| A    | 16核,32G    | 2核,6G,SATA 4T   | 6        | 北京D |
| B    | 16核,32G    | 4核,12G,SATA 8T  | 6        | 北京B |
| C    | 16核,32G    | 2核,6G,SAS 600G  | 6        | 北京B |
| D    | 16核,32G    | 4核,12G,SAS 1.2T | 6        | 北京C |
| E    | 16核,32G    | 2核,6G,vh 600G   | 6        | 北京B |

-----

## 2.Yarn集群wordcount测试

使用BigDataBench的数据源，从10GB开始，每次增加10GB，直至200GB。生成数据源代码如下：

- yarn-wordcount数据源生成代码

```
wget http://prof.ict.ac.cn/bdb_uploads/bdb_3_1/packages/BigDataBench_V3.2.1_Hadoop.tar.gz
tar zxf BigDataBench_V3.2.1_Hadoop.tar.gz
cd BigDataBench_V3.2.1_Hadoop_Hive/MicroBenchmarks
sh genData_MicroBenchmarks.sh
200
```

依次从数据源取数据，进行wordcount测试，并打印结果，代码如下：

- yarn-wordcount-run.sh

```
#!/bin/bash
source /home/hadoop/.bashrc

WORK_DIR=`pwd`
echo "WORK_DIR=$WORK_DIR data should be put in $WORK_DIR/data-MicroBenchmarks/in"

${HADOOP_HOME}/bin/hadoop fs -mkdir -p /MicroBenchmarks/in/
${HADOOP_HOME}/bin/hadoop fs -mv /MicroBenchmarks/in/* ${WORK_DIR}/data-MicroBenchmarks/in/
for i in {1..20}
do
    #move 10GB file from ${WORK_DIR}/data-MicroBenchmarks/in/ to /MicroBenchmarks/in/
    for f in `${HADOOP_HOME}/bin/hadoop fs -ls ${WORK_DIR}/data-MicroBenchmarks/in/ | grep lda_wiki1w | awk '{print $NF}' | head -n 20`
    do
        ${HADOOP_HOME}/bin/hadoop fs -mv $f /MicroBenchmarks/in/
    done
    ${HADOOP_HOME}/bin/hadoop fs -rmr ${WORK_DIR}/data-MicroBenchmarks/out/wordcount
    echo -n "file count = "
    ${HADOOP_HOME}/bin/hadoop fs -ls /MicroBenchmarks/in/ | grep lda_wiki1w | wc -l
    time ${HADOOP_HOME}/bin/hadoop jar  ${HADOOP_HOME}/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar  wordcount  /MicroBenchmarks/in/ ${WORK_DIR}/data-MicroBenchmarks/out/wordcount
done

```

对四个集群的测试结果如下：

- 表2.1 yarn集群wordcount测试结果

|      | 耗时(s) |      |      |      | 处理速度(MB/s) |       |       |       |
| ---- | ----- | ---- | ---- | ---- | ---------- | ----- | ----- | ----- |
|      | B2    | B2-x | D1   | D1-x | B2         | B2-x  | D1    | D1-x  |
| 10G  | 332   | 176  | 319  | 177  | 30.84      | 58.18 | 32.10 | 57.85 |
| 20G  | 624   | 319  | 618  | 322  | 32.82      | 64.20 | 33.14 | 63.60 |
| 30G  | 990   | 455  | 922  | 462  | 31.03      | 67.52 | 33.32 | 66.49 |
| 40G  | 1267  | 599  | 1210 | 613  | 32.33      | 68.38 | 33.85 | 66.82 |
| 50G  | 1536  | 752  | 1518 | 757  | 33.33      | 68.09 | 33.73 | 67.64 |
| 60G  | 1834  | 903  | 1840 | 925  | 33.50      | 68.04 | 33.39 | 66.42 |
| 70G  | 2113  | 1041 | 2111 | 1064 | 33.92      | 68.86 | 33.96 | 67.37 |
| 80G  | 2504  | 1184 | 2411 | 1215 | 32.72      | 69.19 | 33.98 | 67.42 |
| 90G  | 2785  | 1334 | 2746 | 1360 | 33.09      | 69.09 | 33.56 | 67.76 |
| 100G | 3073  | 1476 | 3006 | 1469 | 33.32      | 69.38 | 34.07 | 69.71 |
| 110G | 3290  | 1624 | 3312 | 1625 | 34.24      | 69.36 | 34.01 | 69.32 |
| 120G | 3633  | 1797 | 3619 | 1793 | 33.82      | 68.38 | 33.95 | 68.53 |
| 130G | 3986  | 1910 | 3915 | 1930 | 33.40      | 69.70 | 34.00 | 68.97 |
| 140G | 4200  | 2053 | 4218 | 2090 | 34.13      | 69.83 | 33.99 | 68.59 |
| 150G | 4562  | 2208 | 4517 | 2226 | 33.67      | 69.57 | 34.00 | 69.00 |
| 160G | 4813  | 2356 | 4732 | 2369 | 34.04      | 69.54 | 33.91 | 69.16 |
| 170G | 5146  | 2496 | 5120 | 2521 | 33.83      | 69.74 | 34.00 | 69.05 |
| 180G | 5453  | 2686 | 5413 | 2662 | 33.80      | 68.62 | 34.05 | 69.24 |
| 190G | 5712  | 2795 | 5723 | 2804 | 34.06      | 69.61 | 34.00 | 69.39 |
| 200G | 6147  | 2938 | 6031 | 2953 | 33.32      | 69.71 | 33.96 | 69.35 |

- 图2.1 yarn集群wordcount处理耗时

![](/images/testdata/perf-uhadoop1.png)

- 图2.2 yarn集群wordcount处理速度

![](/images/testdata/perf-uhadoop2.png)

> BigDataBench介绍内容与下载链接：
> <http://prof.ict.ac.cn/BigDataBench/dowloads/>

## 3.Yarn集群terasort测试

使用hadoop自带的hadoop-example.jar的teragen生成200GB数据。从10GB开始，每次增加10GB数据作为测试数据源，测试terasort排序时间。生成数据和测试代码如下：

- terasort生成数据和测试代码

```
#!/bin/bash
/home/hadoop/bin/hadoop jar /home/hadoop/hadoop-examples.jar teragen -Dmapred.map.tasks=200 10737418240 /terasort/200G-input

${HADOOP_HOME}/bin/hadoop fs -mkdir -p /terasort/in/
${HADOOP_HOME}/bin/hadoop fs -mv /terasort/in/* /terasort/200G-input/
for i in {1..20}
do
        for f in `${HADOOP_HOME}/bin/hadoop fs -ls /terasort/200G-input/ | grep part | awk '{print $NF}' | head -n 10`
        do
                ${HADOOP_HOME}/bin/hadoop fs -mv $f /terasort/in/
        done
        ${HADOOP_HOME}/bin/hadoop fs -rmr /terasort/output
        echo -n "file count = "
        ${HADOOP_HOME}/bin/hadoop fs -ls /terasort/in/ | grep part | wc -l
        echo ""
        time ${HADOOP_HOME}/bin/hadoop jar /home/hadoop/hadoop-examples.jar terasort -Dmapred.reduce.tasks=50 /terasort/in /terasort/output
done
```

对4个集群的测试结果如下：

- 表3.1 yarn集群terasort测试结果

|      | 耗时(s) |      |      |      | 处理速度(MB/s) |       |       |       |
| ---- | ----- | ---- | ---- | ---- | ---------- | ----- | ----- | ----- |
|      | B2    | B2-x | D1   | D1-x | B2         | B2-x  | D1    | D1-x  |
| 10G  | 263   | 135  | 261  | 146  | 38.94      | 75.85 | 39.23 | 70.14 |
| 20G  | 440   | 218  | 473  | 275  | 46.55      | 93.94 | 43.30 | 74.47 |
| 30G  | 685   | 368  | 721  | 440  | 44.85      | 83.48 | 42.61 | 69.82 |
| 40G  | 834   | 483  | 1001 | 627  | 49.11      | 84.80 | 40.92 | 65.33 |
| 50G  | 1103  | 594  | 1251 | 824  | 46.42      | 86.20 | 40.93 | 62.14 |
| 60G  | 1352  | 796  | 1523 | 1036 | 45.44      | 77.19 | 40.34 | 59.31 |
| 70G  | 1635  | 948  | 1851 | 1144 | 43.84      | 75.61 | 38.73 | 62.66 |
| 80G  | 1851  | 1077 | 2092 | 1389 | 44.26      | 76.06 | 39.16 | 58.98 |
| 90G  | 2177  | 1357 | 2377 | 1589 | 42.33      | 67.91 | 38.77 | 58.00 |
| 100G | 2500  | 1478 | 2603 | 1906 | 40.96      | 69.28 | 39.34 | 53.73 |
| 110G | 2784  | 1536 | 2977 | 1990 | 40.46      | 73.33 | 37.84 | 56.60 |
| 120G | 3075  | 1769 | 3158 | 2149 | 39.96      | 69.46 | 38.91 | 57.18 |
| 130G | 3163  | 1874 | 3579 | 2425 | 42.09      | 71.04 | 37.19 | 54.89 |
| 140G | 3423  | 2234 | 3817 | 2767 | 41.88      | 64.17 | 37.56 | 51.81 |
| 150G | 3950  | 2058 | 4082 | 2828 | 38.89      | 74.64 | 37.63 | 54.31 |
| 160G | 4336  | 2411 | 4428 | 3131 | 37.79      | 67.96 | 37.00 | 52.33 |
| 170G | 4461  | 2573 | 4652 | 3249 | 39.02      | 67.66 | 37.42 | 53.58 |
| 180G | 4842  | 2711 | 5077 | 3733 | 38.07      | 67.99 | 36.30 | 49.38 |
| 190G | 4945  | 2826 | 5394 | 4001 | 39.34      | 68.85 | 36.07 | 48.63 |
| 200G | 5306  | 3132 | 5778 | 4240 | 38.60      | 65.39 | 35.44 | 48.30 |

- 图3.1 yarn集群terasort处理耗时

![](/images/testdata/perf-uhadoop3.png)

- 图3.2 yarn集群terasort处理速度

![](/images/testdata/perf-uhadoop4.png)

> terasort算法简介参见：
> <http://dongxicheng.org/mapreduce/hadoop-terasort-analyse/>

## 4.Hive测试

Hive测试使用了TPC-H(<http://www.tpc.org/tpch>)的数据源和测试sql进行测试。在一次测试中，会分别执行22个hive文件。这些sql基本包括在日常业务中会使用到的select,join,sort,uniq,子查询等。由于涉及到的sql很多，所以这里的单次测试时间取22条sql的总耗时。对于每条个hive文件的每次测试执行时间可以查看附件。

数据源使用TPC-H提供的方法生成。每次生成8张数据表，总数据量从10GB开始，每次增加10GB直至100GB。生成数据和测试代码如下：

- tpch-hive数据生成和测试代码

```
#!/bin/bash
size=$1'0G'
cp -R dbgen dbgen$size
cd dbgen$size
./dbgen -s ${1}0
mkdir tpch$size
mv *.tbl tpch$size/
for j in customer lineitem nation orders part partsupp region supplier
do
        mkdir tpch${size}/$j/
        mv tpch${size}/$j.tbl tpch${size}/$j/
done
/home/hadoop/bin/hadoop fs -copyFromLocal  tpch$size /tpch$size
$hadoop fs -mv /tpch${size} /tpch
cd ../TPC-H_on_Hive/tpch
for f in `ls -1 --color=never q*.hive`
do
        echo $size $f
        time hive -f $f > $size-$f.log 2>&1
done
```

对4个集群的测试结果如下表

- 表4.1 hive测试结果

|      | 耗时(s) |       |       |       | 处理速度(MB/s) |      |      |      |
| ---- | ----- | ----- | ----- | ----- | ---------- | ---- | ---- | ---- |
|      | B2    | B2-x  | D1    | D1-x  | B2         | B2-x | D1   | D1-x |
| 10G  | 6387  | 4710  | 5880  | 5384  | 1.60       | 2.17 | 1.74 | 1.90 |
| 20G  | 9256  | 6124  | 8665  | 6495  | 2.21       | 3.34 | 2.36 | 3.15 |
| 30G  | 12207 | 7518  | 11443 | 7635  | 2.52       | 4.09 | 2.68 | 4.02 |
| 40G  | 15546 | 8717  | 13964 | 9141  | 2.63       | 4.70 | 2.93 | 4.48 |
| 50G  | 17807 | 9934  | 16562 | 10622 | 2.88       | 5.15 | 3.09 | 4.82 |
| 60G  | 20714 | 11387 | 19519 | 13015 | 2.97       | 5.40 | 3.15 | 4.72 |
| 70G  | 23612 | 12767 | 22238 | 13109 | 3.04       | 5.61 | 3.22 | 5.47 |
| 80G  | 26448 | 14076 | 25128 | 14611 | 3.10       | 5.82 | 3.26 | 5.61 |
| 90G  | 30190 | 15781 | 28262 | 16824 | 3.05       | 5.84 | 3.26 | 5.48 |
| 100G | 32877 | 16979 | 32260 | 19277 | 3.11       | 6.03 | 3.17 | 5.31 |

- 图4.1 hive测试处理总耗时

![](/images/testdata/perf-uhadoop5.png)

- 图4.2 hive测试处理速度

![](/images/testdata/perf-uhadoop6.png)

## 5.Hbase读写性能测试

Hbase读写性能测试使用hbase自带的工具进行。分别测试随机写、顺序写、随机读、顺序读的性能。所有测试均为对100W行数据的读写，结果取100次测试的平均值。测试代码如下：

- hbase读写性能测试代码

```
#!/bin/bash
hbase='/home/hadoop/hbase/bin/hbase'

for test in sequentialWrite randomWrite  sequentialRead  randomRead 
do
        $hbase org.apache.hadoop.hbase.PerformanceEvaluation $test 1 > $test.log 2>&1
done
```

对5个集群的测试结果如下：

- 表5.1 hbase读写性能测试结果

| 处理速度(条/s) | B2      | B2-x     | D1      | D1-x     | vhost   |
| --------- | ------- | -------- | ------- | -------- | ------- |
| 顺序写       | 8183.09 | 11573.19 | 6695.69 | 11210.10 | 5554.05 |
| 随机写       | 7528.85 | 11914.03 | 5698.36 | 9095.78  | 5258.49 |
| 顺序读       | 1928.34 | 2723.67  | 2260.79 | 2462.00  | 1660.79 |
| 随机读       | 1795.51 | 2608.65  | 2205.75 | 2331.72  | 550.88  |

- 图5.1 hbase读写性能测试结果

![](/images/testdata/perf-uhadoop7.png)

## 6.磁盘读写性能测试

磁盘读写性能测试使用dd命令进行测试。单次读或写4k数据26214400次，共100G数据，结果取10次测试的平均值。测试代码如下：

- 磁盘读写性能测试代码

```
#!/bin/bash
#write
dd if=/dev/zero of=/data/test bs=4k count=26214400
#read
dd if=/data/test of=/dev/null bs=4k count=26214400
```

对5个集群的测试结果如下:

- 表6.1 磁盘读写性能测试结果

| 速度(MB/s) | B2-large | B2-xlarge | D1-large | D1-xlarge | vhost  |
| -------- | -------- | --------- | -------- | --------- | ------ |
| 写        | 176.80   | 174.00    | 85.84    | 101.70    | 115.07 |
| 读        | 177.30   | 171.20    | 150.50   | 170.90    | 91.81  |

- 图6.1 磁盘读写性能测试结果

![](/images/testdata/perf-uhadoop8.png)
