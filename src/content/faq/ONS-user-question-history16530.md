---
id: "question-history-16530"
title: "rocketmq 端口有哪些？"
date: "2024-09-26"
category: "expertConsultation"
description: "根据提供的我了解的信息，RocketMQ涉及的主要端口及其功能如下：1. **8080端口**：这是RocketMQ Proxy中默认的remoting协议访问端口（`remotingListenPort`）。此外，在RocketMQ Dashboard中也作为默认访问端口。如果需要更改这个端口号，"
tags: ["rocketmq","端口","列表","介绍","使用","配置","注意事项"]
keywords: ["rocketmq","端口","列表","介绍","使用","配置","注意事项"]
---

## 用户问题 ： 
 rocketmq 端口有哪些？  

---------------
## 专家官方解答 ：

根据提供的我了解的信息，RocketMQ涉及的主要端口及其功能如下：

1. **8080端口**：这是RocketMQ Proxy中默认的remoting协议访问端口（`remotingListenPort`）。此外，在RocketMQ Dashboard中也作为默认访问端口。如果需要更改这个端口号，对于Proxy来说，可以通过修改配置文件`conf/rmq-proxy.json`来实现；而对于Dashboard，则是通过调整`application.yml`文件中的相关设置完成，参考链接为[这里](https://github.com/apache/rocketmq-dashboard/blob/master/src/main/resources/application.yml)。

2. **8081端口**：在RocketMQ Proxy里，该端口被指定为gRPC协议访问端口(`grpcServerPort`)。同样地，若需自定义此端口号，应编辑`rmq-proxy.json`配置文件。

3. **10911端口**：这是RocketMQ Broker组件默认使用的监听端口(`listenPort`)。要修改这一设定值，应该直接在Broker的配置文件内调整`listenPort`参数。

4. **10912端口**：被称为`haListenPort`，专门用于Master Broker与Slave Broker之间进行主备切换时的数据同步。如需改变此端口，同样需要在Broker的配置文件中寻找并更新`haListenPort`项。

5. **10909端口**：一般情况下，它代表了Broker VIPChannel所用到的端口，默认计算方式为`listenPort - 2`（因此通常是10909）。VIPChannel是一种特别设计的网络通道，旨在优化高负载环境下的性能表现。如果想要变更这个特定端口，可以考虑调整`listenPort`或直接在配置中指定新的VIPChannel端口。

综上所述，上述端口各自服务于RocketMQ不同组件之间的通信需求，并且大部分都可以通过修改相应的配置文件来自定义其实际使用值。希望这能帮助您更好地理解RocketMQ系统架构中的端口分配情况。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq端口介绍 
 
 * 专家经验：rocketmq 5.0 ACL相关问题 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17251)给我们反馈。
