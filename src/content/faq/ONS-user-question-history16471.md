---
id: "question-history-16471"
title: "linux rocketmq 部署安装运行"
date: "2024-08-07"
category: "expertConsultation"
description: "内容要有条理，分步骤：**步骤一：理解需求与环境**您希望在Linux系统上部署安装并运行Apache RocketMQ。根据您的需求，我们将依据提供的知识内容来指导您完成这项任务。**步骤二：检查系统要求**确保您的Linux系统满足以下条件：- 64位操作系统- 安装了64位 JDK 1.8+ "
---

## 用户问题 ： 
 linux rocketmq 部署安装运行  

---------------
## 专家官方解答 ：

内容要有条理，分步骤：

**步骤一：理解需求与环境**
您希望在Linux系统上部署安装并运行Apache RocketMQ。根据您的需求，我们将依据提供的知识内容来指导您完成这项任务。

**步骤二：检查系统要求**
确保您的Linux系统满足以下条件：
- 64位操作系统
- 安装了64位 JDK 1.8+ 

**步骤三：下载RocketMQ**
1. 访问[Apache RocketMQ 5.3.0源码包下载链接](https://dist.apache.org/repos/dist/release/rocketmq/5.3.0/rocketmq-all-5.3.0-source-release.zip)或选择[二进制包下载](https://dist.apache.org/repos/dist/release/rocketmq/5.3.0/rocketmq-all-5.3.0-bin-release.zip)，根据您的偏好决定是否需要编译源码。
2. 如果下载源码包，请按照以下命令解压并编译：
   ```shell
   $ unzip rocketmq-all-5.3.0-source-release.zip
   $ cd rocketmq-all-5.3.0-source-release/
   $ mvn -Prelease-all -DskipTests -Dspotbugs.skip=true clean install -U
   $ cd distribution/target/rocketmq-5.3.0/rocketmq-5.3.0
   ```

**步骤四：启动NameServer**
1. 进入解压后的RocketMQ目录。
2. 使用以下命令启动NameServer：
   ```shell
   $ nohup sh bin/mqnamesrv &
   ```
3. 验证NameServer是否启动成功：
   ```shell
   $ tail -f ~/logs/rocketmqlogs/namesrv.log
   ```
   查找"The Name Server boot success..."日志信息。

**步骤五：启动Broker与Proxy（以Local模式为例）**
1. 启动Broker，并启用Proxy功能：
   ```shell
   $ nohup sh bin/mqbroker -n localhost:9876 --enable-proxy &
   ```
2. 验证Broker是否启动成功，监控proxy.log日志：
   ```shell
   $ tail -f ~/logs/rocketmqlogs/proxy.log
   ```
   寻找"The broker[brokerName,ip:port] boot success..."日志信息。

**步骤六：配置客户端并测试消息收发**
1. 设置环境变量，指向NameServer地址：
   ```shell
   $ export NAMESRV_ADDR=localhost:9876
   ```
2. 使用RocketMQ自带的工具测试消息发送与接收：
   ```shell
   $ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
   $ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
   ```
3. （可选）如果您计划使用Java SDK开发，需在项目中添加RocketMQ客户端依赖，并参考官方文档进行Topic创建及消息收发编程。

**解释说明：**
以上步骤详细介绍了在Linux系统上部署RocketMQ的过程，包括从下载、编译（如需）、启动关键服务，到测试消息收发的整个流程。每一步都是基于官方推荐的操作，确保了部署的正确性和高可用性。请注意，实际操作中应根据系统实际情况调整命令行路径和参数。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：本地部署RocketMQ 5.0 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16474)给我们反馈。
