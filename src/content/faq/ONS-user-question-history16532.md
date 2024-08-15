---
id: "question-history-16532"
title: "rocketmq 会占用哪些接口？"
date: "2024-08-08"
category: "expertConsultation"
description: "RocketMQ在运行过程中会占用以下接口（端口）：1. **Nameserver**：默认端口为`9876`，负责管理Broker信息及 Topic 路由信息，是Producer和Consumer发现Broker的入口。2. **Broker服务端口**：默认端口为`10911`，用于非TLS/S"
---

## 用户问题 ： 
 rocketmq 会占用哪些接口？  

---------------
## 专家官方解答 ：

RocketMQ在运行过程中会占用以下接口（端口）：

1. **Nameserver**：默认端口为`9876`，负责管理Broker信息及 Topic 路由信息，是Producer和Consumer发现Broker的入口。

2. **Broker服务端口**：默认端口为`10911`，用于非TLS/SSL的普通通信，处理消息的生产和消费请求。

3. **Broker FastRemoting端口**：默认情况下为`10911+2`，即`10913`，用于快速远程通信，提高通信效率，但具体端口可能会根据配置有所不同。

4. **Broker TLS/SSL端口**：当Broker配置启用`useTLS`为`true`时，会开启一个用于支持TLS/SSL加密传输的端口，通常是`10909`，提供安全的管理功能和网络通信。

5. **Proxy端口**：在RocketMQ 5.0及以上版本中，引入了Proxy模块，默认端口为`8081`，作为轻量级无状态代理，处理客户端协议适配、权限管理等。此端口可配置，若被占用需进行调整。

请注意，以上端口均为RocketMQ默认配置下的端口，实际部署时可能根据需要进行调整。确保您的防火墙或网络安全策略中开放了这些必要的端口，以便RocketMQ能够正常工作。

另外，关于RocketMQ的版本选择，推荐使用5.x系列，尤其是考虑到与JDK17和Spring Boot 3的兼容性，5.x版本提供了云原生化的基础架构、轻量API和多语言SDK支持，以及对事件和流处理场景的良好集成。您可以通过访问[RocketMQ下载页面](https://rocketmq.apache.org/zh/download)来获取最新稳定版的RocketMQ以及对应的gRPC多语言SDK。对于需要使用RocketMQ 4.0的用户，该页面同样提供了RocketMQ Remoting Client的下载选项。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：RocketMQ版本 
 
 * 专家经验：RocketMQ你好 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16566)给我们反馈。
