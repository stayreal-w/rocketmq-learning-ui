---
id: "question-history-16366"
title: "broker负载怎么看"
date: "2024-09-26"
category: "expertConsultation"
description: "### 问题：RocketMQ Broker负载怎么看#### 分析原因要查看RocketMQ Broker的负载情况，可以利用多种命令行工具和监控手段。这些方法能够帮助我们了解Broker当前的工作状态以及资源使用情况，从而判断是否处于高负载状态。根据我了解的信息中的信息，常用的方法包括`top`"
tags: ["RocketMQ","broker","负载","监控","查看","方法","指标"]
keywords: ["RocketMQ","broker","负载","监控","查看","方法","指标"]
---

## 用户问题 ： 
 broker负载怎么看  

---------------
## 专家官方解答 ：

### 问题：RocketMQ Broker负载怎么看

#### 分析原因
要查看RocketMQ Broker的负载情况，可以利用多种命令行工具和监控手段。这些方法能够帮助我们了解Broker当前的工作状态以及资源使用情况，从而判断是否处于高负载状态。根据我了解的信息中的信息，常用的方法包括`top`、`dstat`、`ioutils`等命令。

当Broker负载较高时，可能会触发一系列流控机制来保护自身不被过载请求所压垮。这种情况下，发送消息到Broker可能会失败，并伴随特定错误提示，如`system busy`或`broker busy`等。这些错误提示具体反映了不同类型的流控措施正在生效，比如PageCache繁忙、线程池拒绝新任务等情况。

#### 具体步骤
1. **使用系统级监控工具检查负载**：
   - 使用`top`命令查看CPU使用率。
   - 使用`dstat`获取更详细的系统性能数据（需要先安装`dstat`）。
   - 利用`iostat`或者`ioutil`分析磁盘I/O状况。

2. **识别并理解流控错误**：
   - 当遇到`[REJECTREQUEST]system busy, start flow control for a while`错误时，表明PageCache可能非常忙碌，或者没有足够的堆外内存块可用。
   - `[OVERLOAD]system busy, start flow control for a while`意味着请求处理线程池已满，无法接受更多任务。
   - `[PC_SYNCHRONIZED]broker busy, start flow control for a while`指出在尝试写入CommitLog之前发现PageCache仍然很忙。
   - `[PCBUSY_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: xxms, size of queue: xx`与快速失败机制有关，在PageCache繁忙且有排队请求时触发。
   - `[TIMEOUT_CLEAN_QUEUE]broker busy, start flow control for a while, period in queue: xxms, size of queue: xx`则是在请求队列等待时间超过设定阈值时发生。

3. **调整相关参数缓解压力（如果必要）**：
   - 增加`sendMessageThreadPoolNums`和`sendThreadPoolQueueCapacity`以扩大处理能力。
   - 调整`waitTimeMillsInSendQueue`允许更长的队列等待时间。
   - 适当增加`osPageCacheBusyTimeOutMills`减少因PageCache繁忙导致的流控。
   - 确认`brokerFastFailureEnable`设置为开启状态，以便于及时清理长时间等待的任务。

4. **考虑横向扩容或升级硬件**：
   - 如果上述优化措施仍不足以解决问题，则需考虑通过添加更多的Broker实例来分散负载，或者直接提高现有服务器的配置规格。

#### 解释
以上步骤首先从基础层面入手，通过常用的Linux命令对Broker所在机器进行全面体检，确保能够准确把握其运行状态。接着针对可能出现的各种流控现象进行了说明，并给出了相应的解决策略。最后提醒用户，在软件层面优化达到极限后，还需要从物理资源的角度出发，从根本上提升系统的承载能力。这样一套流程下来，基本上可以有效地诊断出RocketMQ Broker的负载状况，并采取适当的措施进行优化。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：broker负载怎么看 
 
 * 专家经验：rocketmq  经常被问到的问题 
 
 * 专家经验：rocketmq   设计(design) 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17216)给我们反馈。
