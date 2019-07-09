{{indexmenu_n>11}}

## Python开发指南

如果使用pyspark进行机器学习方面的数据分析，需要在集群上安装一些python依赖包。这里将介绍常用的几个依赖包的安装方法。更多的依赖包下载及安装，可以参考[PyPI网站](https://pypi.python.org)。

> 因为部分依赖包不支持2.6版本。所以，以下所有安装均以Python2.7为例。建议将集群上的Python升级到2.7版本。

\>

### 1\. NumPy

[NumPy](http://www.numpy.org/)一个用python实现的科学计算包，可用来存储和处理大型矩阵，比Python自身的嵌套列表结构要高效的多。

最新版NumPy可以在[PyPI网站](https://pypi.python.org/pypi?%3Aaction=search&term=numpy&submit=search)搜索到，numpy-1.12.0版本可以[点击下载](http://uhadoop-new.ufile.ucloud.com.cn/python/numpy-1.12.0.zip)。

以numpy-1.12.0版本为例，安装方法如下：

```
unzip numpy-1.12.0.zip
cd numpy-1.12.0
python setup.py install
```

### 2\. SciPy

[SciPy](https://www.scipy.org/)是一款专为科学和工程设计的Python工具包。

最新版SciPy可以在[PyPI网站](https://pypi.python.org/pypi?%3Aaction=search&term=scipy&submit=search)搜索到，scipy-0.18.1版本可以[点击下载](http://uhadoop-new.ufile.ucloud.com.cn/python/scipy-0.18.1.tar.gz)。

> 在安装Scipy前，需要先安装好NumPy。

\>

以scipy-0.18.1版本为例，安装方法如下：

```
tar zxf scipy-0.18.1.tar.gz
cd scipy-0.18.1
python setup.py install
```

### 3\. Scikit-Learn

[Scikit-Learn](http://scikit-learn.org/)是SciPy下，专门面向机器学习的工具包。

最新版Scikit-Learn可以在[PyPI网站](https://pypi.python.org/pypi?%3Aaction=search&term=scikit-learn&submit=search)搜索到，scikit-learn-0.18.1版本可以[点击下载](http://uhadoop-new.ufile.ucloud.com.cn/python/scikit-learn-0.18.1.tar.gz)。

> 在安装Scikit-Learn前，需要先安装好NumPy和Scipy。

\>

以scikit-learn-0.18.1版本为例，安装方法如下：

```
tar zxf scikit-learn-0.18.1.tar.gz
cd scikit-learn-0.18.1
python setup.py install
```

### 4\. Sympy

[SymPy](http://sympy.org/)是Python的数学符号计算库，用它可以进行数学公式的符号推导。

最新版SymPy可以在[PyPI网站](https://pypi.python.org/pypi?%3Aaction=search&term=sympy&submit=search)搜索到，sympy-1.0版本可以[点击下载](http://uhadoop-new.ufile.ucloud.com.cn/python/sympy-1.0.tar.gz)。

以sympy-1.0版本为例，安装方法如下：

```
tar zxf sympy-1.0.tar.gz
cd sympy-1.0
python setup.py install
```

### 5\. Pandas

[Pandas](http://pandas.pydata.org/) (Python Data Analysis
Library)是基于NumPy的解决数据分析任务的一种工具。

最新版Pandas可以在[PyPI网站](https://pypi.python.org/pypi?%3Aaction=search&term=pandas&submit=search)搜索到，pandas-0.19.2版本可以[点击下载](http://uhadoop-new.ufile.ucloud.com.cn/python/pandas-0.19.2.tar.gz)。

以pandas-0.19.2版本为例，安装方法如下：

```
tar zxf pandas-0.19.2.tar.gz
cd pandas-0.19.2
python setup.py install
```

### 6\. Matplotlib

[Matplotlib](http://matplotlib.org/)是Python常用的绘图库，它提供了一整套和matlab相似的命令API，十分适合交互式地进行制图。

最新版Matplotlib可以在[PyPI网站](https://pypi.python.org/pypi?%3Aaction=search&term=matplotlib&submit=search)搜索到，matplotlib-2.0.0版本可以[点击下载](http://uhadoop-new.ufile.ucloud.com.cn/python/matplotlib-2.0.0.tar.gz)。

以matplotlib-2.0.0版本为例，安装方法如下：

```
yum install libpng-devel libpng -y
tar zxf matplotlib-2.0.0.tar.gz
cd matplotlib-2.0.0
python setup.py install
```

### 7\. MySQLdb

[MySQLdb](https://pypi.python.org/pypi/MySQL-python/)是Python提供的连接MySQL的接口。

最新版MySQLdb可以在[PyPI网站](https://pypi.python.org/pypi/MySQL-python/)搜索到，MySQL-python-1.2.5版本可以[点击下载](http://uhadoop-new.ufile.ucloud.com.cn/python/MySQL-python-1.2.5.zip)。

以MySQL-python-1.2.5版本为例，安装方法如下：

```
yum install python-pip python-devel mysql-devel zlib-devel openssl-devel -y
unzip MySQL-python-1.2.5.zip
cd MySQL-python-1.2.5
python setup.py install
```
