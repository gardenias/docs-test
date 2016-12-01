.. _installing_middleware:

===============
公共组件安装
===============

clickhouse安装
--------------
系统要求
Centos 7 （推荐）
Centos 6 （暂时不支持）

下载安装包
ftp://10.128.9.10/研发中心/基础构架部/基础服务组/clickHouse/54023/clickHouse最新版本（1.1.54023） CentOS X86_64安装包（直接点链接打不开，需要手动输入路径）

解压安装包
需要root权限？？？？

.. code-block:: shell
	mkdir -p /home/metrika/clickhouse
	tar zxf  clickhouse_1.1.54023.tar.gz -C /home/metrika/clickhouse
	
执行安装
.. code-block:: shell
cd /home/metrika/clickhouse
sh install.sh

启停服务
.. code-block:: shell
/etc/init.d/clickhouse-server start

service clickhouse-server start/stop/status... 或者 /etc/init.d/clickhouse-server start/stop/status...

验证服务
.. code-block:: shell

1. 查看到8123端口监听状态
    netstat -anp | grep 8123
    tcp6    0  0 :::8123         :::*      LISTEN   30883/clickhouse-se
 
2. 访问服务地址返回ok状态
    curl localhost:8123
    Ok.


kafka/zookeeper安装 （单机版/集群版）
请参考 http://wiki.oneapm.me/pages/viewpage.action?pageId=14325153
------------
环境准备
Kafka 安装前，确保机器上安装JDK7或8，并正确配置JAVA_HOME
检测是否已安装jdk     java -version

Zookeeper下载地址
http://www.eu.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
Kafka 下载地址
https://dist.apache.org/repos/dist/release/kafka/0.8.2.2/kafka_2.9.2-0.8.2.2.tgz

单机版
step 1: 下载zookeeper、kafka安装包
wget   http://www.eu.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
wget   https://dist.apache.org/repos/dist/release/kafka/0.8.2.2/kafka_2.9.2-0.8.2.2.tgz
step2 ：解压tar包
tar –zxvf  zookeeper-3.4.6.tar.gz
tar –zxvf kafka_2.9.2-0.8.2.2.tgz
step3 :配置zookeeper 、kafka   
 zookeeper配置，conf/zoo.cfg   
clientPort=2181    //zookeeper端口号
dataDir=/data    //持久化数据存放目录
dataLogDir=/datalog   //存放日志文件目录
tickTime=2000   //zookeeper时间单位常量，zookeeper时间单位是多少个tick的
kafka配置，config/server.properties
broker.id=0 //单机版设置成0
port=9092  //kafka 端口号
log.dirs=/tmp/log  //kafka日志存放目录
num.partitions=1  //默认partitions数目
zookeeper.conect=localhost:2181 //根据zookeeper安装主机IP和端口号
delete.topic.enable=true  //配置成true topic可以被删除

step4:启动zookeeper   bin/zkServer.sh start 
step5:启动kafka      bin/kafka-server-start.sh  [-deamon] config/server.properties

常用命令：
查看当前zookeeper  topic列表
./kafka-topics.sh --zookeeper localhost:2181 --list
创建一个topic
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 4 --topic  test
删除一个topic
   ./kafka-topics.sh --delete --zookeeper localhost:2181  --topic test
控制台producer
./kafka-console-producer.sh --broker-list localhost:9092 --topic test
控制台consumer
./kafka-console-consumer.sh --zookeeper localhost:2181 --topic test

集群版
Step1: 将下载的zookeeper kafka分别上传部署的机器上
step2 ：解压tar包（见单机版）
step3 :配置zookeeper 、kafka
 zookeeper配置，除了单机版基本配置外，zookeeper集群部署zoo.cfg配置文件需添加配置（以部署两台zookeeper为例）
server.1=10.128.9.134:2888:3888
server.2=10.128.9.135:2888:3888
以上配置中端口号，2888是集群服务中zookeeper之间交流信息监听端口，集群中产生leader后，followers 就用这个端口同leader建立tcp链接; 3888端口是用来在集群间建立tcp链接，选举产生leader。
Kafka配置，config/server.properties
zookeeper.connect=10.128.9.134:2181,10.128.9.135:2181
如果部署三个broker节点，以kafka config/server.properties事例（假设broker部署节点是：10.128.9.131，10.128.9.132，10.128.9.133）
10.128.9.131配置
broker.id=0
port=9092  
log.dirs=/tmp/log  
num.partitions=1  
zookeeper.conect=10.128.9.134:2181,10.128.9.135:2181
delete.topic.enable=true  
10.128.9.132配置
broker.id=1
port=9092  
log.dirs=/tmp/log  
num.partitions=1  
zookeeper.conect=10.128.9.134:2181,10.128.9.135:2181
delete.topic.enable=true  
10.128.9.133配置
broker.id=2
port=9092  
log.dirs=/tmp/log  
num.partitions=1  
zookeeper.conect=10.128.9.134:2181,10.128.9.135:2181
delete.topic.enable=true  
step4:启动每个节点zookeeper   bin/zkServer.sh start 
step5:启动每个节点kafka      bin/kafka-server-start.sh  [-deamon] config/server.properties

常用命令：
查看当前zookeeper集群topic列表 topic列表
./kafka-topics.sh -- zookeeper 10.128.9.134:2181,10.128.9.135:2181/kafka  --list
创建topic
  ./kafka-topics.sh --create  --zookeeper 10.128.9.134:2181,10.128.9.135:2181/kafka  --replication-factor 1 --partitions 4 --topic  test
删除一个topic
   ./kafka-topics.sh --delete  –zookeeper 10.128.9.134:2181,10.128.9.135:2181/kafka  --topic test
控制台producer
./kafka-console-producer.sh   --broker-list 10.128.9.131:9092,10.128.9.132:9092,10.128.9.133 :9092       --topic test
控制台consumer
./kafka-console-consumer.sh --zookeeper 10.128.9.134:2181,10.128.9.135:2181/kafka  --topic test


