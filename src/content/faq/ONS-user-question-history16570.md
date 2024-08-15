---
id: "question-history-16570"
title: "rocketmq 消息类型有哪些"
date: "2024-08-09"
category: "expertConsultation"
description: "RocketMQ 支持以下几种消息类型：1. **Normal (普通消息)**：这是最常见的一种消息类型，用于一般的消息传递需求，不保证消息的顺序性。2. **FIFO (顺序消息)**：保证同一主题下，按照消息发送的顺序进行消费。适用于需要严格消息顺序的场景，比如交易系统中的订单处理。3. **"
---

## 用户问题 ： 
 rocketmq 消息类型有哪些  

---------------
## 专家官方解答 ：

RocketMQ 支持以下几种消息类型：

1. **Normal (普通消息)**：这是最常见的一种消息类型，用于一般的消息传递需求，不保证消息的顺序性。

2. **FIFO (顺序消息)**：保证同一主题下，按照消息发送的顺序进行消费。适用于需要严格消息顺序的场景，比如交易系统中的订单处理。

3. **Transaction (事务消息)**：事务消息用于实现分布式事务的最终一致，它允许消息发送方在消息发送和业务操作之间保持原子性。RocketMQ 提供了半事务消息和事务消息回查机制来确保事务的完整性。

4. **Delay (定时/延时消息)**：允许消息发送者设置一个延迟时间，消息不会立即被消费，而是在指定的时间后才变为可消费状态。适用于如订单超时取消、定时提醒等场景。

这些消息类型的信息可以在 RocketMQ 的监控指标中通过 `message_type` 这一标签来区分和统计，例如在监控指标详情中，可以看到每种消息类型的生产与消费情况。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq Metrics 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16573)给我们反馈。
