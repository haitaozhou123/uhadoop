{{indexmenu_n>5}}

# Pig开发指南

## 1\. 简单示例

1.  上传数据到hdfs目录



```
[hadoop@uhadoop-******-master1 pig]$ hadoop fs -put /etc/passwd /user/hadoop/passwd
```

1.  启动pig



```
[hadoop@uhadoop-******-master1 pig]$ pig
```

1.  加载数据



```
grunt> A = load 'passwd' using PigStorage(':');
grunt> dump A;
```

显示结果：

```
(root,x,0,0,root,/root,/bin/bash) 
……
```

## 2\. 使用UDF

\- 准备数据

student 文件内容

    any 9 5
    bob 8 4

上传student文件

```
hdfs dfs -put student /user/root/student
```

\- 示例代码

``` java
package myudfs;
import java.io.IOException;
import org.apache.pig.EvalFunc;
import org.apache.pig.data.Tuple;

public class UPPER extends EvalFunc<String>
{
    public String exec(Tuple input) throws IOException {
        if (input == null || input.size() == 0 || input.get(0) == null)
            return null;
        try{
            String str = (String)input.get(0);
            return str.toUpperCase();
        }catch(Exception e){
            throw new IOException("Caught exception processing input row ", e);
        }
    }
}
```

\- 编译

```
cd myudfs
javac -cp $ PIG_HOME/pig-0.12.0-cdh5.4.4.jar UPPER.java
cd ..
jar -cf myudfs.jar myudfs
```

**测试脚本upper.pig**

``` pig
REGISTER myudfs.jar;
A = LOAD 'student' AS (name: chararray, age: int, gpa: float);
B = FOREACH A GENERATE myudfs.UPPER(name);
DUMP B;
```

\- 执行

```
pig upper.pig
```

\- 输出结果

    (ANY 9 5)
    (BOB 8 4)
