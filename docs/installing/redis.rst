.. _installing_redis:

===============
Redis
===============

1.安装说明
---------------

1. 安装前说明
  Linux环境，需要g++，make等，也可以安装在win下的类linux环境cygwin
  
2. 下载
  Redis可以到官方网站：http://www.redis.io/download下载

3. Redis介绍
  Redis是Remote Dictionary Server的缩写。他本质上一个Key/Value数据库，与Memcached类似的NoSQL型数据库；但是他的数据可以持久化的保存在磁盘上，解决了服务重启后数据不丢失的问题，他的值可以是string
  但是他的数据可以持久化的保存在磁盘上，解决了服务重启后数据不丢失的问题，他的值可以是string(字符串)、list(列表)、sets(集合)或者是ordered sets(被排序的集合)；
  所有的数据类型都具有push/pop、add/remove、执行服务端的并集、交集、两个sets集中的差别等等操作，这些操作都是具有原子性的，Redis还支持各种不同的排序能力。

4. 编译及安装
  解压Redis包：tar -xvf redis_${version}.tar
  进入redis的解压目录，执行如下命令：
  
  .. code-block:: shell
  
    cd ${REDIS_DIR}
    make
	
	
5. 启动和验证

  .. code-block:: shell
    
	#启动  
	$ src/redis-server redis.conf
	
	#验证
	$ src/redis-cli
  
