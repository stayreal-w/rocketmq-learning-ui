---
id: "question-history-16595"
title: "架构介绍"
date: "2024-09-26"
category: "expertConsultation"
description: "RocketMQ 的架构设计非常注重高可用性、高性能和可扩展性。下面将基于提供的知识内容，详细介绍 RocketMQ 的技术架构和部署架构。## 1. 技术架构RocketMQ 架构主要分为四个部分：**Producer（生产者）**、**Consumer（消费者）**、**NameServer**"
tags: ["RocketMQ","架构","分布式","消息队列","高可用","事务消息","名称服务"]
keywords: ["RocketMQ","架构","分布式","消息队列","高可用","事务消息","名称服务"]
---

## 用户问题 ： 
 架构介绍  

---------------
## 专家官方解答 ：

RocketMQ 的架构设计非常注重高可用性、高性能和可扩展性。下面将基于提供的知识内容，详细介绍 RocketMQ 的技术架构和部署架构。

## 1. 技术架构
RocketMQ 架构主要分为四个部分：**Producer（生产者）**、**Consumer（消费者）**、**NameServer** 和 **BrokerServer**。

- **Producer**：负责消息的发布，支持分布式集群方式部署。通过负载均衡模块选择相应的 Broker 集群队列进行消息投递。
- **Consumer**：负责消息消费，同样支持分布式集群方式部署。支持推（push）和拉（pull）两种模式的消息消费，并且可以以集群或广播的方式消费消息。
- **NameServer**：作为 Topic 路由注册中心，类似于 Dubbo 中的 Zookeeper。它接受 Broker 的注册信息并保存下来作为路由信息的基础数据，同时提供心跳检测机制检查 Broker 是否存活。NameServer 通常也是集群方式部署，每个实例间不直接通信，但都保存一份完整的路由信息。
- **BrokerServer**：是 RocketMQ 的核心服务端组件，主要职责包括消息存储、消息投递、查询以及服务的高可用保证。其内部包含几个重要子模块：
  - **Remoting Module**：处理来自客户端的请求。
  - **Client Manager**：管理 Producer/Consumer 客户端及其订阅信息。
  - **Store Service**：提供 API 接口用于消息存储到物理硬盘及查询功能。
  - **HA Service**：实现 Master Broker 与 Slave Broker 之间的数据同步。
  - **Index Service**：根据特定 Message key 对消息进行索引以便快速查询。

## 2. 部署架构

### 2.1 单Master模式
这是最简单的部署方式，但风险较高。一旦 Broker 服务器宕机，整个服务将不可用。仅适用于开发测试环境。

启动步骤：
1. 启动 NameServer。
   ```shell
   $ nohup sh mqnamesrv &
   ```
2. 验证 NameServer 是否启动成功。
   ```shell
   $ tail -f ~/logs/rocketmqlogs/namesrv.log
   The Name Server boot success...
   ```
3. 启动 Broker。
   ```shell
   $ nohup sh bin/mqbroker -n localhost:9876 &
   ```
4. 验证 Broker 是否启动成功。
   ```shell
   $ tail -f ~/logs/rocketmqlogs/Broker.log
   The broker[broker-a,192.168.1.2:10911] boot success...
   ```

### 2.2 多Master模式
所有节点都是 Master 主节点，没有 Slave 从节点。这种模式配置简单，性能最高，但是单台机器宕机时会影响未消费消息的实时性。

启动步骤与单Master模式类似，只是需要在多台机器上分别启动多个 Broker 实例。

### 2.3 多Master多Slave模式-异步复制
每个 Master 节点配置一个或多个 Slave 节点。主从之间采用异步复制，具有较高的性能和可靠性。

启动步骤：
1. 启动 NameServer。
2. 在不同机器上依次启动多个 Master 和对应的 Slave Broker 实例。

### 2.4 多Master多Slave模式-同步双写
此模式下，只有当消息成功写入到 Master 并被所有指定的 Slave 成功复制后，才会向应用返回成功响应。适合对数据一致性要求极高的场景。

启动步骤同异步复制模式，区别在于配置文件中需要设置相关参数来启用同步复制。

### 2.5 自动主从切换
自 RocketMQ 5.0 版本开始引入了自动主从切换功能，提高了系统的可用性和稳定性。更多关于自动主从切换的信息，请参考官方文档中的[快速开始](controller/quick_start.md)和[部署文档](controller/deploy.md)。

通过上述介绍可以看出，RocketMQ 提供了多种灵活的部署方案以满足不同的业务需求。对于具体的选择，可以根据实际的应用场景和对性能、可靠性的要求来决定使用哪种部署模式。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq  部署架构和设置步骤 
 
 * 专家经验：rocketmq  架构设计 
 
 * 专家经验：RocketMQ 介绍 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17270)给我们反馈。
