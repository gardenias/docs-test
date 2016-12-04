.. _installing_mi:

==============================
Mobile Insight Installing
==============================


Requirements
-------------------

+------------+----------------+
| 组件       | 版本           |
+============+================+
| JAVA       | 1.8.0_45及以上 |
+------------+----------------+
| CentOS     | 7              |
+------------+----------------+
| Zookeeper  | 官方最新版     |
+------------+----------------+
| Kafka      | 0.8.2.2        |
+------------+----------------+
| Redis      | 官方最新版     |
+------------+----------------+
| MySQL      | 5.6及以上      |
+------------+----------------+
| ClickHouse | 1.1.54023      |
+------------+----------------+


安装包结构说明
-------------------

**解压**

解压拿到的安装包OneAPM-Mobile-Insight-Installer.tar.gz

::

  $ tar -zxvf OneAPM-Mobile-Insight-Installer.tar.gz

进入解压后的目录（接下来的操作都基于这个目录执行）

::

  $ cd OneAPM-Mobile-Insight-Installer


创建 kafka topics
-------------------

1. dc ``must``

+--------------------------+----------------------+----------------------+
| topic_name               |     factor-num       |      partion-num     |
+==========================+======================+======================+
| mi_ol_agent_original     |          1           |         8            |
+--------------------------+----------------------+----------------------+

2. consumer ``must``

+--------------------------+----------------------+----------------------+
| topic_name               |     factor-num       |      partion-num     |
+==========================+======================+======================+
| mi_tl_format_ajax        |          1           |         8            |
+--------------------------+----------------------+----------------------+
| mi_tl_format_http        |          1           |         8            |
+--------------------------+----------------------+----------------------+
| mi_tl_format_session     |          1           |         8            |
+--------------------------+----------------------+----------------------+
|mi_tl_format_measurement  |          1           |         8            |
+--------------------------+----------------------+----------------------+
 
3.  告警模块 ``optional`` [如果安装了系统告警模块，请创建以下topics]

+--------------------------+----------------------+----------------------+
| topic_name               |     factor-num       |      partion-num     |
+==========================+======================+======================+
| as_jl_mi_event           |          1           |         8            |
+--------------------------+----------------------+----------------------+
| as_jl_mi_alert           |          1           |         8            |
+--------------------------+----------------------+----------------------+

**command for topic creation:**
::

  $ $KAFKA_HOME/bin/kafka-topics.sh --zookeeper zookeeper <host:port> --topic <topic_name>   --replication-factor <factor-num>   --partitions <partion-num>  --create

.. important::
  
  kafka 集群为单节点时 ``factor-num`` 必须为 ``1``;
  ``factor-num`` <= ``集群中节点总数``


**验证topic是否创建成功**

::

  $ /opt/kafka-0.8.2.2/bin/kafka-topics.sh –list –zookeeper zookeeper地址:2181


topic配置文件位于./consumer/conf/topicInfo.json,可通过修改配置文件来配置topic的partition和replication个数。
目前安装过程会创建如下topic:




Mysql和ClickHouse初始化
-------------------
sql文件位于OneAPM-Mobile-Insight-Installer/sql目录下

开始安装 Mobile Insight
-------------------
目前该版本仅支持全量安装

^^^^^^^^^^^^^^^
1.需要执行sql/databases目录下所有的sql文件
2.需要单独执行sql/upgradeSql目录下das-ee-4.3.2-alarm.sql文件


配置说明
=========

DC/DV/CONSUMER配置修改
~~~~~~~~~~~~~~~~~~~~~~
基础配置mysql|kafka|zookeeper|redis|clickhouse地址在/install.properties文件中配置

.. code-block:: shell

  vi install.properties
  #格式ip:port
  mysql_ip=MYSQL_IP:PORT
  mysql_username=MYSQL_USER
  mysql_password=MYSQL_PASSWORD

  #格式ip:port,多个使用逗号分割
  kafka_ip=KAFKA_IP:PORT

  #格式ip:port,多个使用逗号分割
  zookeeper_ip=ZOOKEEPER_IP:PORT

  #格式ip:port
  redis_ip=REDIS_IP:PORT
  
  #格式password，不需要密码则设置为n
  redis_password=REDIS_PASSWORD

  #metric-store启动的默认ip和端口，使用默认配置即可
  query_ds_url=jdbc:METRIC_STORE://METRIC_STORE_IP:PORT/all?f=druid

  #ip
  clickHouse_ip=CLICKHOUSE_IP

  #port
  clickHouse_port=CLICKHOUSE_PORT

举例：

.. code-block:: shell

  #格式ip:port
  mysql_ip=10.128.9.134:3302
  mysql_username=root
  mysql_password=oneapm

  #格式ip:port,多个使用逗号分割
  kafka_ip=10.128.9.132:9092

  #格式ip:port,多个使用逗号分割
  zookeeper_ip=10.128.9.132:2181

  #格式ip:port
  redis_ip=10.128.9.132:6379
  #不需要密码则设置为n
  redis_password=123456

  #metric-store启动的默认ip和端口，使用默认配置即可
  query_ds_url=jdbc:METRIC_STORE://10.128.9.132:9123/all?f=druid

  #ip
  clickHouse_ip=10.128.9.135

  #port
  clickHouse_port=8123

  
确认修改配置

.. code-block:: shell

  sh install.sh -c
  1) excuteNow   --确认配置无误，立即修改
  2) enterAgain  --配置有误需要重新输入


METIRC_STORE配置修改
~~~~~~~~~~~~~~~~~~~~~~
metrc_store的相关配置信息在/metric_store/conf/metric.conf文件中配置

.. code-block:: shell

  consumer_dir=touch-metric-store
  click_house_ip=CLICKHOUSE_IP
  click_house_port=CLICKHOUSE_PORT
  bootstraps=KAFKA_IP:PORT
  zookeeper=ZOOKEEPER_IP:PORT
  metric_store_port=METRIC_STORE:PORT
  
举例：

.. code-block:: shell

  consumer_dir=touch-metric-store
  click_house_ip=10.128.9.135
  click_house_port=8123
  bootstraps=10.128.9.132:9092
  zookeeper=10.128.9.132:2181
  metric_store_port=9123
                     
 确认修改配置
 
 .. code-block:: shell

   sh setup.sh
 

启停服务
=========

一次性启停所有模块
-------------------

各模块日志文件在dist/das-web/config/application.properties文件中可以配置
修改各模块日志目录可以通过以下命令:

.. code-block:: shell

  DC: vi application.properties --修改logging.path
  DV: vi application.properties --修改logging.path
  Consumer: vi application.properties  --修改logging.path

启停所有模块

.. code-block:: shell

  ./startup.sh
  ./shutdown.sh

单独模块的启停
-------------------
其中单独启动DC,DV,Consumer的时候,执行命令后可跟若干个jvm参数

启停DC
^^^^^^^^^^

.. code-block:: shell

  ./dc/bin/start.sh dc 
  ./dc/bin/shutdown.sh dc 


启停DV
^^^^^^^^^^

.. code-block:: shell

  ./dv/bin/start.sh dv 
  ./dv/bin/shutdown.sh dv



启停Consumer
^^^^^^^^^^^^^^^

.. code-block:: shell

  ./consumer/bin/start.sh consumer
  ./consumer/bin/shutdown.sh consumer


启停MetricStore
^^^^^^^^^^^^^^^^

.. code-block:: shell

  ./metric-store/bin/startup.sh
  ./metric-store/bin/shutdown_ms.sh





