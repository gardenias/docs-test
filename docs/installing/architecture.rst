.. _installing_architecture:

===============
系统架构
===============

..
 .. image:: ../images/architecture.png

1. 系统总流程图
----------------------------

.. graphviz::

    digraph Application{
      rankdir = BT;

      compound=true;
      ranksep=0.95;

      node [fontsize = 14, shape = "record", fillcolor=yellow, style="filled"];

      subgraph cluster_dc {
         label="数据采集";
         d [label="DataCollector"];
      }

      subgraph cluster_consumer {
         label="数据分析处理";
         con [label="Consumer"];
      }

      subgraph cluster_dv {
         label="数据展示";
         dv [label="DasWeb"];
      }

      subgraph cluster_middware {
         label="Components";
         node [shape="component", fillcolor="lightseagreen", style="filled"];
         redis [label="Redis Cache"];
         db [label="DataBase"];
         metric [label="MetricStore & ClickHouse"];
      }
      subgraph cluster_zk {
        label="Message";
        node [shape="component", fillcolor="lightskyblue", style="filled"];
        zk [label="Kafak & Zookeeper"];
      }

      subgraph cluster_atosl {
         label="符号化组件";
         node [shape=component, fillcolor=".7 .3 1.0", style="filled"];
         atosl [label="Atosl Service"];
      }

      subgraph cluster_users {
         label="客户相关人员";
         node [shape="ellipse", margin=0.1, fillcolor=white];
         others [label="..."];
         manager [label="Manager"];
         developer [label="Developer"];
         it [label="IT Operator"];
      }


      a [label="iOS|Android Agents", shape="ellipse", margin=0.1, fillcolor=white];

      a -> d [lhead=cluster_dc];
      d -> redis [lhead=cluster_middware];
      d -> zk [lhead=cluster_zk];

      atosl -> con [lhead=cluster_consumer,ltail=cluster_atosl];
      con -> zk [dir=both,lhead=cluster_zk,ltail=cluster_consumer]
      con -> db [lhead=cluster_middware, ltail=cluster_consumer];

      atosl -> dv [ltail=cluster_atosl,lhead=cluster_dv];
      db -> dv [lhead=cluster_dv,ltail=cluster_middware];
      dv -> developer [lhead=cluster_users,ltail=cluster_dv];
    }

2. 数据采集模块数据流图
----------------------------
.. graphviz::

    digraph Dc {


		node [fontsize=14, shape=box, style="filled", fillcolor=white]
		a [label="iOS|Android Agents", shape="ellipse", margin=0.1]
		d [label="DataCollector", fillcolor=yellow]
		k [label="Kafka/Zookeeper", shape="component", fillcolor="lightskyblue"]
		r [label="Redis", shape="component", fillcolor="lightseagreen"]
		m [label="mysql", shape="component", fillcolor="lightseagreen"]

		a -> d -> {k,r,m}
    {m, r} -> d
	}

3. 数据处理模块数据流图
----------------------------
.. graphviz::

    digraph Consumer {

  	node [fontsize=14, shape=box, style="filled", fillcolor=white]
  	con [label="Consumer", fillcolor=yellow]
    k [label="Kafka/Zookeeper", shape="component", fillcolor="lightskyblue"]
    r [label="Redis", shape="component", fillcolor="lightseagreen"]
    m [label="mysql", shape="component", fillcolor="lightseagreen"]

  	metric [label="MetricStore & ClickHouse", shape="component", fillcolor="lightseagreen", style="filled"];

    osl [label="Atosl Service", shape=component,style=filled,color=".7 .3 1.0", fillcolor=".7 .3 1.0"]

    {m,r,k,osl} -> con -> {m,k,r}
    k -> metric;
  }


4. 数据展示模块数据流图
----------------------------
.. graphviz::

      digraph DasWeb {
      rankdir = BT;
      compound=true;
      ranksep=0.75;

  		node [fontsize=14, shape=box, style="rounded,filled", fillcolor=white]

  		dv [label="DasWeb", fillcolor=yellow]

      r [label="Redis", shape="component", fillcolor="lightseagreen"]
      m [label="mysql", shape="component", fillcolor="lightseagreen"]
      metric [label="MetricStore & ClickHouse", shape="component", fillcolor="lightseagreen", style="filled"];

      osl [label="Atosl Service", shape=component,style=filled,color=".7 .3 1.0", fillcolor=".7 .3 1.0"]

      subgraph cluster_users {
         label="客户相关人员";
         node [shape="ellipse", margin=0.1, fillcolor=white];
         others [label="..."];
         manager [label="Manager"];
         developer [label="Developer"];
         it [label="IT Operator"];
      }

  		{r, m, metric, osl} -> dv;
      dv -> developer [lhead=cluster_users];
  	}

5. 流程图详解
----------------------------
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
