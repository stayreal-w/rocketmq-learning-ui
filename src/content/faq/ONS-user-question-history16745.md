---
id: "question-history-16745"
title: "linux 安装 RocketMQ快速入门，安装 配置 并开机自启 介绍"
date: "2024-09-26"
category: "expertConsultation"
description: "根据我了解的信息中的内容，我将按照以下步骤来解答如何在Linux环境下安装、配置RocketMQ，并设置其开机自启。这些步骤基于第1篇参考内容《本地部署RocketMQ 5.0》。### 分析问题原因要解决的问题是如何在Linux系统上快速地完成RocketMQ的安装与配置，并确保服务能够在系统重启"
tags: ["rocketmq","linux","安装","快速入门","配置","开机自启","介绍"]
keywords: ["rocketmq","linux","安装","快速入门","配置","开机自启","介绍"]
---

## 用户问题 ： 
 linux 安装 RocketMQ快速入门，安装 配置 并开机自启 介绍  

---------------
## 专家官方解答 ：

根据我了解的信息中的内容，我将按照以下步骤来解答如何在Linux环境下安装、配置RocketMQ，并设置其开机自启。这些步骤基于第1篇参考内容《本地部署RocketMQ 5.0》。

### 分析问题原因
要解决的问题是如何在Linux系统上快速地完成RocketMQ的安装与配置，并确保服务能够在系统重启后自动启动。这涉及到软件包下载、环境准备、RocketMQ组件（如NameServer, Broker）的启动以及通过适当的方法实现服务的持久化运行。

### 具体步骤

#### 一、准备工作
1. 确认你的操作系统为64位，并已安装JDK 1.8+。
2. 下载Apache RocketMQ 5.3.0版本的源码包或二进制包。推荐使用二进制包以简化流程。[点击这里](https://dist.apache.org/repos/dist/release/rocketmq/5.3.0/rocketmq-all-5.3.0-bin-release.zip)下载二进制包。

#### 二、安装RocketMQ
1. 解压下载好的文件：
   ```shell
   $ unzip rocketmq-all-5.3.0-bin-release.zip
   $ cd rocketmq-all-5.3.0-bin-release/
   ```
2. 启动NameServer：
   ```shell
   $ nohup sh bin/mqnamesrv &
   ```
   检查是否成功启动：`$ tail -f ~/logs/rocketmqlogs/namesrv.log`

3. 启动Broker：
   ```shell
   $ nohup sh bin/mqbroker -n localhost:9876 &
   ```
   验证启动状态：`$ tail -f ~/logs/rocketmqlogs/broker.log`

#### 三、配置开机自启
为了使RocketMQ随系统一起启动，可以利用systemd服务管理器创建相应的服务单元文件。

1. 创建名为`rocketmq-namesrv.service`的服务文件：
   ```ini
   [Unit]
   Description=Apache RocketMQ Name Server
   After=network.target

   [Service]
   Type=forking
   ExecStart=/path/to/your/rocketmq/bin/mqnamesrv
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

2. 类似地，创建`rocketmq-broker.service`：
   ```ini
   [Unit]
   Description=Apache RocketMQ Broker
   After=network.target

   [Service]
   Type=forking
   Environment="NAMESRV_ADDR=localhost:9876"
   ExecStart=/path/to/your/rocketmq/bin/mqbroker -n ${NAMESRV_ADDR}
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

3. 将上述两个`.service`文件放置于`/etc/systemd/system/`目录下。
4. 使用以下命令启用并立即启动服务：
   ```shell
   $ sudo systemctl enable rocketmq-namesrv
   $ sudo systemctl start rocketmq-namesrv
   $ sudo systemctl enable rocketmq-broker
   $ sudo systemctl start rocketmq-broker
   ```

现在，RocketMQ应该已经正确安装、配置并且能够随着系统的启动而自动运行了。

### 解释
通过以上步骤，我们首先确保了必要的软件环境满足要求，然后手动启动了RocketMQ的核心组件NameServer和Broker。最后，通过编写Systemd服务单元文件，实现了RocketMQ服务的开机自启功能。这样做的好处是保证了即使服务器意外重启后，消息队列服务仍然能够正常提供服务，提高了系统的可靠性和可用性。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：本地部署RocketMQ 5.0 
 
 * 专家经验：RocketMQ Dashboard 
 
 * 专家经验：rocketmq JVM/OS配置 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17285)给我们反馈。
