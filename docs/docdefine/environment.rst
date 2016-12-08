.. _environment:


==================
environment
==================

   
软件需求
----------

+------------+----------------+
| 组件       | 版本           |
+============+================+
| Python     |python2或3      |
+------------+----------------+

.. important::
  python 环境
  
  
python下载
-----------
::

    linux用户下载
    https://www.python.org/ftp/python/3.4.5/Python-3.4.5.tgz

    window、Mac用户根据操作系统下载相应版本
    https://www.python.org/downloads/release/python-352
	

linux安装
-----------------
-  **解压** ，tar -zxvf Python-3.4.5.tgz
-  **configure** ，进入解压Python-3.4.5包，执行./configure
-  **make** ，编译源代码
-  **make install**， 把编译的可执行文件拷贝到Linux相应目录，使所有用户可用

.. Warning::
  linux安装完成python后，还需要单独安装，pip命令。

Mac安装
---------


widow安装
---------
-  **安装**， widow环境可以下载python安装可执行文件,按照操作提示安装到磁盘
-  **python_home**，建立python_home环境变量，将%python_home%添加到path路径，将%python_home%\Scripts添加到path路径，此路径包含pip命令

