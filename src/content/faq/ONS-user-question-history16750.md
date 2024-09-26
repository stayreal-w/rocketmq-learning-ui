---
id: "question-history-16750"
title: "顺序消息原理和使用"
date: "2024-09-26"
category: "expertConsultation"
description: "### RocketMQ 顺序消息原理和使用#### 1. 原理分析顺序消息是 Apache RocketMQ 提供的一种高级消息类型，支持消费者按照发送消息的先后顺序获取消息。这种特性在某些业务场景下非常重要，例如撮合交易、数据实时增量同步等，需要保证消息处理的顺序性。- **应用场景**：在有序"
tags: ["rocketmq","顺序消息","原理","使用"]
keywords: ["rocketmq","顺序消息","原理","使用"]
---

## 用户问题 ： 
 顺序消息原理和使用  

---------------
## 专家官方解答 ：

### RocketMQ 顺序消息原理和使用

#### 1. 原理分析
顺序消息是 Apache RocketMQ 提供的一种高级消息类型，支持消费者按照发送消息的先后顺序获取消息。这种特性在某些业务场景下非常重要，例如撮合交易、数据实时增量同步等，需要保证消息处理的顺序性。

- **应用场景**：在有序事件处理、撮合交易（如证券、股票交易）、数据实时增量同步（如数据库变更日志传输）等场景中，顺序消息能够确保消息按照发送顺序被消费，从而保持业务逻辑的一致性和准确性。
  
- **功能原理**：
  - 顺序消息通过消息组（MessageGroup）来判定和识别顺序关系。相同消息组的消息遵循先进先出的原则。
  - 生产顺序性：单个生产者串行地发送消息，并按序存储和持久化。为了保证生产顺序性，必须满足以下条件：
    - 单一生产者：不同生产者之间无法保证消息顺序。
    - 串行发送：多线程并行发送会导致消息顺序混乱。
  - 消费顺序性：消费者和服务端协议保障消息消费严格按照存储顺序处理。为了保证消费顺序性，必须满足以下条件：
    - 投递顺序：消息按照服务端存储顺序一条一条投递给消费者。
    - 有限重试：超过最大重试次数后将不再重试，跳过这条消息消费。

- **生命周期**：
  - 初始化：消息被构建并准备发送。
  - 待消费：消息被发送到服务端，等待消费者消费。
  - 消费中：消息被消费者获取并处理。
  - 消费提交：消费者完成消费并向服务端提交结果。
  - 消息删除：消息从物理文件中删除。

- **使用限制**：顺序消息仅支持使用 `MessageType` 为 `FIFO` 的主题，即顺序消息只能发送至类型为顺序消息的主题中。

#### 2. 使用步骤
根据上述原理，以下是使用 RocketMQ 顺序消息的具体步骤：

1. **创建顺序消息主题**：
   ```shell
   sh mqadmin updateTopic -c DefaultCluster -t FIFOTopic -o true -n 127.0.0.1:9876 -a +message.type=FIFO
   ```

2. **创建顺序订阅消费组**：
   ```shell
   sh mqadmin updateSubGroup -c DefaultCluster -g FIFOGroup -n 127.0.0.1:9876 -o true
   ```

3. **发送顺序消息**：
   在发送顺序消息时，必须设置消息组。以 Java 语言为例，示例代码如下：
   ```java
   // 创建生产者实例
   DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
   producer.setNamesrvAddr("localhost:9876");
   producer.start();

   // 发送顺序消息
   for (int i = 0; i < 100; i++) {
       Message msg = new Message("FIFOTopic", "TagA", ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
       // 设置消息组
       msg.setKeys("MessageGroup1");
       SendResult sendResult = producer.send(msg);
       System.out.printf("%s%n", sendResult);
   }

   // 关闭生产者
   producer.shutdown();
   ```

4. **消费顺序消息**：
   消费者需要串行处理消息，避免批量消费导致乱序。示例代码如下：
   ```java
   // 创建消费者实例
   DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("FIFOGroup");
   consumer.setNamesrvAddr("localhost:9876");
   consumer.subscribe("FIFOTopic", "*");

   // 注册消息监听器
   consumer.registerMessageListener(new MessageListenerOrderly() {
       @Override
       public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
           for (MessageExt msg : msgs) {
               System.out.printf("Receive message: %s, %n", new String(msg.getBody()));
           }
           return ConsumeOrderlyStatus.SUCCESS;
       }
   });

   // 启动消费者
   consumer.start();
   ```

#### 3. 解释
- **创建顺序消息主题**：通过 `mqadmin` 工具创建一个 `MessageType` 为 `FIFO` 的主题，确保该主题支持顺序消息。
- **创建顺序订阅消费组**：通过 `mqadmin` 工具创建一个顺序订阅消费组，确保该消费组支持顺序消息消费。
- **发送顺序消息**：在发送消息时，设置消息组，确保相同消息组的消息按顺序存储在同一队列中。
- **消费顺序消息**：消费者需要串行处理消息，避免批量消费导致乱序。通过 `MessageListenerOrderly` 监听器确保消息按顺序消费。

通过以上步骤，可以实现 RocketMQ 顺序消息的发送和消费，确保消息处理的顺序性。更多详细信息和示例代码可以参考 [RocketMQ 官方文档](https://rocketmq.apache.org/zh/docs/featureBehavior/03fifomessage)。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：顺序消息 
 
 * 专家经验：rocketmq  特性(features) 
 
 * 专家经验：rocketmq  基本概念 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17290)给我们反馈。
