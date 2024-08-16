---
id: "question-history-16579"
title: "rocketmq默认端口有哪些？"
date: "2024-08-09"
category: "expertConsultation"
description: "RocketMQ的默认端口有以下几个：1. **RocketMQ Proxy**:   - **remoting协议访问端口**: 默认为8080。   - **gRPC协议访问端口**: 默认为8081。   可以通过修改`conf/rmq-proxy.json`配置文件来自定义这些端口。2. *"
---

## 用户问题 ： 
 rocketmq默认端口有哪些？  

---------------
## 专家官方解答 ：

RocketMQ的默认端口有以下几个：

1. **RocketMQ Proxy**:
   - **remoting协议访问端口**: 默认为8080。
   - **gRPC协议访问端口**: 默认为8081。
   可以通过修改`conf/rmq-proxy.json`配置文件来自定义这些端口。

2. **RocketMQ Dashboard**:
   - 访问端口默认为8080。
   端口可以通过修改Spring Boot应用的`application.yml`配置文件来自定义，参考链接为:[https://github.com/apache/rocketmq-dashboard/blob/master/src/main/resources/application.yml](https://github.com/apache/rocketmq-dashboard/blob/master/src/main/resources/application.yml)。

3. **RocketMQ Broker**:
   - **默认访问端口**: 10911。
   - **haListenPort** (主备交互端口): 默认为10912。
   - **VIPChannel使用端口**: 默认情况下是`listenPort-2`，即默认值为10909。
   这些端口可以通过修改Broker的配置文件中的相应参数进行调整。

综上所述，RocketMQ的主要默认端口包括8080、8081用于Proxy服务，8080用于Dashboard，以及Broker服务的10911、10912和默认的VIPChannel端口10909。根据实际部署需求，这些端口都可以进行相应的配置更改。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq端口介绍 
 
 * 专家经验：日志的相关说明 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16582)给我们反馈。
