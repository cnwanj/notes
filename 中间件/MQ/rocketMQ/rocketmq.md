# RocketMQ

参考链接：[http://rocketmq.apache.org/docs/quick-start/](http://rocketmq.apache.org/docs/quick-start/)

## 一、概念和特性

### 1.概念：

### 2.特性：

## 二、架构设计

### 1.架构

### 2.设计

 ![img](https://github.com/apache/rocketmq/raw/master/docs/cn/image/rocketmq_design_1.png) 

#### 主要流程：

- 消息生产者producer发送的消息包括主题Topic、队列编号QueueId、消息Message。
- 将生产的消息写入到CommitLog日志文件中。
- 消费队列ConsumeQueue保存了指定的Topic下的消费消息在CommitLog的起始物理偏移量offset、消息大小msgSize、消息标签Tag的HashCode；说白了就是保存了topic订阅消息的索引，提高查找效率。
- 消费者consumer从消费队列中获取对应的消息。

#### 消息存储整体架构：

- CommitLog：消息主体，存储producer端生产的消息主题内容，单个文件存储大小默认为1G，第一个文件写满了就会写入到下一个文件；在存储过程中同时记录主题内容的偏移量。

- ConsumeQueue：消息消费队列，目的是提高消息消费效率；存储了topic订阅在commitLog日志文件的起始物理偏移量offset、消息大小msgSize、消息标签tagCode（HashCode），consumeQueue文件可以看成是基于topic的commitLog的索引文件indexFile。

- indexFile：索引文件，提供了可以通过key或时间区来查找消息的方法，单个indexFile文件固定大小为400M，一个indexFile可以保存2000W个索引，底层设计中实现了HashMap结构，也就是使用了Hash索引。