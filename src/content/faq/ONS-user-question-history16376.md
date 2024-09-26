---
id: "question-history-16376"
title: "集群情况下，push模式，偶现拉取消息到队列不及时的排查思路"
date: "2024-09-26"
category: "expertConsultation"
description: "根据提供的我了解的信息，在RocketMQ集群情况下，如果采用push模式并且偶尔出现拉取消息到队列不及时的问题，可以按照以下步骤进行排查：### 1. 使用`ConsumerProgress`命令查看消费进度首先，利用`ConsumerProgress`命令来检查整个消费组的消费进度。特别注意使用"
tags: ["rocketmq","集群","push模式","拉取消息","不及时","排查思路"]
keywords: ["rocketmq","集群","push模式","拉取消息","不及时","排查思路"]
---

## 用户问题 ： 
 集群情况下，push模式，偶现拉取消息到队列不及时的排查思路  

---------------
## 专家官方解答 ：

根据提供的我了解的信息，在RocketMQ集群情况下，如果采用push模式并且偶尔出现拉取消息到队列不及时的问题，可以按照以下步骤进行排查：

### 1. 使用`ConsumerProgress`命令查看消费进度
首先，利用`ConsumerProgress`命令来检查整个消费组的消费进度。特别注意使用`-s`参数以获取消费端队列负载均衡的情况及每个客户端的堆积情况。

```shell
sh bin/mqadmin consumerProgress -n 192.168.1.x:9876 -g ConsumerGroupName [-s]
```

### 2. 分析`consumerProgress`输出
- 如果`BrokerOffset`持续增长而消费者已经下线，则表明发送端持续发送消息但消费端已停止消费。
- 若消费者在线且多次执行`consumerProgress`后发现相关队列中的`ConsumerOffset`和`LastTime`字段未更新，这可能意味着消费卡住了。此时需要区分普通消息与顺序消息分别处理：
  - **对于顺序消息**：由于为了保证顺序性，除非超过最大重试次数，否则会在本地不断重试。因此需检查消息是否因失败或超时未能成功消费。可以通过查询应用机器上的`rocketmq_client.log`日志文件，并使用`queryMsgByOffset`工具定位具体问题消息。
  - **对于普通消息**：即使消费失败也会向服务端重发并移动位点。但如果大量消息都超时（默认超时时间为15分钟），仍会出现类似卡住的现象。此时同样建议检查日志以确定原因。

### 3. 检查RETRY Topic中是否有大量堆积
若发现普通队列中堆积不多，但RETRY Topic下的队列中有大量堆积，则说明消费端存在较多消息消费失败的情况。可进一步通过如下命令查看审计日志确认最近一段时间内消费失败最频繁的Top 10 ConsumerGroup：
```shell
grep "`date --date="1 minutes ago" +"%Y-%m-%d %H:%M"`" ~/logs/rocketmqlogs/stats.log | grep SNDBCK_PUT_NUMS |sort -nk12 | tail -n 10
```
同时也可以检查死信队列(DLQ)中是否存在消息：
```shell
sh  bin/mqadmin topicStatus -n 192.168.1.x:9876 -t %DLQ%Consumer_Group_Name
```
解决方法主要是修复导致消费失败的具体业务逻辑问题。

### 4. 确认是否为个别消费者慢
如果`consumerProgress`显示堆积集中在某些特定消费者上，而其他消费者正常，则可能是这些消费者的处理能力不足。这时应该去具体的消费者应用上检查资源使用情况、打印jstack等信息，看是否存在full gc或者资源紧张等问题，并据此调整。

### 5. 整体消费速度慢
当所有消费者的消费进度都比较缓慢时，应考虑整体性能瓶颈所在。可以通过审计日志或`statsAll`命令确认生产和消费的速度差异。此外，连续三次对消费者进程执行`jstack`操作可以帮助识别是否有耗时操作影响了消费效率。如果确实是因为客户端消费能力不足，在资源允许的情况下增加消费线程数或扩容消费组内的消费者数量可能会有所帮助。

以上步骤旨在帮助您定位并解决问题根源，从而提高RocketMQ集群环境下Push模式的消息消费效率。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：消息堆积的排查思路 
 
 * 专家经验：rocketMQ的基本概念 
 
 * 专家经验：rocketmq  特性(features) 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17221)给我们反馈。
