---
id: "question-history-16746"
title: "docker 安装 RocketMQ快速入门，安装 配置"
date: "2024-09-05"
category: "expertConsultation"
description: "要进行RocketMQ的快速入门，包括Docker安装与配置，可以按照以下步骤操作：### 1. 准备环境与拉取镜像确保您的系统满足以下要求：- **64位操作系统**- **64位 JDK 1.8+** 已安装并配置好环境变量接着，从Docker Hub上拉取RocketMQ的镜像，这里以5.3."
---

## 用户问题 ： 
 docker 安装 RocketMQ快速入门，安装 配置  

---------------
## 专家官方解答 ：

要进行RocketMQ的快速入门，包括Docker安装与配置，可以按照以下步骤操作：

### 1. 准备环境与拉取镜像
确保您的系统满足以下要求：
- **64位操作系统**
- **64位 JDK 1.8+** 已安装并配置好环境变量

接着，从Docker Hub上拉取RocketMQ的镜像，这里以5.3.0版本为例：
```shell
docker pull apache/rocketmq:5.3.0
```

### 2. 创建Docker网络
为了便于RocketMQ的各个组件之间相互通信，我们需要创建一个Docker网络：
```shell
docker network create rocketmq
```

### 3. 启动NameServer
NameServer是RocketMQ集群中的命名服务，用于管理Broker的注册信息：
```shell
docker run -d --name rmqnamesrv -p 9876:9876 --network rocketmq apache/rocketmq:5.3.0 sh mqnamesrv
```
您可以通过检查NameServer的日志确认其是否成功启动：
```shell
docker logs -f rmqnamesrv
```

### 4. 启动Broker
Broker负责接收、存储和转发消息，同时我们将启用Proxy功能以便更灵活地处理消息：
```shell
# 配置Broker的IP地址（此处以Linux环境为例）
echo "brokerIP1=$(hostname -I | cut -d' ' -f1)" > broker.conf

docker run -d \
--name rmqbroker \
--network rocketmq \
-p 10912:10912 -p 10911:10911 -p 10909:10909 \
-p 8080:8080 -p 8081:8081 \
-e "NAMESRV_ADDR=rmqnamesrv:9876" \
-v $(pwd)/broker.conf:/home/rocketmq/rocketmq-5.3.0/conf/broker.conf \
apache/rocketmq:5.3.0 sh mqbroker --enable-proxy \
-c /home/rocketmq/rocketmq-5.3.0/conf/broker.conf

# 验证Broker是否启动成功
docker exec -it rmqbroker bash -c "tail -n 10 /home/rocketmq/logs/rocketmqlogs/proxy.log"
```

### 5. （可选）使用RocketMQ Dashboard进行管理
为了更直观地管理和监控RocketMQ集群，您可以部署RocketMQ Dashboard。以下是使用Docker镜像安装Dashboard的步骤：
```shell
docker pull apacherocketmq/rocketmq-dashboard:latest
docker run -d --name rocketmq-dashboard -e "JAVA_OPTS=-Drocketmq.namesrv.addr=127.0.0.1:9876" -p 8080:8080 -t apacherocketmq/rocketmq-dashboard:latest
```
之后，您可以通过访问`http://localhost:8080`来使用Dashboard界面进行Topic创建、消息发送与监控等操作。

### 6. 测试与验证
使用Java SDK或其他语言的SDK进行消息的生产和消费，以验证RocketMQ集群是否工作正常。具体步骤可参照[rocketmq-clients](https://github.com/apache/rocketmq-clients)项目中的文档和示例。

### 7. 停止容器
实验或使用完毕后，记得停止并清理Docker容器：
```shell
docker stop rmqnamesrv
docker stop rmqbroker
```

以上步骤完成了RocketMQ的Docker安装与基本配置，您可以基于此环境进行更深入的学习与应用开发。如需了解更多高级配置或遇到问题，请参考[Apache RocketMQ官方文档](https://rocketmq.apache.org/docs/)。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：Docker 部署 RocketMQ 5.x 
 
 * 专家经验：RocketMQ Dashboard 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16766)给我们反馈。
