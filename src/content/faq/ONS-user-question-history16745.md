---
id: "question-history-16745"
title: "linux 安装 RocketMQ快速入门，安装 配置 并开机自启 介绍"
date: "2024-09-05"
category: "expertConsultation"
description: "根据您的需求，我将指导您完成RocketMQ在Linux上的安装配置以及如何实现开机自启。首先，让我们依托已有的知识内容来逐步进行。### 1. **下载与安装RocketMQ**按照之前的指引，您需要先下载RocketMQ的二进制包或源码包。对于快速入门，我们推荐直接使用二进制包进行部署。您可以点"
---

## 用户问题 ： 
 linux 安装 RocketMQ快速入门，安装 配置 并开机自启 介绍  

---------------
## 专家官方解答 ：

根据您的需求，我将指导您完成RocketMQ在Linux上的安装配置以及如何实现开机自启。首先，让我们依托已有的知识内容来逐步进行。

### 1. **下载与安装RocketMQ**

按照之前的指引，您需要先下载RocketMQ的二进制包或源码包。对于快速入门，我们推荐直接使用二进制包进行部署。您可以点击[此处](https://dist.apache.org/repos/dist/release/rocketmq/5.3.0/rocketmq-all-5.3.0-bin-release.zip)下载Apache RocketMQ 5.3.0的二进制包。

#### 具体步骤如下：

- **下载二进制包**：
  ```shell
  wget https://dist.apache.org/repos/dist/release/rocketmq/5.3.0/rocketmq-all-5.3.0-bin-release.zip
  ```

- **解压安装**：
  ```shell
  unzip rocketmq-all-5.3.0-bin-release.zip
  cd rocketmq-all-5.3.0-bin-release/
  ```

### 2. **配置RocketMQ**

虽然基础配置在快速入门中未详细展开，但为了确保服务稳定运行，您可能需要调整一些基本配置，例如修改broker配置文件（位于`conf/broker.conf`）以适应您的生产环境需求，比如IP地址、端口号等。

### 3. **启动NameServer**

接下来，您需要启动NameServer，它是RocketMQ集群中的命名服务，负责管理路由信息。

- **启动命令**：
  ```shell
  nohup sh bin/mqnamesrv &
  tail -f ~/logs/rocketmqlogs/namesrv.log
  ```
  
  确保看到 "The Name Server boot success..." 表明NameServer已成功启动。

### 4. **启动Broker**

然后启动Broker，这里我们以单节点部署为例：

- **启动命令**：
  ```shell
  nohup sh bin/mqbroker -n localhost:9876 autoCreateTopicEnable=true & 
  tail -f ~/logs/rocketmqlogs/broker.log
  ```
  
  注意，这里开启了`autoCreateTopicEnable`参数以便自动创建主题。确认日志中显示"The broker... boot success..."表明Broker已启动成功。

### 5. **开机自启配置**

为了让RocketMQ服务在系统重启后自动启动，您需要将启动命令添加到系统的启动脚本中。具体操作因Linux发行版而异，以下以Systemd为例（适用于大多数现代Linux发行版）：

1. **创建Systemd服务单元文件**：
   - 编辑 `/etc/systemd/system/rocketmq-namesrv.service` 和 `/etc/systemd/system/rocketmq-broker.service` 文件，分别加入以下内容：

   **namesrv.service**:
   ```ini
   [Unit]
   Description=Apache RocketMQ NameServer
   After=network.target

   [Service]
   Type=simple
   User=root
   ExecStart=/path/to/your/rocketmq/bin/mqnamesrv
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

   **broker.service**:
   ```ini
   [Unit]
   Description=Apache RocketMQ Broker
   After=network.target rocketmq-namesrv.service

   [Service]
   Type=simple
   User=root
   ExecStart=/path/to/your/rocketmq/bin/mqbroker -n localhost:9876 autoCreateTopicEnable=true
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

2. **启用并启动服务**：
   ```shell
   systemctl daemon-reload
   systemctl enable rocketmq-namesrv
   systemctl enable rocketmq-broker
   systemctl start rocketmq-namesrv
   systemctl start rocketmq-broker
   ```

通过以上步骤，您不仅完成了RocketMQ在Linux上的安装与基本配置，还实现了服务的开机自启。请根据实际环境调整路径与用户权限，确保服务能够顺利运行。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：本地部署RocketMQ 5.0 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16765)给我们反馈。
