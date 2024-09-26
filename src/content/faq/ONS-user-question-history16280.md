---
id: "question-history-16280"
title: "消息在服务器上可以保存多长时间？"
date: "2024-09-26"
category: "expertConsultation"
description: "根据提供的我了解的信息，RocketMQ 消息在服务器上的默认保存时间是 3 天。超过 3 天未被使用的消息将会被删除。### 详细信息- **默认保存时长**：存储的消息最多保存 3 天。- **配置修改**：可以通过调整 `fileReservedTime` 参数来改变消息的保留时间。这个参数以"
tags: ["rocketmq","消息","保存","服务器","时间"]
keywords: ["rocketmq","消息","保存","服务器","时间"]
---

## 用户问题 ： 
 消息在服务器上可以保存多长时间？  

---------------
## 专家官方解答 ：

根据提供的我了解的信息，RocketMQ 消息在服务器上的默认保存时间是 3 天。超过 3 天未被使用的消息将会被删除。

### 详细信息
- **默认保存时长**：存储的消息最多保存 3 天。
- **配置修改**：可以通过调整 `fileReservedTime` 参数来改变消息的保留时间。这个参数以小时为单位，默认值为72小时（即3天）。

### 修改方法
如果需要延长或缩短消息在服务器上的保存时间，可以按照以下步骤操作：

1. 找到 RocketMQ 的配置文件，通常位于 Broker 的配置目录下。
2. 在配置文件中找到 `fileReservedTime` 参数。
3. 将该参数设置为您期望的值（以小时计）。例如，若希望消息保留5天，则应将 `fileReservedTime` 设置为 120（5 * 24 = 120小时）。
4. 重启 Broker 使配置生效。

### 配置示例
```properties
# 设置消息保留时间为5天
fileReservedTime=120
```

### 解释
通过上述步骤，您可以自定义消息在服务器上保存的时间长度，这有助于更好地适应不同的业务需求。比如，在某些情况下可能需要更长时间来处理和消费消息，此时适当增加 `fileReservedTime` 可以避免因超期而丢失重要数据。但同时需要注意的是，增加消息保存时间也会相应地增加磁盘空间占用，因此在调整此参数时需综合考虑存储成本与业务需求之间的平衡。

更多关于消息存储机制的信息，请参考[这里](https://rocketmq.apache.org/zh/docs/featureBehavior/11messagestorepolicy)。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq  经常被问到的问题 
 
 * 专家经验：消息存储和清理机制 
 
 * 专家经验：消息在服务器上可以保存多长时间？ 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17186)给我们反馈。
