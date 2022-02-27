# 一、Kafka概述

官网：https://kafka.apache.org



> 官方首页描述：
>
> ​		Apache Kafka is an open-source distributed event streaming platform used by thousands of companies for high-performance data pipelines, streaming analytics, data integration, and mission-critical applications.
>
> 翻译：Apache Kafka是一个开源分布式事件流平台，被上千家公司使用的高性能管道、流分析、数据集成和关键任务应用程序。

# 二、Kafka搭建

## 1.kafka配置

## 2.群起集群

```shell
#!/bin/bash
if [ $# -lt 1 ]
then
	echo "No Input Args..."
	exit;
fi

case $1 in
"start")
	for i in hadoop102 hadoop103 hadoop104
	do
		echo "----------------- kafka $i 启动 -----------------"
		ssh $i "/opt/install/kafka/bin/kafka-server-start.sh -daemon /opt/install/kafka/config/server.properties"
	done
;;
"stop")
	for i in hadoop102 hadoop103 hadoop104
	do
		echo "----------------- kafka $i 关闭 -----------------"
		ssh $i "/opt/install/kafka/bin/kafka-server-stop.sh"
	done
;;
*)
	echo "Input Args Error..."
;;
esac
```

