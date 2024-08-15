---
id: "question-history-16595"
title: "架构介绍"
date: "2024-08-09"
category: "expertConsultation"
description: "内容：RocketMQ的架构设计基于高可用、高性能和可扩展性的原则，旨在满足各种消息传递场景的需求，尤其是在金融级业务中对可靠性和低延迟的严格要求。下面是对RocketMQ架构的一个概括性介绍：1. **NameServer集群**：RocketMQ采用去中心化的设计理念，NameServer作为其"
---

## 用户问题 ： 
 架构介绍  

---------------
## 专家官方解答 ：

内容：

RocketMQ的架构设计基于高可用、高性能和可扩展性的原则，旨在满足各种消息传递场景的需求，尤其是在金融级业务中对可靠性和低延迟的严格要求。下面是对RocketMQ架构的一个概括性介绍：

1. **NameServer集群**：RocketMQ采用去中心化的设计理念，NameServer作为其核心的元数据管理组件，负责存储和提供Broker的路由信息，以及Broker的存活状态。客户端在发送或消费消息前会先从NameServer获取Broker的信息列表。NameServer之间不进行数据同步，每个NameServer都是独立的，客户端通常会连接多个NameServer以提高可用性。

2. **Broker集群**：Broker是RocketMQ中实际负责消息存储与转发的角色，分为Master Broker和Slave Broker（或称为Replica）。Master Broker负责接收生产者发送的消息并提供给消费者消费，同时将消息数据同步到与其配对的Slave Broker以确保高可用。在Master Broker故障时，Slave Broker可以自动晋升为新的Master，保证消息服务的连续性。

3. **Producer与Consumer**：
   - **Producer**：消息生产者，负责将消息发送到Broker。支持多种消息模式，如点对点（Queue）和发布/订阅（Topic）模型。RocketMQ保证同一Topic内消息的顺序性。
   - **Consumer**：消息消费者，分为广播消费和集群消费两种模式。广播消费模式下，每个消息都会被集群内的所有消费者接收；集群消费则会将消息负载均衡地分发给不同的消费者实例。

4. **存储机制**：RocketMQ采用高性能的文件存储方式，保证了消息的持久化和高吞吐量。消息在Broker端被组织成Commit Log、Consume Queue等多种结构，优化了读写性能和消息查找效率。

5. **高可用与容灾**：RocketMQ通过Master-Slave架构实现数据冗余，配合NameServer的动态发现机制，可以实现自动故障转移。此外，还支持分布式事务消息，确保消息在分布式系统中的最终一致性。

6. **消息过滤**：RocketMQ支持基于标签（Tag）的消息过滤，消费者可以在订阅时指定标签表达式， Broker端会根据这些规则过滤消息，减少不必要的网络传输和消费端处理。

综上所述，RocketMQ的架构设计充分考虑了消息中间件的核心需求，通过一系列机制保障了消息的高效、可靠传递，适合大规模分布式系统的消息处理场景。

为了帮助您更深入地了解RocketMQ，推荐访问其官方网站[[https://rocketmq.apache.org/zh/](https://rocketmq.apache.org/zh/)] 和中文学习社区[[https://rocketmq-learning.com](https://rocketmq-learning.com)] 获取更多详细资料和实践案例。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq中文学习社区介绍 
 
 * 专家经验：RocketMQ 介绍 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16602)给我们反馈。
