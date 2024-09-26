---
id: "question-history-16439"
title: "rocketmq broker架构与原理"
date: "2024-09-26"
category: "expertConsultation"
description: "RocketMQ 的 Broker 架构与原理涉及多个方面，包括技术架构、部署架构以及关键子模块的功能。下面将根据提供的知识内容进行详细解析。### 1. 技术架构RocketMQ 的技术架构主要分为四个部分：Producer（生产者）、Consumer（消费者）、NameServer（名称服务）和"
tags: ["rocketmq","broker","架构","原理"]
keywords: ["rocketmq","broker","架构","原理"]
---

## 用户问题 ： 
 rocketmq broker架构与原理  

---------------
## 专家官方解答 ：

RocketMQ 的 Broker 架构与原理涉及多个方面，包括技术架构、部署架构以及关键子模块的功能。下面将根据提供的知识内容进行详细解析。

### 1. 技术架构
RocketMQ 的技术架构主要分为四个部分：Producer（生产者）、Consumer（消费者）、NameServer（名称服务）和 BrokerServer（消息服务器）。每个组件的具体职责如下：

- **Producer**：负责发送消息到Broker，支持集群方式部署。
- **Consumer**：负责从Broker接收并消费消息，同样支持集群方式部署，并且提供Push（推）和Pull（拉）两种消费模式。
- **NameServer**：作为Topic路由注册中心，负责Broker的动态注册与发现，保存整个Broker集群的路由信息。
- **BrokerServer**：是RocketMQ的核心组件之一，主要负责消息的存储、投递和查询等操作，同时保证服务高可用。它包含以下几个重要子模块：
  - **Remoting Module**：处理来自客户端的请求。
  - **Client Manager**：管理客户端连接及维护消费者的订阅信息。
  - **Store Service**：处理消息的持久化存储及查询功能。
  - **HA Service**：实现Master Broker与Slave Broker之间的数据同步。
  - **Index Service**：基于特定Message key对消息进行索引以加速查询过程。

### 2. 部署架构
RocketMQ 支持多种部署模式来满足不同的业务需求，主要包括以下几种：

- **单Master模式**：最简单的部署形式，但存在单点故障的风险。适用于开发测试环境。
- **多Master模式**：所有节点都是主节点，没有从节点。此模式下性能最优，但是当某台机器宕机时，其上的未消费消息需等待恢复后才能继续被订阅。
- **多Master多Slave异步复制模式**：每个Master配置有对应的Slave节点，采用异步复制策略。这种设置在保持高性能的同时提供了较好的数据冗余性。
- **多Master多Slave同步双写模式**：与异步复制类似，但这里的消息只有在成功写入所有指定的Master和Slave之后才会返回确认给客户端，从而进一步增强了数据的安全性，但也牺牲了一定程度的性能。

### 3. 关键特性
- **自动主从切换**：自RocketMQ 5.0版本起引入了自动主从切换机制，提高了系统的可靠性和自动化管理水平。更多详情请参阅[快速开始](controller/quick_start.md)、[部署文档](controller/deploy.md)和[设计思想](controller/design.md)。

### 4. 启动步骤示例
以启动一个基本的RocketMQ集群为例，通常需要依次启动NameServer和Broker实例。对于多Master多Slave模式，还需要为每个Broker配置相应的属性文件（如`broker-a.properties`, `broker-b.properties`等），并通过命令行参数指定这些配置文件的位置。

#### 启动NameServer
```shell
$ nohup sh mqnamesrv &
```

#### 启动Broker
假设要启动两个Master和两个Slave，则执行如下命令：
```shell
# 在A机器上启动第一个Master
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-a.properties &

# 在B机器上启动第二个Master
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-b.properties &

# 在C机器上启动第一个Slave
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-a-s.properties &

# 在D机器上启动第二个Slave
$ nohup sh mqbroker -n 192.168.1.1:9876 -c $ROCKETMQ_HOME/conf/2m-2s-async/broker-b-s.properties &
```

### 解释
通过上述步骤，我们构建了一个具有高可用性的RocketMQ集群。其中NameServer充当路由中心的角色，而Broker则承担实际的消息存储和转发任务。通过合理地分配Master和Slave角色，可以有效地分散负载并提高系统的容错能力。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq  部署架构和设置步骤 
 
 * 专家经验：rocketmq  架构设计 
 
 * 专家经验：RocketMQ原理学习 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17229)给我们反馈。
