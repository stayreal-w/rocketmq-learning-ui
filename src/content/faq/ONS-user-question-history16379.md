---
id: "question-history-16379"
title: "麻烦问下 mq5.3.0版本如何开启ACL2.0,在部署的时候需要修改哪些配置文件？"
date: "2024-09-26"
category: "expertConsultation"
description: "要开启RocketMQ 5.3.0版本中的ACL 2.0功能，并完成相应的配置文件修改，可以按照以下步骤进行：### 1. 开启认证功能在Broker端启用认证功能需要对broker的配置文件进行如下设置：```propertiesauthenticationEnabled = trueauthen"
tags: ["rocketmq","mq530","ACL20","开启","配置文件","修改","部署"]
keywords: ["rocketmq","mq530","ACL20","开启","配置文件","修改","部署"]
---

## 用户问题 ： 
 麻烦问下 mq5.3.0版本如何开启ACL2.0,在部署的时候需要修改哪些配置文件？  

---------------
## 专家官方解答 ：

要开启RocketMQ 5.3.0版本中的ACL 2.0功能，并完成相应的配置文件修改，可以按照以下步骤进行：

### 1. 开启认证功能
在Broker端启用认证功能需要对broker的配置文件进行如下设置：
```properties
authenticationEnabled = true
authenticationProvider = org.apache.rocketmq.auth.authentication.provider.DefaultAuthenticationProvider
initAuthenticationUser = {"username":"rocketmq","password":"12345678"}
innerClientAuthenticationCredentials = {"accessKey":"rocketmq","secretKey":"12345678"}
authenticationMetadataProvider = org.apache.rocketmq.auth.authentication.provider.LocalAuthenticationMetadataProvider
```

### 2. 开启授权功能
同样地，在Broker端启用授权功能也需添加或修改相关参数：
```properties
authorizationEnabled = true
authorizationProvider = org.apache.rocketmq.auth.authorization.provider.DefaultAuthorizationProvider
authorizationMetadataProvider = org.apache.rocketmq.auth.authorization.provider.LocalAuthorizationMetadataProvider
```

### 3. 如果是从ACL 1.0迁移到ACL 2.0
如果你之前已经使用了ACL 1.0，并希望迁移至ACL 2.0，请确保在Broker的配置文件中加入如下配置以支持平滑迁移：
```properties
migrateAuthFromV1Enabled = true
```
这将使得系统在启动时自动执行迁移操作，把旧版的用户权限信息转换到新版格式下。

### 4. 修改`plain_acl.yml`文件（可选）
虽然对于ACL 2.0而言，不再强制要求像以前那样直接编辑`plain_acl.yml`文件来定义权限规则，但你仍然可以通过此文件预设一些基础权限设定。不过主要的权限管理现在更多是通过命令行工具`mqadmin`来进行动态调整。

### 5. 使用`mqadmin`命令管理ACL
- 创建、更新、删除用户及查询相关信息。
- 管理用户的访问控制列表(ACL)，包括创建、更新、删除授权以及查看授权详情等操作。

例如，创建一个新的ACL用户可以使用如下命令：
```shell
sh mqadmin createUser -n 127.0.0.1:9876 -c DefaultCluster -u rocketmq -p 12345678
```

### 解释
上述步骤涵盖了从启用认证和授权服务开始，直至如何利用`mqadmin`工具进行日常运维所需的各种操作。特别是对于从较早版本升级上来的用户来说，提供了明确的迁移指南。通过这些配置与命令，你可以有效地管理和保护你的RocketMQ消息队列服务，确保只有被允许的客户端才能访问特定资源。

此外，有关更详细的文档和示例，您可以参考官方提供的学习资料：[RocketMQ ACL 学习](https://rocketmq-learning.com/learning/rocketmq_learning-acl/)。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq 5.0 ACL相关问题 
 
 * 专家经验：rocketmq  权限控制 
 
 * 专家经验：rocketmq ACL mqadmin配置管理命令 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17222)给我们反馈。
