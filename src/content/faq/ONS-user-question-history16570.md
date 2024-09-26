---
id: "question-history-16570"
title: "rocketmq 消息类型有哪些"
date: "2024-09-26"
category: "expertConsultation"
description: "根据我了解的信息中的信息，Apache RocketMQ 支持的消息类型有以下几种：1. **普通消息**：这是最基础的消息类型，用于一般的业务场景。2. **顺序消息**：这类消息保证了消息的消费顺序与发送顺序一致。顺序消息又分为两种：   - 全局顺序消息：对于指定的一个Topic，所有消息按照"
tags: ["rocketmq","消息类型"]
keywords: ["rocketmq","消息类型"]
---

## 用户问题 ： 
 rocketmq 消息类型有哪些  

---------------
## 专家官方解答 ：

根据我了解的信息中的信息，Apache RocketMQ 支持的消息类型有以下几种：

1. **普通消息**：这是最基础的消息类型，用于一般的业务场景。
2. **顺序消息**：这类消息保证了消息的消费顺序与发送顺序一致。顺序消息又分为两种：
   - 全局顺序消息：对于指定的一个Topic，所有消息按照严格的先入先出（FIFO）的顺序进行发布和消费。
   - 分区顺序消息：对于指定的一个Topic，所有消息根据sharding key进行区块分区。同一个分区内的消息按照严格的FIFO顺序进行发布和消费。
3. **事务消息**：这种高级消息类型支持在分布式场景下保障消息生产和本地事务的最终一致性。通过RocketMQ的事务消息功能可以实现类似XA的分布事务效果。
4. **定时/延时消息**：这类消息被发送至服务端后，并不会立即被消费者消费，而是等待到特定的时间点才开始投递。这使得开发者能够基于时间触发某些事件或流程。

每种消息类型都有其适用的具体业务场景，选择合适的消息类型可以帮助更好地解决实际问题。例如，在需要确保操作顺序的应用中使用顺序消息；在需要跨服务间保持数据一致性的场景下采用事务消息等。此外，从5.0版本开始，RocketMQ还引入了对消息类型的强制校验机制，即每个主题只允许发送一种类型的消息，以此来简化运维管理并避免潜在的问题。但同时，该特性也向下兼容4.x版本的行为，默认情况下是开启状态。

更多关于这些消息类型的详细说明及其配置方法，请参阅官方文档：
- [主题（Topic）](https://rocketmq.apache.org/zh/docs/domainModel/02topic)
- [队列（MessageQueue）](https://rocketmq.apache.org/zh/docs/domainModel/03messagequeue)
- [消息（Message）](https://rocketmq.apache.org/zh/docs/domainModel/04message)
- [事务消息](https://rocketmq.apache.org/zh/docs/featureBehavior/04transactionmessage)


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketMQ的基本概念 
 
 * 专家经验：rocketmq  特性(features) 
 
 * 专家经验：rocketmq  经常被问到的问题 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17255)给我们反馈。
