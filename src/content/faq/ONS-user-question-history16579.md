---
id: "question-history-16579"
title: "rocketmq默认端口有哪些？"
date: "2024-09-26"
category: "expertConsultation"
description: "根据我了解的信息中提供的信息，RocketMQ默认使用了几个关键端口，这些端口号及其用途如下：1. **8080** - 这个端口主要用于RocketMQ Dashboard的访问。如果需要修改此端口，可以通过调整`application.yml`配置文件来实现。更多详情可参考[这里](https:"
tags: ["rocketmq","默认端口","列表","问题","查询","消息队列"]
keywords: ["rocketmq","默认端口","列表","问题","查询","消息队列"]
---

## 用户问题 ： 
 rocketmq默认端口有哪些？  

---------------
## 专家官方解答 ：

根据我了解的信息中提供的信息，RocketMQ默认使用了几个关键端口，这些端口号及其用途如下：

1. **8080** - 这个端口主要用于RocketMQ Dashboard的访问。如果需要修改此端口，可以通过调整`application.yml`配置文件来实现。更多详情可参考[这里](https://github.com/apache/rocketmq-dashboard/blob/master/src/main/resources/application.yml)。

2. **8081** - 在RocketMQ Proxy组件中，这是gRPC协议访问时使用的默认端口。若需自定义该端口，则应编辑`conf/rmq-proxy.json`配置文件，并指定新的端口号。

3. **10911** - RocketMQ Broker的主要监听端口，默认情况下设置为10911。要更改这个值，可以在Broker配置文件里调整`listenPort`参数。

4. **10912** - 作为haListenPort存在，它用于Master Broker与Slave Broker之间的主备通信。同样地，通过修改Broker配置文件中的`haListenPort`可以设定不同的端口值。

5. **10909** - 默认情况下代表VIPChannel所使用的端口，通常等于`listenPort-2`（即当`listenPort`为10911时，VIPChannel端口就是10909）。VIPChannel设计目的是为了在高负载条件下提高网络连接效率。此端口也可通过适当修改配置文件进行调整。

综上所述，RocketMQ涉及到多个服务组件，每个都有其特定的默认端口设置。对于大多数场景来说，上述列出的端口已经足够满足日常需求；但如果您有特殊要求或安全考虑，可以根据实际需要对它们进行相应的调整。


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
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17260)给我们反馈。
