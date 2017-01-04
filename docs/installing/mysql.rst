.. _installing_mysql:

===============
MySQL
===============

1.数据库设置
-----------------------

要支持中文，需要将mysql设置成utf-8编码，设置方法如下：

  .. code-block:: shell
  
   1. vim /etc/my.cnf
   2. [mysqld]下添加：character_set_server = “utf8” ，保存
   3. 重启数据库：service mysqld restart
   4. mysql –uroot –p 回车 #进入mysql，查看修改后的编码格式
      show variables like 'character%';
	  