---
id: "question-history-16454"
title: "rocketmq 发送消息样例"
date: "2024-09-26"
category: "expertConsultation"
description: "根据提供的我了解的信息，我将给出RocketMQ发送消息的几种样例。这些样例涵盖了同步消息、异步消息和单向消息的发送方式。### 1. 同步发送消息**原因分析：**在需要确保消息发送成功并立即获得反馈的应用场景中，使用同步发送是最合适的。这种方式下，生产者会阻塞等待服务器返回响应，直到确认消息已被"
tags: ["rocketmq","发送","消息","样例"]
keywords: ["rocketmq","发送","消息","样例"]
---

## 用户问题 ： 
 rocketmq 发送消息样例  

---------------
## 专家官方解答 ：

根据提供的我了解的信息，我将给出RocketMQ发送消息的几种样例。这些样例涵盖了同步消息、异步消息和单向消息的发送方式。

### 1. 同步发送消息

**原因分析：**
在需要确保消息发送成功并立即获得反馈的应用场景中，使用同步发送是最合适的。这种方式下，生产者会阻塞等待服务器返回响应，直到确认消息已被Broker接收或遇到错误。

**具体步骤：**

1. 创建一个`DefaultMQProducer`实例，并设置其组名。
2. 设置NameServer地址。
3. 启动生产者。
4. 构造一条或多条消息，指定Topic、Tag以及消息内容。
5. 调用`send`方法发送消息，并处理返回的结果。
6. 发送完成后关闭生产者。

**代码示例：**

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;

public class SyncProducer {
    public static void main(String[] args) throws Exception {
        // 1. 创建生产者实例
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
        
        // 2. 设置NameServer地址
        producer.setNamesrvAddr("localhost:9876");
        
        // 3. 启动生产者
        producer.start();
        
        for (int i = 0; i < 100; i++) {
            // 4. 创建消息
            Message msg = new Message("TopicTest", "TagA", ("Hello RocketMQ " + i).getBytes());
            
            // 5. 发送消息
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        
        // 6. 关闭生产者
        producer.shutdown();
    }
}
```

### 2. 异步发送消息

**原因分析：**
当应用程序不能容忍长时间等待Broker响应时，可以采用异步发送模式。这样可以在发送消息后立即返回，而不会阻塞当前线程。通过回调函数来处理发送结果。

**具体步骤：**

- 按照上述同步发送的前两步创建并配置好生产者。
- 在调用`send`方法时传入一个实现了`SendCallback`接口的对象作为参数，该对象定义了如何处理发送成功的回调(`onSuccess`)及失败的回调(`onException`)。
- 继续执行后续逻辑而不必等待消息发送完成的通知。

**代码示例：**

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

public class AsyncProducer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();
        
        int messageCount = 100;
        for (int i = 0; i < messageCount; i++) {
            final int index = i;
            Message msg = new Message("TopicTest", "TagA", ("Hello world " + i).getBytes());
            
            producer.send(msg, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.printf("%-10d OK %s %n", index, sendResult.getMsgId());
                }

                @Override
                public void onException(Throwable e) {
                    System.out.printf("%-10d Exception %s %n", index, e);
                    e.printStackTrace();
                }
            });
        }

        // 等待所有消息被处理
        Thread.sleep(5000);
        producer.shutdown();
    }
}
```

### 3. 单向发送消息

**原因分析：**
对于那些对可靠性要求不高但追求极致性能的应用场景，可以选择单向发送模式。在这种模式下，消息一旦发送出去就不再关心其状态，也不会收到任何确认信息，从而实现最低延迟。

**具体步骤：**

- 生产者的创建与启动过程与之前相同。
- 使用`sendOneway`方法代替`send`来发送消息。

**代码示例：**

```java
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;

public class OnewayProducer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();

        for (int i = 0; i < 100; i++) {
            Message msg = new Message("TopicTest", "TagA", ("Hello RocketMQ " + i).getBytes());
            producer.sendOneway(msg);
        }

        producer.shutdown();
    }
}
```

以上就是基于RocketMQ的消息发送基本示例。每种模式都有其适用场景，请根据实际需求选择合适的方式。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq  样例 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17236)给我们反馈。
