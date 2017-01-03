.. _installing_mi:

==============================
Mobile Ingsight 产品安装手册
==============================


软件需求
-------------------
+------------+----------------+
| 组件       | 版本           |
+============+================+
| JAVA       | 1.8.0_45 \+    |
+------------+----------------+
| CentOS     | 7              |
+------------+----------------+
| Zookeeper  | 3.4.6          |
+------------+----------------+
| Kafka      | 0.8.2.2        |
+------------+----------------+
| Redis      | 3.0.2          |
+------------+----------------+
| MySQL      | 5.6及以上      |
+------------+----------------+
| ClickHouse | 1.1.54023      |
+------------+----------------+

.. important::
  请确认以上组件全部安装成功，再开始安装Mi后台。具体的安装方法请查看中间件安装章节。

  环境需求:Mi产品和组件之间网络及相关端口胡同，请用telnet工具做网络环境检查

  centOS 上telnet工具安装和防火墙服务关闭命令示例：

  .. code-block:: shell

    #判断系统是否已经安装telnet客户端及服务端
    $ rpm -qa | grep "telnet"

    #安装使用如下命令
    $ yum install telnet

    #centos7关闭防火墙，开启防火墙
    $ systemctl stop firewalld.service

    #关闭防火墙的开机启动
    $ systemctl disable firewalld.service

安装包结构说明
-------------------

1. 解压拿到的安装包OneAPM-Mobile-Insight-Installer.tar.gz

::

  $ tar -zxvf OneAPM-Mobile-Insight-Installer.tar.gz

2. 进入解压后的目录（接下来的操作都基于这个目录执行， 协定 OneAPM-Mobile-Insight-Installer 目录为 ``$WORK_DIR``）

::

  $ cd OneAPM-Mobile-Insight-Installer

3. ``$WORK_DIR`` 目录结构如下：

::

   ├── dist
   │   ├── das-web
   │   │   ├── bin
   │   │   ├── config
   │   │   ├── lib
   │   │   ├── logs
   │   │   └── static
   │   ├── data-collector
   │   │   ├── bin
   │   │   ├── config
   │   │   ├── lib
   │   │   └── logs
   │   └── data-consumer
   │       ├── bin
   │       ├── config
   │       ├── lib
   │       └── logs
   ├── libs
   ├── metric_store
   │   ├── ajax
   │   ├── conf
   │   ├── httptransaction
   │   ├── measurement
   │   ├── session
   │   └── touch-metric-store
   │       ├── bin
   │       ├── conf
   │       └── lib
   ├── sql
   │   ├── databases
   │   └── upgradeSql
   └── tools

准备安装
-------------------

.. note::
  前置步骤:
    1. 所有需要的中间件已经正确安装
    2. 与中间件之间网络环境已经配置正确
  该步骤产出：
    1. kafka 的topic创建: ``ok``
    2. mysql 数据库及账号初始化: ``ok``

1. 创建 kafka topics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**创建topics的命令:**

::

  $ $KAFKA_HOME/bin/kafka-topics.sh --zookeeper <host:port> --topic <topic_name> --replication-factor <factor_num> --partitions <partion_num> --create

.. warning::
  kafka 集群为单节点时 ``factor-num`` 必须为 ``1``;
  ``factor-num`` <= ``集群中节点总数``

**验证topics是否创建成功**

::

  $ /opt/kafka-0.8.2.2/bin/kafka-topics.sh –list –zookeeper <host:port>

**需要创建的topics列表**

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


2. 数据库初始化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. warning::

  当且仅当全新安装Mi产品时执行该部分的所有步骤，否则请跳过！

**1. 数据库授权**

  Mi产品配置的默认数据库连接账号: ``root``, 默认密码: ``oneapm``；

用root 账号登录mysql后执行以下语句:

.. code-block:: shell

  // 1. 设置root账号的密码为 oneapm
  $ GRANT ALL PRIVILEGES ON *.* TO 'oneapm'@'%' IDENTIFIED BY 'oneapm';
  or
  $ GRANT ALL PRIVILEGES ON *.* TO 'oneapm'@'%' IDENTIFIED BY PASSWORD '*51943DF5B9B67D7BAEE2A8F29C03CEF3D9D6B862';
  // 2. flush
  $ FLUSH ALL PRIVILEGES
  // 3. 退出当前登录，测试新的登录密码
  $ exit
  $ mysql -uroot -poneapm


**2. 初始化数据库**

  创建系统需要的所有database，并插入一些初始化数据。

.. code-block:: shell

  $ cd sql
  $ sh run.sh
  or
  $ sh run.sh  /path/to/mysql   //指定mysql命令的位置
.. note::

  | ``run.sh`` 执行使用了mysql命令，请确保机器上有该命令；
  | 默认执行方式为: ``mysql -h mysql.oneapm.me -P 3306 -u root -p oneapm``
  | 如需变更 ``host``, ``port``, ``user`` or ``password`` 请直接修改脚本后在执行;

**3. 升级安装Mi产品需要执行的数据脚本** ``optional``

  根据当前安装的版本号和升级的目标版本号在 ``sql/upgradeSql`` 文件夹下找到所有需要执行的sql文件并执行；并找产品确认需要执行的sql列表，无误后再做执行；


开始安装
----------------------------

.. note::
  前置步骤:
    1. ``准备安装`` 部分正确完成
  该步骤产出：
    1. Mi 产品的相关组件：dc, dv, consumer,atosl 都正确启动


1. 配置中间件信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在 ``$WORK_DIR/install.properties`` 文件中配置各个中间件的地址 mysql, kafka, zookeeper, redis, clickhouse

.. code-block:: shell

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

2. 执行安装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. note::
  该步骤将执行以下操作：
    1. 将配置的中间件信息应用到Mi产品对应的配置文件中；
    2. 执行ip和端口检查
    3. 尝试insert测试数据到clickhouse并输出查询结果

.. code-block:: shell

    $ sh install.sh -c
    1) excuteNow   --确认配置无误，立即修改
    2) enterAgain  --配置有误需要重新输入
    1 // 选择1，进行静默安装

修改的配置文件列表：
  1. /dist/das-web/config/application.properties
  2. /dist/das-web/config/gear.properties
  3. /dist/data-collector/config/application.properties
  4. /dist/data-collector/config/gear.properties
  5. /dist/data-consumer/config/application.properties
  6. /dist/data-consumer/config/gear.properties
  7. /metric_store/conf/metric.conf

.. important::

  | ``sh install.sh -c`` 命令默认使用当前路径下的 ``install.properties`` 文件来应用变更；
  | 在项目实际实施中，可能要在多台机器上安装Mi产品的一个或者多个组件；
  | 可以备份 ``install.properties`` 文件同步到需要安装应用的机器上，
  | 在每台机器的 ``$WORK_DIR`` 下执行 ``sh install.sh -c /etc/install.properties`` 的方式来指定安装配置文件

3. 其它重要配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  这些配置目前还不支持通过install安装来自动配置，需要手动修改对应的文件

  *  企业级用户中心[optional]

      * ``$WORK_DIR/dist/das-web/config/application.properties``
      * ``$WORK_DIR/dist/data-collector/config/application.properties``

    如果没有安装企业级用户中心，请使用默认配置；
  .. code-block:: shell

        #启动该配置项后，需要修改login_domain,logout_domain为企业级用户中心通过页面访问时的机器地址加端口
        #修改login_path，logout_path为登陆页面的路径
        user-center-ee=true                             #是否使用企业级用户中心,使用为true,不使用为false
        local_session=false                             #使用企业级用户中心时为false,单点登陆时为true

        login_domain=/mobile/login                        # 需要配置为用户中心的访问domain
        login_check_domain=http://mi.oneapm.ent:8080      # 需要配置为用户中心的访问domain
        login_path=/mobile/login                          # 需要配置为用户中心的访问url
        logout_domain=http://mi.oneapm.ent:8080           # 需要配置为用户中心的访问domain
        logout_path=/mobile/logout                        # 需要配置为用户中心的访问url

        #mi页面访问时的机器地址加端口
        mi.host.facade=http://127.0.0.1:8080

  .. note::

    用户中心与Mi产品是通过cookie实现统一登录控制的，所以在部署时请将Mi Dv 部署到与中户中心相同的顶级域名下；例如：``oneapm.ent``,  ``mi.oneapm.ent``, ``user.oneapm.ent``

  * 告警模块 //TODO

      * ``$WORK_DIR/dist/das-web/config/application.properties``


  * 符号化服务 //TODO

      * ``$WORK_DIR/dist/das-web/config/application.properties``
      * ``$WORK_DIR/dist/das-consumer/config/application.properties``

4. 服务启动、关闭
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  Mi 产品的相关组件：dc, dv, consumer, atosl 的启动停止脚本

.. code-block:: shell

  $ sh start.sh [<component-name> [JAVA_OPTS]]

.. code-block:: shell

  $ sh shutdown.sh [component-name]

.. note::
  | ``start.sh`` 脚本说明：
  | 1. 不带任何参数，启动所有组件;
  | 2. 带1个参数  component-name，仅启动该component的进程;
  | 3. 带2个参数  component-name JAVA_OPTS， JAVA_OPTS 为标准的jvm启动参数，请使用双引号;

  | 默认启动的 ``JAVA_OPTS`` :
  | -server -Duser.timezone=GMT+08 -Xms1g -Xmx1g -XX:PermSize=128m -XX:MaxPermSize=128m -XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:+UseConcMarkSweepGC -XX:+DisableExplicitGC -XX:+UseCMSCompactAtFullCollection -XX:CMSMaxAbortablePrecleanTime=5000 -XX:+CMSClassUnloadingEnabled -XX:CMSInitiatingOccupancyFraction=80 -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:+DisableExplicitGC
.. note::
  | ``shutdown.sh`` 脚本说明：
  | 1. 不带任何参数，停止所有组件;
  | 2. 带1个参数  component-name，仅停止该component的进程;

1. dc

.. code-block:: shell

  $ sh start.sh dc "-Xmx10240m -Xms10240m -Xmn5120m"
  $ sh shutdown.sh dc

2. dv | atosl符号化

.. code-block:: shell

  $ sh start.sh dv "-Xmx10240m -Xms10240m -Xmn5120m"
  $ sh shutdown.sh dv

3. consumer

.. code-block:: shell

  $ sh start.sh consumer "-Xmx10240m -Xms10240m -Xmn5120m"
  $ sh shutdown.sh consumer

4. MetricStore

.. code-block:: shell

  $ sh start.sh metric_store
  $ sh shutdown.sh metric_store

5. 日志文件配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  使用启动脚本启动Mi组件的默认日志文件路径在各个组件的logs目录之下，如需修改请配置 ``application.properties`` 文件的 ``logging.path`` 配置


.. warning::

  当前版本的配置文件还没有做好统一，key/value 间隔符使用的是 ``:`` 或者 ``=``, 在修改配置文件时，请确保使用与该文件中其它配置一致的分隔符
