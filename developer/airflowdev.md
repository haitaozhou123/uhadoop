{{indexmenu_n>9}}

# Airflow开发指南

Airflow是一个工作流分配管理系统，它通过有向无环图的方式管理任务流程，可以设置任务依赖关系和时间调度。Airflow服务会在master1上启动2个服务，AirflowScheduler与AirflowWebserver。AirflowScheduler用于管理所有DAGs、Tasks和Tasks之间的调度，AirflowWebserver是Airflow的web服务，用于可视化管理。

如果您创建集群勾选了Airflow，Airflow将被安装在uhadoop-\*\*\*\*\*\*-master1节点上。访问Airflow
Web服务通过master1节点外网IP:8999访问（需要您开放master1节点绑定的外网防火墙8999端口）

## 1. Airflow示例

详情请参考[官网介绍](http://pythonhosted.org/airflow/tutorial.html)

请将如下代码放置于uhadoop-\*\*\*\*\*\*-master2的/home/hadoop/airflow/dags目录下

> 如无此目录，则需手动创建，并修改用户组为hadoop

```
cat tutorial.py
```

``` python
"""
Code that goes along with the Airflow tutorial located at:
https://github.com/airbnb/airflow/blob/master/airflow/example_dags/tutorial.py
"""
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta


default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2015, 6, 1),
    'email': ['airflow@airflow.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
}

dag = DAG('tutorial', default_args=default_args)

# t1, t2 and t3 are examples of tasks created by instantiating operators
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=dag)

templated_command = """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
        echo "{{ params.my_param }}"
    {% endfor %}
"""

t3 = BashOperator(
    task_id='templated',
    bash_command=templated_command,
    params={'my_param': 'Parameter I passed in'},
    dag=dag)

t2.set_upstream(t1)
t3.set_upstream(t1)
```

页面刷新后将会存在 tutorial DAG

详细使用方法请参考[Airflow官网网站](http://pythonhosted.org/airflow/)
