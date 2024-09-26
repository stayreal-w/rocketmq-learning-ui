---
id: "question-history-16318"
title: "Broker崩溃以后有什么影响？"
date: "2024-09-26"
category: "expertConsultation"
description: "根据我了解的信息中的信息，RocketMQ Broker崩溃后的影响取决于崩溃的是Master节点还是Slave节点，以及崩溃的Slave节点数量。具体影响如下：### 1. Master节点崩溃- 消息不能再发送到该Broker集群。- 如果有另一个可用的Broker集群，并且主题存在，则仍然可以"
tags: ["Broker","崩溃","影响","RocketMQ","消息系统","稳定性","数据丢失"]
keywords: ["Broker","崩溃","影响","RocketMQ","消息系统","稳定性","数据丢失"]
---

## 用户问题 ： 
 Broker崩溃以后有什么影响？  

---------------
## 专家官方解答 ：

根据我了解的信息中的信息，RocketMQ Broker崩溃后的影响取决于崩溃的是Master节点还是Slave节点，以及崩溃的Slave节点数量。具体影响如下：

### 1. Master节点崩溃
- 消息不能再发送到该Broker集群。
- 如果有另一个可用的Broker集群，并且主题存在，则仍然可以发送消息。
- 消息可以从Slave节点消费。

### 2. 一些Slave节点崩溃
- 只要有另一个工作的Slave节点，就不会影响发送消息。
- 对消费消息也不会产生影响，除非消费者组设置为优先从该Slave消费。默认情况下，消费者组从Master消费。

### 3. 所有Slave节点崩溃
- 向Master发送消息不会有任何影响。
- 但如果Master是SYNC_MASTER，Producer会得到一个SLAVE_NOT_AVAILABLE错误，表示消息没有发送给任何Slave。
- 对消费消息也没有影响，除非消费者组设置为优先从Slave消费。默认情况下，消费者组从Master消费。

#### 详细原理
有关RocketMQ 5.0版本中高可用设计的具体细节，可以参考官方文档《解读 RocketMQ 5.0 全新的高可用设计》： [https://rocketmq-learning.com/learning/rocketmq-new/](https://rocketmq-learning.com/learning/rocketmq-new/)

综上所述，RocketMQ通过主从架构来保证系统的高可用性。当Master节点或部分Slave节点发生故障时，系统能够通过其他节点继续提供服务。如果所有Slave节点都崩溃，那么虽然不影响向Master发送消息，但可能会导致某些特定配置下的消费问题。因此，在实际部署中，建议合理配置主从节点以确保系统的稳定性和可靠性。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：Broker崩溃以后有什么影响？ 
 
 * 专家经验：rocketmq  经常被问到的问题 
 
 * 专家经验：broker负载怎么看 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17195)给我们反馈。
