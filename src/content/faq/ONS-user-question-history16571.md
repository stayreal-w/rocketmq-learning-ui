---
id: "question-history-16571"
title: "rocketmq 10911"
date: "2024-09-26"
category: "expertConsultation"
description: "根据提供的我了解的信息，RocketMQ Broker默认使用10911端口进行访问。这个端口号是Broker配置文件中`listenPort`参数的默认值，用于处理来自生产者和消费者的请求。如果需要修改该端口号，可以通过调整Broker配置文件中的相应设置来实现。具体步骤如下：1. 打开Broke"
tags: ["rocketmq","10911"]
keywords: ["rocketmq","10911"]
---

## 用户问题 ： 
 rocketmq 10911  

---------------
## 专家官方解答 ：

根据提供的我了解的信息，RocketMQ Broker默认使用10911端口进行访问。这个端口号是Broker配置文件中`listenPort`参数的默认值，用于处理来自生产者和消费者的请求。如果需要修改该端口号，可以通过调整Broker配置文件中的相应设置来实现。

具体步骤如下：

1. 打开Broker的配置文件（通常位于`conf/broker.conf`）。
2. 查找或添加`listenPort`配置项，并将其设置为您希望使用的端口号。
3. 保存更改并重启Broker服务以应用新的端口设置。

这样做的目的是为了满足特定环境下的需求，比如避免与其他应用程序发生端口冲突，或者遵循某些特定的安全策略要求。

通过上述操作，您可以自定义RocketMQ Broker的服务监听端口，使其更好地适应您的应用场景。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：RocketMQ 版本说明 下载 
 
 * 专家经验：rocketmq端口介绍 
 
 * 专家经验：rocketmq github 地址 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17256)给我们反馈。
