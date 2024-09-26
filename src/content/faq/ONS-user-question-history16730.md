---
id: "question-history-16730"
title: "RocketMQ与Springboot整合（rocketmq-spring-boot-starter）详细步骤"
date: "2024-09-26"
category: "expertConsultation"
description: "根据提供的我了解的信息，要将RocketMQ与Spring Boot整合（使用`rocketmq-spring-boot-starter`），可以遵循以下步骤。这里主要基于Spring Cloud Alibaba RocketMQ Binder的集成方法进行说明。### 1. 添加依赖首先，在您的项"
tags: ["RocketMQ","Springboot","整合","starter","","详细"]
keywords: ["RocketMQ","Springboot","整合","starter","","详细"]
---

## 用户问题 ： 
 RocketMQ与Springboot整合（rocketmq-spring-boot-starter）详细步骤  

---------------
## 专家官方解答 ：

根据提供的我了解的信息，要将RocketMQ与Spring Boot整合（使用`rocketmq-spring-boot-starter`），可以遵循以下步骤。这里主要基于Spring Cloud Alibaba RocketMQ Binder的集成方法进行说明。

### 1. 添加依赖
首先，在您的项目中引入RocketMQ Binder相关的Maven依赖。您可以选择直接添加Binder依赖或者使用Spring Cloud Stream RocketMQ Starter，后者更为简便。推荐使用Starter方式：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
    <!-- 根据需要指定版本号 -->
</dependency>
```

确保您使用的`spring-cloud-starter-stream-rocketmq`版本与您的Spring Boot版本兼容。对于Spring Boot 3.x，建议使用RocketMQ Spring 2.3.0及以上版本。

### 2. 配置RocketMQ Binder属性
接下来，配置RocketMQ Binder的相关属性。这些属性通常在`application.yml`或`application.properties`文件中设置。以下是几个关键配置项示例：

```yaml
spring:
  cloud:
    stream:
      rocketmq:
        binder:
          name-server: localhost:9876 # RocketMQ NameServer地址
          access-key: your-access-key # 如果是阿里云服务，填写AccessKey
          secret-key: your-secret-key # 如果是阿里云服务，填写SecretKey
          enable-msg-trace: true # 是否开启消息轨迹功能，默认为true
```

如果您的应用需要更复杂的消费者配置，比如基于TAGS订阅、广播消费模式等，请参考我了解的信息中的详细配置选项部分，并按照实际需求调整配置。

### 3. 创建消息生产者和消费者
利用Spring Cloud Stream简化消息发送和接收逻辑。下面是一个简单的例子，展示了如何创建一个消息生产者和服务端点来处理接收到的消息：

#### 生产者代码

```java
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@EnableBinding(Source.class)
@RestController
public class MessageProducer {

    private final Source source;

    public MessageProducer(Source source) {
        this.source = source;
    }

    @PostMapping("/send")
    public String sendMessage(@RequestParam("message") String message) {
        source.output().send(MessageBuilder.withPayload(message).build());
        return "Message sent: " + message;
    }
}
```

#### 消费者代码

```java
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.stereotype.Component;

@Component
@EnableBinding(Sink.class)
public class MessageConsumer {

    @StreamListener(Sink.INPUT)
    public void handle(String message) {
        System.out.println("Received: " + message);
    }
}
```

### 解释
上述步骤通过引入适当的依赖、配置必要的参数以及编写简洁的消息发送/接收逻辑，实现了RocketMQ与Spring Boot应用程序的无缝集成。这种方式不仅减少了开发者对底层消息中间件细节的关注，还提供了高度可扩展性和灵活性。特别是利用Spring Cloud Stream框架，使得切换不同的消息队列实现变得异常简单，只需更改相关依赖和少量配置即可。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：Spring Cloud Alibaba 与 RocketMQ 集成 
 
 * 专家经验：RocketMQ 版本说明 下载 
 
 * 专家经验：rocketmq github 地址 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17275)给我们反馈。
