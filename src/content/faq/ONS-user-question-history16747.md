---
id: "question-history-16747"
title: "延迟消息的原理与使用详解"
date: "2024-09-26"
category: "expertConsultation"
description: "### RocketMQ 延迟消息的原理与使用详解#### 1. 原理分析RocketMQ 提供了延迟消息功能，允许用户设置一个特定的时间点或延时后投递消息。这种特性在诸如订单处理、定时任务等场景下非常有用。根据我了解的信息中的信息，延迟消息和定时消息在本质上是相同的，都是基于设定的时间戳来触发消息"
tags: ["rocketmq","延迟消息","原理","使用","详解"]
keywords: ["rocketmq","延迟消息","原理","使用","详解"]
---

## 用户问题 ： 
 延迟消息的原理与使用详解  

---------------
## 专家官方解答 ：

### RocketMQ 延迟消息的原理与使用详解

#### 1. 原理分析
RocketMQ 提供了延迟消息功能，允许用户设置一个特定的时间点或延时后投递消息。这种特性在诸如订单处理、定时任务等场景下非常有用。根据我了解的信息中的信息，延迟消息和定时消息在本质上是相同的，都是基于设定的时间戳来触发消息投递。

- **消息生命周期**：从初始化到最终被消费，延迟消息会经历几个关键阶段，包括初始化（构建消息）、定时中（存储等待触发时刻）、待消费（到达指定时间后重新写入普通队列）、消费中（由消费者处理）以及消费提交（标记为已处理）。如果遇到消费失败，系统还会尝试重试。
- **精度与时长限制**：默认情况下，RocketMQ 的延迟消息支持秒级精度，并且最长可设置24小时内的延时。超过这个范围的消息会被立即投递。此外，对于具体的延时时长，需要通过预设的等级来指定，而不是任意值。
- **实现机制**：服务端接收到带有延迟属性的消息后，并不会立即放入常规的消息队列，而是将其存放在专门的延迟存储区，直到达到设定的时间才释放给消费者。

#### 2. 使用步骤
根据提供的资料，以下是使用 RocketMQ 发送和接收延迟消息的具体步骤：

- **创建主题**：
  - 使用 `mqadmin` 工具更新或创建一个新的主题，并确保该主题支持延迟消息类型。命令示例如下：
    ```shell
    sh mqadmin updateTopic -n <nameserver_address> -t <topic_name> -c <cluster_name> -a +message.type=DELAY
    ```
  - 参数说明：
    - `-n` 指定 NameServer 地址。
    - `-t` 指定 Topic 名称。
    - `-c` 指定集群名称。
    - `-a` 添加额外属性，这里指定了 `+message.type=DELAY` 来启用延迟消息支持。

- **发送延迟消息**：
  - 创建一个 `DefaultMQProducer` 实例并启动它。
  - 构造要发送的消息对象，设置其 `delayTimeLevel` 属性来指定延时时长级别。
  - 调用 `send()` 方法将消息发送出去。
  - 示例代码如下（Java）：
    ```java
    import org.apache.rocketmq.client.producer.DefaultMQProducer;
    import org.apache.rocketmq.common.message.Message;

    public class ScheduledMessageProducer {
        public static void main(String[] args) throws Exception {
            DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
            producer.setNamesrvAddr("localhost:9876");
            producer.start();
            
            Message message = new Message("TestTopic", ("Hello scheduled message").getBytes());
            // 设置延时等级3, 这个消息将在10s之后发送
            message.setDelayTimeLevel(3);
            producer.send(message);

            producer.shutdown();
        }
    }
    ```

- **消费延迟消息**：
  - 启动一个 `DefaultMQPushConsumer` 并订阅相关主题。
  - 注册一个 `MessageListenerConcurrently` 来处理传入的消息。
  - 示例代码如下（Java）：
    ```java
    import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
    import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
    import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
    import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
    import org.apache.rocketmq.common.message.MessageExt;
    import java.util.List;

    public class ScheduledMessageConsumer {
        public static void main(String[] args) throws Exception {
            DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ExampleConsumer");
            consumer.setNamesrvAddr("localhost:9876");
            consumer.subscribe("TestTopic", "*");

            consumer.registerMessageListener(new MessageListenerConcurrently() {
                @Override
                public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                    for (MessageExt msg : msgs) {
                        System.out.println("Receive message[msgId=" + msg.getMsgId() + "] " + (System.currentTimeMillis() - msg.getStoreTimestamp()) + "ms later");
                    }
                    return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                }
            });

            consumer.start();
        }
    }
    ```

#### 3. 解释
以上步骤首先定义了一个可以发送延迟消息的主题，然后通过设置消息的 `delayTimeLevel` 属性来控制其何时被消费者接收到。消费者则持续监听指定主题上的新消息，并在消息到达时执行相应的逻辑。这种方法非常适合于那些需要在未来某个确定时间点执行的操作，比如自动取消未支付订单等。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：定时/延时消息 
 
 * 专家经验：rocketmq  Schedule example 
 
 * 专家经验：rocketmq  样例 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17287)给我们反馈。
