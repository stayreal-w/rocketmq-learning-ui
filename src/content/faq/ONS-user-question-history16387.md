---
id: "question-history-16387"
title: "队列粒度负载均衡怎么做？"
date: "2024-09-26"
category: "expertConsultation"
description: "### 问题分析RocketMQ中的队列粒度负载均衡策略主要用于历史版本（服务端4.x/3.x版本）的消费者，包括PullConsumer、DefaultPushConsumer、DefaultPullConsumer、LitePullConsumer等。在这种策略下，同一消费者分组内的多个消费者将"
tags: ["rocketmq","队列","粒度","负载均衡","实现","方法"]
keywords: ["rocketmq","队列","粒度","负载均衡","实现","方法"]
---

## 用户问题 ： 
 队列粒度负载均衡怎么做？  

---------------
## 专家官方解答 ：

### 问题分析
RocketMQ中的队列粒度负载均衡策略主要用于历史版本（服务端4.x/3.x版本）的消费者，包括PullConsumer、DefaultPushConsumer、DefaultPullConsumer、LitePullConsumer等。在这种策略下，同一消费者分组内的多个消费者将按照队列粒度消费消息，即每个队列仅被一个消费者消费。

### 具体步骤
根据我了解的信息中提供的信息，队列粒度负载均衡的具体实现步骤如下：

1. **启动Consumer**：首先启动RocketMQ Consumer实例，确保它属于特定的消费者分组。
2. **心跳包发送**：Consumer启动后，会通过定时任务不断地向RocketMQ集群中的所有Broker实例发送心跳包。心跳包中包含了消息消费分组名称、订阅关系集合、消息通信模式和客户端ID等信息。
3. **维护元数据**：Broker在接收到Consumer的心跳消息后，会将其维护在`ConsumerManager`的本地缓存变量`consumerTable`中，并将封装后的客户端网络通道信息保存在`channelInfoTable`中，为之后做负载均衡提供依据。
4. **启动负载均衡服务线程**：在Consumer实例的启动流程中，会完成负载均衡服务线程`RebalanceService`的启动，该线程每隔20秒执行一次。
5. **调用`rebalanceByTopic`方法**：`RebalanceService`线程的`run()`方法最终调用的是`RebalanceImpl`类的`rebalanceByTopic()`方法。该方法是实现Consumer端负载均衡的核心。
6. **获取消息消费队列集合**：从`RebalanceImpl`实例的本地缓存变量`topicSubscribeInfoTable`中，获取该Topic主题下的消息消费队列集合（`mqSet`）。
7. **获取消费者Id列表**：根据Topic和消费者组作为参数调用`mQClientFactory.findConsumerIdList()`方法向Broker端发送RPC请求，获取该消费组下的消费者Id列表。
8. **计算待拉取的消息队列**：对Topic下的消息消费队列和消费者Id进行排序，然后使用消息队列分配策略算法（默认为平均分配算法）计算出待拉取的消息队列。
9. **更新`processQueueTable`**：调用`updateProcessQueueTableInRebalance()`方法，将分配到的消息队列集合与`processQueueTable`做一个过滤比对。
   - 对于互不包含的队列，设置`Dropped`属性为`true`，并尝试移除这些队列。
   - 对于交集部分，判断是否已经过期，如果是Push模式，设置`Dropped`属性为`true`，并尝试移除这些队列。
10. **创建`ProcessQueue`对象**：为过滤后的消息队列集合中的每个`MessageQueue`创建一个`ProcessQueue`对象，并存入`RebalanceImpl`的`processQueueTable`队列中。
11. **创建拉取请求**：调用`computePullFromWhere(MessageQueue mq)`方法获取`MessageQueue`对象的下一个进度消费值`offset`，随后填充至`pullRequest`对象属性中，并将`pullRequest`添加到拉取列表`pullRequestList`中。
12. **发起Pull消息请求**：执行`dispatchPullRequest()`方法，将Pull消息的请求对象`PullRequest`依次放入`PullMessageService`服务线程的阻塞队列`pullRequestQueue`中，待该服务线程取出后向Broker端发起Pull消息的请求。

### 解释
- **心跳包发送**：心跳包用于Broker端维护消费者的状态信息，确保Broker能够及时了解消费者的在线情况。
- **元数据维护**：`consumerTable`和`channelInfoTable`用于存储消费者的信息，为负载均衡提供必要的数据支持。
- **负载均衡服务线程**：`RebalanceService`定期执行负载均衡操作，确保消息队列的分配始终是最新的。
- **消息队列分配**：通过平均分配算法，确保每个消费者都能均匀地分配到消息队列，避免某些消费者处理过多或过少的消息。
- **`processQueueTable`更新**：通过过滤和更新`processQueueTable`，确保每个消费者只处理分配给它的消息队列，避免重复消费或漏消费。
- **创建拉取请求**：通过创建`ProcessQueue`对象和`PullRequest`，确保每个消费者能够正确地从Broker拉取消息并进行消费。

以上步骤详细描述了RocketMQ中队列粒度负载均衡的实现过程，确保了消息在消费者之间的均匀分配。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq   设计(design) 
 
 * 专家经验：消费者负载均衡 5.x 
 
 * 专家经验：broker负载怎么看 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17225)给我们反馈。
