.. _installing_architecture:

===============
Architecture
===============


系统架构
--------------
.. image:: ../images/architecture.png

流程图
--------------

系统流程图
^^^^^^^^^^^^^^
.. graphviz::

    digraph G {

		node [margin=0.2, fontsize=16, shape=box, style="rounded,filled", fillcolor=white]
		a [label="MobileAgent", shape="ellipse", margin=0.1]
		d [label="DataCollector", fillcolor=yellow]
		con [label="Consumer", fillcolor=yellow]
		dv [label="DasWeb", fillcolor=yellow]
		k [label="Kafka/Zookeeper"]
		r [label="Redis"]
		ms [label="MetricStore", fillcolor=deepskyblue1]
		m [label="mysql", shape=circle, fillcolor=deepskyblue1]
		c [label="ClickHouse", fontsize=12, shape=circle, fillcolor=deepskyblue1, margin=0.1]

		a -> d -> {k,r}

		{k,r} -> con -> {m,k}

		k -> ms -> c

		{r, m, ms} -> dv
	}


流程图详解
^^^^^^^^^^^^^^
.. uml:: 
   
    @startuml
	start
	:移动端探针上报数据;
	partition DataCollector {
		:【DataCollector】;
		if (验证数据合法性) then (true)
		    :发送数据到消息中间件 【Kafka】;    
		else
			:数据直接丢弃;
		endif
	}
	partition DataConsumer {
		:【DataConsumer】 数据处理模块从【kafka】拉取数据处理;
	    if (元数据,Trace数据/App系能数据) then (元数据,Trace数据)
	       :直接写入【Mysql】数据库;
	    else
	        :格式化后回放到【Kafka】;
	        :【MetricStore】处理和消费性能数据;
	        :数据入【ClickHouse】;
	    endif
		
	}
	partition DasWeb {
		:从redis、mysql,MetricStore 查询数据;
	    :页面展示数据;
	}
	stop
	@enduml
