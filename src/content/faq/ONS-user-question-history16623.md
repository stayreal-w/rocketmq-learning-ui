---
id: "question-history-16623"
title: "tools.sh CODE: 1 DESC: org.apache.rocketmq.acl.common.AclExce"
date: "2024-09-26"
category: "expertConsultation"
description: "根据您提供的我了解的信息，在使用`tools.sh`脚本调用quickStart下的Producer时遇到`CODE: 1 DESC: org.apache.rocketmq.acl.common.AclException: No accessKey is configured`错误，是因为`too"
tags: ["rocketmq","toolssh","错误码","1","描述","AclException","访问密钥"]
keywords: ["rocketmq","toolssh","错误码","1","描述","AclException","访问密钥"]
---

## 用户问题 ： 
 tools.sh CODE: 1 DESC: org.apache.rocketmq.acl.common.AclException: No accessKey is configured 。 如何处理？  

---------------
## 专家官方解答 ：

根据您提供的我了解的信息，在使用`tools.sh`脚本调用quickStart下的Producer时遇到`CODE: 1 DESC: org.apache.rocketmq.acl.common.AclException: No accessKey is configured`错误，是因为`tools.sh`不支持直接通过参数或环境变量指定AccessKey和SecretKey。

### 分析问题的原因
- `tools.sh`脚本本身并不支持直接传递AccessKey和SecretKey作为参数。
- 当RocketMQ启用了ACL（访问控制列表）功能后，发送消息等操作需要提供有效的AccessKey和SecretKey来验证身份。如果未正确配置这些信息，则会抛出`AclException: No accessKey is configured`异常。

### 解决方案
建议改用benchmark包中的`producer.sh`脚本来执行您的测试，并且可以通过该脚本的特定参数来设置所需的认证信息。具体步骤如下：

1. **切换到benchmark目录**：
   - 确保你已经位于RocketMQ安装目录下，然后进入`examples/benchmark`子目录。
   ```bash
   cd $ROCKETMQ_HOME/examples/benchmark
   ```

2. **运行producer.sh并指定必要的参数**：
   - 使用`-a`参数启用ACL，同时利用`-ak`和`-sk`分别指定AccessKey和SecretKey。
   - 假设你的NameServer地址为`127.0.0.1:9876`，并且你想使用的AccessKey是`your_access_key`，SecretKey是`your_secret_key`，那么命令可以这样写：
   ```bash
   sh producer.sh -n 127.0.0.1:9876 -t your_topic_name -a -ak your_access_key -sk your_secret_key
   ```

### 对上述步骤的解释
- `-n`：指定NameServer的服务地址，这是RocketMQ集群中一个非常重要的组件，用于管理Topic路由信息。
- `-t`：指定要发送消息的目标Topic名称。
- `-a`：开启ACL认证。
- `-ak` 和 `-sk`：分别为AccessKey和SecretKey，用来进行身份验证。确保这两个值与你在Broker上配置的一致。

通过这种方式，您可以成功地绕过原始`tools.sh`脚本对于ACL认证的支持限制，从而顺利完成消息发送测试。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：tools.sh使用说明 
 
 * 专家经验：Admin Tool 
 
 * 专家经验：rocketmq 5.0 ACL相关问题 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17272)给我们反馈。
