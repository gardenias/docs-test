.. _installing_clickhouse:

==================
Click House
==================


.. important::

	Centos 7 （推荐）
	Centos 6 （暂时不支持）


下载与解压
------------------
ftp://10.128.9.10/研发中心/基础构架部/基础服务组/clickHouse/54023/clickHouse最新版本（1.1.54023） CentOS X86_64安装包（直接点链接打不开，需要手动输入路径）

::

	$ mkdir -p /home/metrika/clickhouse
	$ tar zxf  clickhouse_1.1.54023.tar.gz -C /home/metrika/clickhouse


.. important::

	安装ClickHouse 需要 sudo 权限

执行安装
------------------
::

	$ cd /home/metrika/clickhouse
	$ sh install.sh

停止服务
------------------
::

	$ /etc/init.d/clickhouse-server start

启动服务
------------------
::

	$ /etc/init.d/clickhouse-server stop

**more commands**

::

	$ service clickhouse-server <start|stop|status...>

OR

::

	$ /etc/init.d/clickhouse-server <start|stop|status...>



验证服务
------------------
+ 查看到8123端口监听状态
:: 
    
    $ netstat -anp | grep 8123
    tcp6    0  0 :::8123         :::*      LISTEN   30883/clickhouse-se
 
+ 访问服务地址返回ok状态
::

    $ curl localhost:8123
    Ok.

