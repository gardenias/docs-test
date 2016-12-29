.. _installing_kafka:

==================
Kafak & Zookeeper
==================

::
  请参考 http://wiki.oneapm.me/pages/viewpage.action?pageId=14325153

软件需求
------------------------

+------------+----------------+
| 组件       | 版本           |
+============+================+
| JAVA       | 1.8.0_45 及以上|
+------------+----------------+

.. important::
	请配置 ``JAVA_HOME`` 环境变量

下载与解压
------------------------

::
  $ wget http://www.eu.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
  $ wget https://dist.apache.org/repos/dist/release/kafka/0.8.2.2/kafka_2.9.2-0.8.2.2.tgz
  $ tar –zxvf  zookeeper-3.4.6.tar.gz
  $ tar -zxvf  kafka_2.9.2-0.8.2.2.tgz


zookeeper配置
------------------------

-   **单机版**

进入解压的zookeeper目录，进入子目录conf,会发现zoo_sample.cfg文件，这是apche官方网站提供的默认配置文件，根据配置需求修改相应配置，
并重命名zoo_sample.cfg为zoo.cfg。

配置详细 ``zoo.cfg``
::

	#zookeeperj监听端口
	clientPort=2181
	#持久化数据存放目录
	dataDir=/data
	#存放日志文件目录
	dataLogDir=/datalog
	#zookeeper时间单位常量，zookeeper时间单位是多少个tick的
	tickTime=2000
	#与集群配置相关选项，是指leader与follower请求应答，最多不超过5tickTime
	syncLimit=5
	#是指zookeeper接受客户端初始化最长时间，超过这个时间，zookeeper认为客户端连接失败
	initLimit=10

-  **集群版**

与单机版相比较，集群版还需在每一个zoo.cfg要添加一下配置,假设两台zookeeper服务组成一个集群。``注意:`` 需要在dataDir目录下创建myid文件，
该文件只有一行，内容是该server的id，下面配置中，server.1的myid是1。

::
  server.1=10.128.9.134:2888:3888  //2888是集群信息交流端口，leader与followers 用这个端口建立tcp连接
  server.2=10.128.9.135:2888:3888  //3888是集群建立tcp连接选举产生leader的端口

zookeeper启动
------------------------
zookeeper启动脚本，在bin目录下，启动命名如下

::
	$ ./zkServer.sh start


kafka配置
------------------------

-   **单机版** ``$KAFKA_HOME/config/server.properties``

::
	#broker的唯一标识符
	broker.id=0
	#broker 配置的ip
	host.name=10.128.9.135
	#broker 端口号
	port=9092
	#broker向zookeeper注册的名称
	advertised.host.name=10.128.9.135
	#broker向zookeeper注册端口
	advertised.port=9092
	#根据zookeeper安装主机IP和端口号
	zookeeper.conect=10.128.9.135:2181
	#配置成true topic可以被删除
	delete.topic.enable=true
	#kafka日志存放目录
	log.dirs=/tmp/log
	#默认partitions数目
	num.partitions=1


-  **集群版**

与单机版相比较,集群版需要修改配置
::

	#注意每一台broker服务的ID不能相同
	broker.id=1
	zookeeper.host.name=10.128.9.134:2181,10.128.9.135:2181

kafka启动
------------
kafka启动脚本，在bin目录下。deamon参数启动后自动退出日志，建议第一次启动，不带deamon参数，
查看日志是否报错，如果没有错误信息，退出，kafka进程结束，添加deamon参数启动。
::

  $ $KAFKA_HOME/bin/kafka-server-start.sh  [-deamon] config/server.properties


环境验证
------------

.. important::
	kafak & zookeeper 环境验证，主要是通过 ``kafka-console-producer.sh`` & ``kafka-console-consumer.sh`` 来检验环境是佛配置正确以及网络条件是否是能连通的；

- 开启一个命令行窗口执行一下命令， ``不要执行 Ctrl + C``

::

	$ $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list <broker1:port1>[,broker2:port2] --topic test

- 新开一个命令行窗口执行：

::
	$ $KAFKA_HOME/bin/kafka-console-consumer.sh --zookeeper <zookeeper1:port1>[,zookeeper2:port2] --topic test

常用命令
------------

- 查看topic 列表

::
	$ $KAFKA_HOME/bin/kafka-topics.sh --list --zookeeper <zookeeper1:port1>[,zookeeper2:port2]

- 创建topic

::
	$ $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper <zookeeper1:port1>[,zookeeper2:port2] --replication-factor <factor_num> --partitions <partition_num> --topic  <topic_name>

- 查看topic配置

::
	$ $KAFKA_HOME/bin/kafka-topics.sh --describe --zookeeper <zookeeper1:port1>[,zookeeper2:port2] --topic <topic_name>
