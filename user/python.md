{{indexmenu_n>8}}

## Python

## 如何为Python安装新的库？

1.yum安装

可以使用yum search命令来查找具体的包名称

> 请确认ucloud源上的版本是否和预期的版本一致

2.pip安装

如果本地源上面没有，yum和pip都可以通过设置代理来通过有外网权限的机器来下载

参考[yum设置代理](https://my.oschina.net/yinlei212/blog/142434)和[pip设置代理](http://blog.csdn.net/li575098618/article/details/49817055)

3.源码安装

可以在[PyPI网站](https://pypi.python.org)搜索需要的依赖包。下载后解压，并执行`python setup.py
install`来安装。

具体安装步骤，可以参考[Python开发指南](/analysis/uhadoop/developer/pythondev)。
