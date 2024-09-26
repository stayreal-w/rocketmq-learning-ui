---
id: "question-history-16455"
title: "java rocketmq 使用方法"
date: "2024-09-26"
category: "expertConsultation"
description: "根据我了解的信息中的第1篇参考内容，我们可以了解到使用Java进行RocketMQ消息发送和接收的基本方法。以下是详细的步骤说明：### 1. 配置文件修改首先，需要在`application.properties`中配置RocketMQ的NameServer地址以及生产者组名：```propert"
tags: ["rocketmq","java","使用","方法"]
keywords: ["rocketmq","java","使用","方法"]
---

## 用户问题 ： 
 java rocketmq 使用方法  

---------------
## 专家官方解答 ：

根据我了解的信息中的第1篇参考内容，我们可以了解到使用Java进行RocketMQ消息发送和接收的基本方法。以下是详细的步骤说明：

### 1. 配置文件修改
首先，需要在`application.properties`中配置RocketMQ的NameServer地址以及生产者组名：
```properties
rocketmq.name-server=127.0.0.1:9876
rocketmq.producer.group=my-group
```

### 2. 发送消息
接下来，在Spring Boot应用中通过`RocketMQTemplate`来发送不同类型的消息。

- **同步发送**：直接调用`convertAndSend`方法。
- **发送Spring消息**：使用`MessageBuilder`构建消息对象后发送。
- **异步发送**：通过提供`SendCallback`回调接口处理发送结果。
- **顺序发送**：指定特定的hashkey确保消息按序到达。

示例代码如下：
```java
@SpringBootApplication
public class ProducerApplication implements CommandLineRunner {
    @Resource
    private RocketMQTemplate rocketMQTemplate;

    public static void main(String[] args) {
        SpringApplication.run(ProducerApplication.class, args);
    }

    public void run(String... args) throws Exception {
        // 同步发送
        rocketMQTemplate.convertAndSend("test-topic-1", "Hello, World!");
        
        // 发送Spring消息
        rocketMQTemplate.send("test-topic-1", MessageBuilder.withPayload("Hello, World! I'm from spring message").build());
        
        // 异步发送
        rocketMQTemplate.asyncSend("test-topic-2", new OrderPaidEvent("T_001", new BigDecimal("88.00")), new SendCallback() {
            @Override
            public void onSuccess(SendResult var1) {
                System.out.printf("async onSucess SendResult=%s %n", var1);
            }

            @Override
            public void onException(Throwable var1) {
                System.out.printf("async onException Throwable=%s %n", var1);
            }
        });
        
        // 顺序发送
        rocketMQTemplate.syncSendOrderly("orderly_topic", MessageBuilder.withPayload("Hello, World").build(), "hashkey");
    }

    @Data
    @AllArgsConstructor
    public class OrderPaidEvent implements Serializable {
        private String orderId;
        private BigDecimal paidMoney;
    }
}
```

### 3. 接收消息
对于消费端，同样基于Spring Boot框架，定义一个实现了`RocketMQListener<String>`接口的类作为消息监听器，并通过注解`@RocketMQMessageListener`指定监听的主题与消费者组。

示例代码如下：
```java
@SpringBootApplication
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

    @Slf4j
    @Service
    @RocketMQMessageListener(topic = "test-topic-1", consumerGroup = "my-consumer_test-topic-1")
    public class MyConsumer1 implements RocketMQListener<String> {
        public void onMessage(String message) {
            log.info("received message: {}", message);
        }
    }

    @Slf4j
    @Service
    @RocketMQMessageListener(topic = "test-topic-2", consumerGroup = "my-consumer_test-topic-2")
    public class MyConsumer2 implements RocketMQListener<OrderPaidEvent> {
        public void onMessage(OrderPaidEvent orderPaidEvent) {
            log.info("received orderPaidEvent: {}", orderPaidEvent);
        }
    }
}
```

### 解释
上述步骤展示了如何利用Spring Boot集成RocketMQ来实现消息的发送与接收功能。通过配置文件设置必要的连接信息后，可以轻松地创建生产者发送各种类型的消息（包括普通文本、复杂对象等），同时也可以定义消费者来处理这些消息。这种方式不仅简化了开发过程，还提供了丰富的API支持以满足不同场景下的需求。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：收发消息的例子 
 
 * 专家经验：Admin Tool 
 
 * 专家经验：RocketMQ连接报错RemotingConnectException: connect to <ip:port＞解决方法 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17237)给我们反馈。
