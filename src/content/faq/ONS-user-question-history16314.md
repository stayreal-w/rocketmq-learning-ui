---
id: "question-history-16314"
title: "服务端版本是4.x版本，是否可以使用gRPC SDK ？"
date: "2024-09-26"
category: "expertConsultation"
description: "根据我了解的信息中的信息，RocketMQ 服务端版本为4.x时，是**不支持**使用gRPC SDK的。gRPC SDK仅兼容版本大于等于5.0的服务端。因此，如果您的服务端版本为4.x，并且希望采用gRPC SDK，建议您首先将服务端平滑升级至至少5.0版本后再进行SDK更换。对于更多关于Roc"
tags: ["rocketmq","4x","服务端","gRPC","SDK","兼容性","使用"]
keywords: ["rocketmq","4x","服务端","gRPC","SDK","兼容性","使用"]
---

## 用户问题 ： 
 服务端版本是4.x版本，是否可以使用gRPC SDK ？  

---------------
## 专家官方解答 ：

根据我了解的信息中的信息，RocketMQ 服务端版本为4.x时，是**不支持**使用gRPC SDK的。gRPC SDK仅兼容版本大于等于5.0的服务端。因此，如果您的服务端版本为4.x，并且希望采用gRPC SDK，建议您首先将服务端平滑升级至至少5.0版本后再进行SDK更换。

对于更多关于RocketMQ SDK的信息，您可以参考[这里](https://rocketmq-learning.com/learning/rocketmq-sdk/)获取详细介绍。同时，有关于RocketMQ不同版本的下载链接，请访问[官方下载页面](https://rocketmq.apache.org/zh/download/)以获得最新的版本信息和支持资源。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq sdk 整体说明 
 
 * 专家经验：服务端版本是4.x版本，是否可以使用gRPC SDK ？ 
 
 * 专家经验：从 Remoting SDK 更换到 gRPC SDK 是否需要修改代码？ 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17191)给我们反馈。
