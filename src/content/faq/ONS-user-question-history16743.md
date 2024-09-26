---
id: "question-history-16743"
title: "RocketMQ快速入门使用说明，包括但不限于下载，安装，启动，测试可用，发送接收消息，关闭"
date: "2024-09-26"
category: "expertConsultation"
description: "### RocketMQ 快速入门使用说明RocketMQ 是一个分布式消息中间件，支持发布/订阅模型。本指南将介绍如何下载、安装、启动 RocketMQ，并进行简单的发送和接收消息的测试。#### 1. 下载与安装首先，您需要从官方源下载最新版本的 RocketMQ。根据[官方文档](https:"
tags: ["RocketMQ","快速入门","使用说明","下载","安装","启动","测试"]
keywords: ["RocketMQ","快速入门","使用说明","下载","安装","启动","测试"]
---

## 用户问题 ： 
 RocketMQ快速入门使用说明，包括但不限于下载，安装，启动，测试可用，发送接收消息，关闭  

---------------
## 专家官方解答 ：

### RocketMQ 快速入门使用说明

RocketMQ 是一个分布式消息中间件，支持发布/订阅模型。本指南将介绍如何下载、安装、启动 RocketMQ，并进行简单的发送和接收消息的测试。

#### 1. 下载与安装
首先，您需要从官方源下载最新版本的 RocketMQ。根据[官方文档](https://rocketmq.apache.org/zh/download/)提供的链接，选择合适的版本进行下载。这里以二进制包为例：

- 访问 [RocketMQ 官方下载页面](https://rocketmq.apache.org/zh/download/)
- 选择最新的稳定版（如5.3.0），并下载对应的二进制文件 `rocketmq-all-5.3.0-bin-release.zip`。

解压下载好的压缩包到您的服务器上：
```bash
unzip rocketmq-all-5.3.0-bin-release.zip -d /opt/
```

设置环境变量以便于后续操作：
```bash
export ROCKETMQ_HOME=/opt/rocketmq-all-5.3.0-bin-release
export PATH=$ROCKETMQ_HOME/bin:$PATH
```

#### 2. 启动 NameServer 和 Broker
在开始之前，请确保您的系统满足以下要求：
- Linux/Unix/Mac 操作系统
- 64位 JDK 1.8+
- Maven 3.2.x (如果计划编译源码)

##### 启动 NameServer
打开终端，执行以下命令启动 NameServer：
```bash
nohup sh $ROCKETMQ_HOME/bin/mqnamesrv &
```
这会以后台进程的方式运行 NameServer。可以通过查看日志确认是否成功启动：
```bash
tail -f ~/logs/rocketmqlogs/namesrv.log
```

##### 启动 Broker
接着启动 Broker，同样以 nohup 方式运行：
```bash
nohup sh $ROCKETMQ_HOME/bin/mqbroker -n localhost:9876 &
```
这里的 `-n` 参数指定了 NameServer 的地址。检查 Broker 是否正常工作：
```bash
tail -f ~/logs/rocketmqlogs/broker.log
```

#### 3. 测试可用性
为了验证 RocketMQ 是否可以正常使用，我们将创建一个简单的生产者-消费者示例来发送和接收消息。

##### 生产者代码
新建一个 Java 文件 `Producer.java` 并添加如下内容：
```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;

public class Producer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("test_producer");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();

        Message msg = new Message("TestTopic", "TagA", ("Hello RocketMQ").getBytes());
        producer.send(msg);

        producer.shutdown();
    }
}
```

##### 消费者代码
然后创建另一个 Java 文件 `Consumer.java` 如下所示：
```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class Consumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("test_consumer");
        consumer.setNamesrvAddr("localhost:9876");
        consumer.subscribe("TestTopic", "*");
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                for (MessageExt msg : msgs) {
                    System.out.println("Received message: " + new String(msg.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
    }
}
```

编译并运行上述两个程序，先启动消费者再启动生产者，观察控制台输出以验证消息传递情况。

#### 4. 关闭服务
当不再需要时，可以通过以下命令停止 NameServer 和 Broker：
```bash
sh $ROCKETMQ_HOME/bin/mqshutdown namesrv
sh $ROCKETMQ_HOME/bin/mqshutdown broker
```

以上步骤涵盖了从下载安装到基本使用的全过程，帮助您快速上手 RocketMQ。更多高级功能及配置请参考官方文档。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：RocketMQ Dashboard 
 
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
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17283)给我们反馈。
