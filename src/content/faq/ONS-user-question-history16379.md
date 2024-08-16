---
id: "question-history-16379"
title: "麻烦问下 mq5.3.0版本如何开启ACL2.0,在部署的时候需要修改哪些配置文件？"
date: "2024-07-30"
category: "expertConsultation"
description: "为了在mq5.3.0版本中开启ACL2.0，您需要按照以下步骤修改配置文件：1. **认证配置**：   - 首先，确保您的Broker配置中启用了认证功能。这可以通过设置以下参数实现：     ```     authenticationEnabled = true     authenticat"
---

## 用户问题 ： 
 麻烦问下 mq5.3.0版本如何开启ACL2.0,在部署的时候需要修改哪些配置文件？  

---------------
## 专家官方解答 ：

为了在mq5.3.0版本中开启ACL2.0，您需要按照以下步骤修改配置文件：

1. **认证配置**：
   - 首先，确保您的Broker配置中启用了认证功能。这可以通过设置以下参数实现：
     ```
     authenticationEnabled = true
     authenticationProvider = org.apache.rocketmq.auth.authentication.provider.DefaultAuthenticationProvider
     initAuthenticationUser = {"username":"your_username","password":"your_password"}
     innerClientAuthenticationCredentials = {"accessKey":"your_accessKey","secretKey":"your_secretKey"}
     authenticationMetadataProvider = org.apache.rocketmq.auth.authentication.provider.LocalAuthenticationMetadataProvider
     ```
   - 如果您使用的是RocketMQ Proxy，相应的配置应为：
     ```
     {
       "authenticationEnabled": true,
       "authenticationProvider": "org.apache.rocketmq.auth.authentication.provider.DefaultAuthenticationProvider",
       "authenticationMetadataProvider": "org.apache.rocketmq.proxy.auth.ProxyAuthenticationMetadataProvider",
       "innerClientAuthenticationCredentials": "{\"accessKey\":\"your_accessKey\", \"secretKey\":\"your_secretKey\"}"
     }
     ```
   这些设置会启用身份验证并配置默认的身份验证提供程序。

2. **授权配置**：
   - 接着，您需要开启授权功能，添加如下Broker配置：
     ```
     authorizationEnabled = true
     authorizationProvider = org.apache.rocketmq.auth.authorization.provider.DefaultAuthorizationProvider
     authorizationMetadataProvider = org.apache.rocketmq.auth.authorization.provider.LocalAuthorizationMetadataProvider
     ```
   - 对于Proxy，配置如下：
     ```
     {
       "authorizationEnabled": true,
       "authorizationProvider": "org.apache.rocketmq.auth.authorization.provider.DefaultAuthorizationProvider",
       "authorizationMetadataProvider": "org.apache.rocketmq.proxy.auth.ProxyAuthorizationMetadataProvider"
     }
     ```
   这些配置会激活授权控制，允许您对用户访问资源进行细粒度管理。

3. **从ACL 1.0迁移至ACL 2.0**（如适用）：
   - 如果您之前使用的是ACL 1.0，并希望迁移至ACL 2.0，需要在Broker配置中添加：
     ```
     migrateAuthFromV1Enabled = true
     ```
   这将自动迁移旧的权限配置至新系统，但请注意，IP白名单不会被迁移。

请根据您的部署情况（直接使用Broker还是通过Proxy），选择对应的配置进行修改。完成这些配置更改后，重启RocketMQ服务以应用新的ACL设置。

以上步骤基于提供的知识内容进行解答，具体配置细节可能需要根据您的实际环境和需求调整。如果您在实施过程中遇到任何问题，建议查阅RocketMQ官方文档或社区论坛以获取更详尽的帮助。[[了解更多ACL详情]](https://rocketmq-learning.com/learning/rocketmq_learning-acl/)


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq 5.0 ACL相关问题 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16380)给我们反馈。
