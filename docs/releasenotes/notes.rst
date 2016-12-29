.. _releasenotes_notes:


v4.5.1[#f2]_ (released 12 28, 2016)
------------------------------------

**Improvements**

- #91: 关闭“返回旧总览”入口
- #92: 性能数据不再存入数据库
- 去除appUseCount依赖
- #86: 压力测试和性能评估
- 迁移日活的数据查询. 不使用 redis，而是直接从MetricStore获取数据
- 关闭"组合分析"功能;（暂时关闭，后续得看功能在MetricStore的支持程度，逐步迁移）

**Bugs fixed**

- #99: 应用版本查询为空
- #111: 页面查询请求参数错误


v4.5.0[#f1]_ (released 12 05, 2016)
------------------------------------

**Features**

- 支持 MetricStore 替代Druid作为时序数据存储查询引擎
- 使用SpringBoot架构
- 集成企业级用户中心 ``3.4.19``

**Improvements**

- 优化部署方式，提供 ``install.sh`` , ``start.sh`` , ``shutdown.sh`` 等运维脚本


.. rubric::

.. [#f2] `v4.5.1[55fc38d1]<http://git.oneapm.me/mobile/das/commits/v4.5.1>`_
.. [#f1] `v4.5.0[1543fd73]<http://git.oneapm.me/mobile/das/commits/v4.5.0>`_
