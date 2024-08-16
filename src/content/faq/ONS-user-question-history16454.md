---
id: "question-history-16454"
title: "rocketmq 发送消息样例"
date: "2024-08-07"
category: "expertConsultation"
description: "内容如下：### 分析问题原因您想了解关于RocketMQ发送消息的样例。根据提供的我了解的知识，我们可以直接展示如何在Java应用中使用Spring Boot集成RocketMQ发送不同类型的消息。### 具体步骤与示例代码#### 1. 配置RocketMQ连接信息在`application.p"
---

## 用户问题 ： 
 rocketmq 发送消息样例  

---------------
## 专家官方解答 ：

内容如下：

### 分析问题原因
您想了解关于RocketMQ发送消息的样例。根据提供的我了解的知识，我们可以直接展示如何在Java应用中使用Spring Boot集成RocketMQ发送不同类型的消息。

### 具体步骤与示例代码

#### 1. 配置RocketMQ连接信息
在`application.properties`中配置RocketMQ的NameServer地址和生产者组信息：
```properties
rocketmq.name-server=127.0.0.1:9876
rocketmq.producer.group=my-group
```

#### 2. 发送同步消息
使用`RocketMQTemplate`的`convertAndSend`方法发送一个简单的字符串消息到指定主题：
```java
rocketMQTemplate.convertAndSend("test-topic-1", "Hello, World!");
```

#### 3. 发送Spring Message
通过`MessageBuilder`构建消息并发送一个带有自定义payload的对象（例如，普通Java对象）：
```java
rocketMQTemplate.send("test-topic-1", MessageBuilder.withPayload("Hello, World! I'm from spring message").build());
```

#### 4. 异步发送消息
异步发送消息时，需要提供一个`SendCallback`来处理发送结果：
```java
rocketMQTemplate.asyncSend("test-topic-2", new OrderPaidEvent("T_001", new BigDecimal("88.00")), new SendCallback() {
    // 实现onSuccess和onException方法
});
```

#### 5. 顺序发送消息
使用`syncSendOrderly`方法确保消息按照一定的顺序被消费：
```java
rocketMQTemplate.syncSendOrderly("orderly_topic", MessageBuilder.withPayload("Hello, World").build(), "hashkey");
```

### 注意事项
- 在应用关闭前，若不再发送消息，可以调用`rocketMQTemplate.destroy()`来释放资源。
- 确保消费者已经正确配置并运行，以接收这些消息。

### 解释
以上步骤展示了如何在Spring Boot应用中配置RocketMQ并实现不同类型消息的发送，包括同步消息、Spring Message、异步消息以及顺序消息的发送方式。每种发送方式针对不同的业务场景提供了灵活性和效率的选择，比如异步发送适用于对发送响应时间有严格要求的场景，而顺序发送则保证了消息的有序处理，适合对消息处理顺序敏感的应用。

### 前置问题
在开始实施上述步骤前，请确认您已安装并启动了RocketMQ服务，并且知道如何配置消费者以接收这些消息。如果您对如何配置RocketMQ消费者或服务端有疑问，请先解决这些问题。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：收发消息的例子 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16456)给我们反馈。
