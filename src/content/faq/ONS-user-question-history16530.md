---
id: "question-history-16530"
title: "rocketmq 端口有哪些？"
date: "2024-08-08"
category: "expertConsultation"
description: "RocketMQ涉及多个组件，每个组件有不同的默认端口，具体如下：1. **RocketMQ Proxy**:   - **remoting协议访问端口** (`remotingListenPort`): 默认为8080。此端口用于Proxy的remoting协议通信。如需修改，可通过编辑`conf"
---

## 用户问题 ： 
 rocketmq 端口有哪些？  

---------------
## 专家官方解答 ：

RocketMQ涉及多个组件，每个组件有不同的默认端口，具体如下：

1. **RocketMQ Proxy**:
   - **remoting协议访问端口** (`remotingListenPort`): 默认为8080。此端口用于Proxy的remoting协议通信。如需修改，可通过编辑`conf/rmq-proxy.json`配置文件实现。
   - **gRPC协议访问端口** (`grpcServerPort`): 默认为8081。用于gRPC协议的访问，同样可在`rmq-proxy.json`中配置自定义端口。

2. **RocketMQ Dashboard**:
   - 访问端口: 默认为8080。Dashboard是RocketMQ的可视化监控管理界面，其端口可修改，操作方式是调整`spring`程序的`application.yml`配置文件，参考链接为[https://github.com/apache/rocketmq-dashboard/blob/master/src/main/resources/application.yml](https://github.com/apache/rocketmq-dashboard/blob/master/src/main/resources/application.yml)。

3. **RocketMQ Broker**:
   - **默认访问端口** (`listenPort`): 通常为10911。Broker作为消息存储和转发的核心服务，此端口用于客户端与Broker的通信。要修改此端口，需编辑Broker的配置文件。
   - **HA监听端口** (`haListenPort`): 默认为10912。此端口用于Master Broker与Slave Broker之间的主备通信，确保高可用性。修改方法同上，在Broker配置文件中调整`haListenPort`。
   - **VIPChannel端口**: 默认情况下，VIPChannel使用的是`listenPort - 2`，即默认为10909。这是一种特殊设计的网络通道，用于优化高负载场景下的通信效率，也是在Broker配置中调整。

综上所述，RocketMQ的关键端口包括8080(Proxy的remoting和Dashboard默认端口)、8081(Proxy的gRPC端口)、10911(Broker默认端口)、10912(Broker的HA端口)，以及基于Broker listenPort计算得出的VIPChannel默认端口10909。这些端口均可根据实际需求在各自的配置文件中进行调整。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq端口介绍 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16531)给我们反馈。
