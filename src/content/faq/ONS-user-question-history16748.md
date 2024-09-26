---
id: "question-history-16748"
title: "事务消息的原理与使用详解"
date: "2024-09-26"
category: "expertConsultation"
description: "### RocketMQ 事务消息的原理与使用详解#### 原理RocketMQ 提供了一种高级的消息类型——事务消息，它支持在分布式场景下保障消息生产和本地事务的最终一致性。事务消息的设计基于两阶段提交（2PC）的思想，并引入了补偿机制来处理二阶段超时或者失败的情况。##### 事务消息流程概要事"
tags: ["rocketmq","事务消息","原理","使用","详解"]
keywords: ["rocketmq","事务消息","原理","使用","详解"]
---

## 用户问题 ： 
 事务消息的原理与使用详解  

---------------
## 专家官方解答 ：

### RocketMQ 事务消息的原理与使用详解

#### 原理
RocketMQ 提供了一种高级的消息类型——事务消息，它支持在分布式场景下保障消息生产和本地事务的最终一致性。事务消息的设计基于两阶段提交（2PC）的思想，并引入了补偿机制来处理二阶段超时或者失败的情况。

##### 事务消息流程概要
事务消息的发送及提交过程可以分为两个主要阶段：正常事务消息的发送及提交、以及事务消息的补偿流程。

1. **事务消息发送及提交**：
   - 发送消息（half消息），此时消息对业务不可见。
   - 服务端响应消息写入结果。
   - 根据发送结果执行本地事务（如果写入失败，则不执行本地逻辑）。
   - 根据本地事务状态执行Commit或Rollback操作。Commit操作会生成消息索引，使消息对消费者可见；而Rollback则不会。

2. **补偿流程**：
   - 对于没有Commit/Rollback的事务消息（即处于pending状态的消息），服务端会发起一次“回查”请求。
   - 生产者收到回查消息后，检查对应的本地事务状态。
   - 根据检查到的本地事务状态，再次向服务端提交Commit或Rollback操作。

##### 关键设计点
- **Half消息的隐藏性**：通过替换消息的主题和队列属性，将原主题和队列信息存储在消息属性中，确保消息在第一阶段不对用户可见。
- **Op消息**：用于标识事务消息的状态（Commit或Rollback）。如果一条事务消息没有对应的Op消息，则说明这个事务的状态还无法确定。
- **补偿机制**：当服务端未收到生产者的二次确认结果或收到的结果为未知状态时，服务端会定期回查生产者以获取最终的事务状态。

#### 使用详解
使用RocketMQ事务消息需要遵循一定的步骤和注意事项，包括创建特定类型的Topic、配置生产者等。

##### 创建事务消息主题
```shell
./bin/mqadmin updatetopic -n localhost:9876 -t TestTopic -c DefaultCluster -a +message.type=TRANSACTION
```
这里`-a +message.type=TRANSACTION`参数指定了该主题仅支持事务消息。

##### 发送事务消息
在Java应用中使用事务消息时，首先需要初始化一个带有事务监听器的生产者实例，并设置预绑定的事务消息主题列表。
```java
// 初始化生产者
DefaultMQProducer producer = new DefaultMQProducer("TransactionProducerGroup");
producer.setNamesrvAddr("localhost:9876");

// 设置事务监听器
TransactionListener transactionListener = new TransactionListenerImpl();
TransactionMQProducer transactionProducer = (TransactionMQProducer) producer;
transactionProducer.setTransactionListener(transactionListener);

// 预绑定事务消息主题
List<String> tranMsgTopics = new ArrayList<>();
tranMsgTopics.add("TestTopic");
transactionProducer.setTranStateTableColumns(tranMsgTopics);

// 启动生产者
transactionProducer.start();

// 构造并发送事务消息
Message msg = new Message("TestTopic", "TagA", "OrderID001", ("Hello RocketMQ").getBytes(RemotingHelper.DEFAULT_CHARSET));
SendResult sendResult = transactionProducer.sendMessageInTransaction(msg, null);
System.out.printf("SendResult: %s%n", sendResult);
```

##### 注意事项
- 确保事务监听器能够正确处理本地事务逻辑及其状态查询。
- 在消息回查时，对于正在进行中的事务应保持Unknown状态，避免过早返回Commit或Rollback导致数据不一致。
- 调整首次事务回查时间间隔以适应具体业务需求，但需注意这可能会影响依赖回查完成事务的时间。

更多关于如何编写事务监听器以及处理事务消息的具体代码示例，请参考官方文档[此处](https://rocketmq.apache.org/zh/docs/featureBehavior/04transactionmessage)。

以上就是RocketMQ事务消息的基本原理及使用方法介绍。希望这些信息能帮助你更好地理解和运用这一特性。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq   设计(design) 
 
 * 专家经验：事务消息 
 
 * 专家经验：rocketMQ的基本概念 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17288)给我们反馈。
