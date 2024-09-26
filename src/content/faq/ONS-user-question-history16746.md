---
id: "question-history-16746"
title: "docker 安装 RocketMQ快速入门，安装 配置"
date: "2024-09-26"
category: "expertConsultation"
description: "根据我了解的信息中提供的信息，我们可以使用Docker来快速部署一个单节点单副本的RocketMQ服务。以下是详细的步骤：### 1. 拉取RocketMQ镜像首先需要从Docker Hub上拉取所需的RocketMQ镜像。这里以5.3.0版本为例：```shelldocker pull apach"
tags: ["rocketmq","docker","安装","快速入门","配置"]
keywords: ["rocketmq","docker","安装","快速入门","配置"]
---

## 用户问题 ： 
 docker 安装 RocketMQ快速入门，安装 配置  

---------------
## 专家官方解答 ：

根据我了解的信息中提供的信息，我们可以使用Docker来快速部署一个单节点单副本的RocketMQ服务。以下是详细的步骤：

### 1. 拉取RocketMQ镜像
首先需要从Docker Hub上拉取所需的RocketMQ镜像。这里以5.3.0版本为例：
```shell
docker pull apache/rocketmq:5.3.0
```

### 2. 创建容器共享网络
为了便于RocketMQ中的多个服务（如NameServer, Broker等）之间相互通信，我们需要创建一个Docker网络。
```shell
docker network create rocketmq
```

### 3. 启动NameServer
NameServer是RocketMQ的核心组件之一，用于管理路由信息。接下来启动NameServer容器。
```shell
# 启动 NameServer
docker run -d --name rmqnamesrv -p 9876:9876 --network rocketmq apache/rocketmq:5.3.0 sh mqnamesrv

# 验证 NameServer 是否启动成功
docker logs -f rmqnamesrv
```
如果看到输出 'The Name Server boot success..'，则表示NameServer已成功启动。

### 4. 启动Broker+Proxy
在确保NameServer正常运行后，可以继续启动Broker和Proxy服务。这一步还需要配置Broker的IP地址。
```shell
# 配置 Broker 的IP地址
echo "brokerIP1=127.0.0.1" > broker.conf

# 启动 Broker 和 Proxy
docker run -d \
--name rmqbroker \
--network rocketmq \
-p 10912:10912 -p 10911:10911 -p 10909:10909 \
-p 8080:8080 -p 8081:8081 \
-e "NAMESRV_ADDR=rmqnamesrv:9876" \
-v ./broker.conf:/home/rocketmq/rocketmq-5.3.0/conf/broker.conf \
apache/rocketmq:5.3.0 sh mqbroker --enable-proxy \
-c /home/rocketmq/rocketmq-5.3.0/conf/broker.conf

# 验证 Broker 是否启动成功
docker exec -it rmqbroker bash -c "tail -n 10 /home/rocketmq/logs/rocketmqlogs/proxy.log"
```
若日志显示 'The broker boot success..'，则表明Broker也已成功启动。

至此，您已经完成了一个基本的RocketMQ环境搭建。接下来可以通过编写Java程序等方式进行消息收发测试了。

**解释：**
- **拉取镜像**：这是获取所需软件的第一步，通过指定版本号确保安装的是正确的版本。
- **创建网络**：为容器间通信提供便利，避免因默认桥接网络导致的问题。
- **启动NameServer与Broker**：这两个步骤分别启动了RocketMQ的消息中心及存储服务，其中NameServer负责维护集群元数据，而Broker则是实际处理消息存储与转发的角色。
- **验证启动状态**：每一步操作后都应检查相关服务是否正常启动，保证后续步骤能够顺利进行。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：Docker 部署 RocketMQ 5.x 
 
 * 专家经验：Docker Compose 部署 RocketMQ 5.x 
 
 * 专家经验：RocketMQ Dashboard 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17286)给我们反馈。
