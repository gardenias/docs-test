.. _tech_daus:

==================================
Daily Active Users [日活]
==================================


----------------------------------



Interfaces
------------------------------------------------------------------------------------
活跃会话数：session.json
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
通过BasicServiceCreateNewItemSwitch.DRUID_SWITCH 控制，查询session数据统计活跃会话

``else 分支走的也是druid查询 ？``

日活：activeSessionSeries.json
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#. 如果 BasicServiceCreateNewItemSwitch.REDIS_SWITCH=true：从redids查询
#. 否则从 dao.queryActiveSessionSeries 从druid查询

.. tip::
    不支持从数据库查询

周活|月活：activeSessionSeriesNew.json
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: java

  public ListZ<CustomPoint<SimpleResult>> activeSessionSeriesWithPoolNew(
          final MiscFilterParams misc, final AppFilterParams aps,
          final ListZ<TimeFilterParams2> tpss, final OSFilterParams ops,
          final AreaFilterParams areaps, final DeviceFilterParams dps,
          final CarriesFilterParams cps, final LimitFilterParams lps,
          final String queryType) {
      ListZ<CustomPoint<SimpleResult>> result = ListZ.newListZ();

      ListZ<Callable<CustomPoint<SimpleResult>>> cs = ListZ.newListZ();
      for (final TimeFilterParams2 tps : tpss) {
          cs.put(() -> repo.queryActiveSessionSeriesNew(misc, aps, tps, ops, areaps, dps, cps, lps, queryType));
      }

      return result.putAll(POOL.run(cs).resultList);
  }

.. tip::
  ``repo.queryActiveSessionSeriesNew`` 当且仅当redis可用时从redis获得活跃数数据，否则直接返回空.

/mobile/user/cronredis
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当前活跃用户数：activeDeviceCount.json
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
