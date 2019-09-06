

# 1 概述

## 1.1 定义

- 定义：一个高可用、高可靠的分布式的海量日志采集、聚合和传输，基于流式框架

- 为什么要使用Flume?

  - 使用场景：

    ~~~mermaid
  graph LR
    pyhon[Python爬虫数据]-->diskFolder[服务器本地磁盘文件夹];
    diskFolder-->flume[Flume];
    java[Java后台日志数据]-->diskFolder;
    flume-->hdfs[HDFS];
    flume-->kafka[Kafka];
    portData[网络端口数据]-->flume;
    
    ~~~
  
  - 作用：实时读取服务器本地磁盘的数据，将数据写入到HDFS

## 1.2 组成构架

![](/Users/wuxinhong/Documents/study-notes/images/Flume/Flume组成架构图.png)

### 1.2.1 Agent

​	Agent是一个JVM进程，它以事件的形式将数据从源头送到目的，是Flume传输的基本单位

​	Agent主要有三个组成部分：Source、Channel、Sink

### 1.2.2 Source

​	Source是负责接收数据到Flume Agent的组件，Source组件可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy

### 1.2.3 Channel

​	Channel是位于Source和Sink之间的缓冲区。因此，Channel允许Source和Sink动作在不同的速率上。Channel是线程安全的，可以同时处理几个Source的写入操作和几个Sink的读取操作

​	Flume自带两个Channel：Memory Channel和File Channel

​	Memory Channel是内存中的队列，Memory Channel在不需要关心数据丢失的情景下适用。如果需要关心数据丢失，那么Memory Channel就不应该使用，因为程序死亡、机器宕机或者重启都会导致数据丢失

​	File Channel将所有事件写到磁盘。因此在程序关闭或机器宕机的情况下不会丢失数据

### 1.2.4 Sink

​	Sink不断地轮询Channel中的事件且批量地移除它们，并将这些事件批量写入存储或索引系统、或者被发送到另一个Flume Agent.

​	Sink是完全事务性的。在从Channel批量删除数据之前，每个Sink用Channel启动一个事务。批量事件一旦成功写出到存储系统或下一个Flume Agent，Sink就利用Channel提交事务。事务一旦被提交，该Channel从自己的内部缓冲区删除事件。

​	Sink组件目的地包括：hdfs、logger、avro、thrift、ipc、file、null、HBase、solr、自定义。

### 1.2.5 Event

​	传输单元，Flume数据传输的基本单元，以事件的形式将数据从源头送至目的地。

## 1.3 Flume拓扑结构（https://www.bilibili.com/video/av35106806/?p=4）

## 1.4 Flume Agent内部原理

~~~mermaid
graph TB
	start(开始)--1.接收事件-->source[Source];
	source--2.处理事件-->channel[Channel处理器];
	channel-->interupt1[拦截器1];
	subgraph 
		interupt1-->interupt2[拦截器2];
		interupt2-->interupt3[拦截器3];
		end
interupt3-->channel;
channel--4.将每个事件传给Channel选择器-->channelSelector[Channel选择器];
channelSelector--5.返回写入事件Channel列表-->channel
channel-->channel1[Channel1];
channel--6.根据Channel选择器的选择结果,将事件写入相应Channel-->channel2[Channel2];
channel-->channel3[Channel3];
channel1-.->sink[Sink处理器];
channel2-.->sink[Sink处理器];
channel3-.->sink[Sink处理器];
sink-.->sink1[Sink1];
sink-.->sink2[Sink2];
sink-.->sink3[Sink3];
sink1-->e[结束];
sink2--7.Sink处理器选择其中一个Sink去获取Channel数据,并将获取数据写入下一个阶段-->e[结束];
sink3-->e[结束];

~~~

- Channel Selectors有两种类型：
  - Replicating Channel Selector（default）：会将source传过来的events发往所有channel
  - Multiplexing Channel Selector：可以配置发往哪些Channel

# 3 快速入门

## 3.1 安装地址

- 官网地址：https://flume.apache.org

- 下载地址：http://archive.apache.org/dist/flume/1.8.0/

## 3.2 安装步骤

1. 将apache-flume-1.8.0-bin.tar.gz上传到linux的/opt/software目录下

2. 解压apache-flume-1.8.0-bin.tar.gz到opt/module/目录下

   ~~~
   tar -zxf apache-flume-1.8.0-bin.tar.gz -C /opt/module/
   ~~~

3. 复制conf下的flume-env.sh.template为flume-env.sh，并配置flume-env.sh文件

   ~~~
   export JAVE_HOME=/opt/module/jdk1.8.0_144
   ~~~

   