---
id: "question-history-16471"
title: "linux rocketmq 部署安装运行"
date: "2024-09-26"
category: "expertConsultation"
description: "根据提供的我了解的信息，要解决在Linux上部署安装并运行RocketMQ的问题，我们可以遵循以下步骤：### 1. 确认系统要求- 64位操作系统（推荐 Linux/Unix/macOS）。- 64位 JDK 1.8+ 已经安装。### 2. 下载与解压RocketMQ首先需要下载Apache R"
tags: ["rocketmq","linux","部署","安装","运行"]
keywords: ["rocketmq","linux","部署","安装","运行"]
---

## 用户问题 ： 
 linux rocketmq 部署安装运行  

---------------
## 专家官方解答 ：

根据提供的我了解的信息，要解决在Linux上部署安装并运行RocketMQ的问题，我们可以遵循以下步骤：

### 1. 确认系统要求
- 64位操作系统（推荐 Linux/Unix/macOS）。
- 64位 JDK 1.8+ 已经安装。

### 2. 下载与解压RocketMQ
首先需要下载Apache RocketMQ的源码包或二进制包。这里以源码包为例：
```shell
$ wget https://dist.apache.org/repos/dist/release/rocketmq/5.3.0/rocketmq-all-5.3.0-source-release.zip
$ unzip rocketmq-all-5.3.0-source-release.zip
$ cd rocketmq-all-5.3.0-source-release/
```
接着编译构建二进制可执行文件：
```shell
$ mvn -Prelease-all -DskipTests -Dspotbugs.skip=true clean install -U
```
完成后进入构建后的目录：
```shell
$ cd distribution/target/rocketmq-5.3.0/rocketmq-5.3.0
```

### 3. 启动NameServer
NameServer是RocketMQ中的一个非常重要的组件，用于管理broker信息。启动命令如下：
```shell
$ nohup sh bin/mqnamesrv &
```
检查是否成功启动：
```shell
$ tail -f ~/logs/rocketmqlogs/namesrv.log
```
如果看到`The Name Server boot success...`的信息，则表示NameServer已经成功启动。

### 4. 启动Broker+Proxy
接下来启动Broker和Proxy服务，这里使用Local模式进行部署：
```shell
$ nohup sh bin/mqbroker -n localhost:9876 --enable-proxy &
```
验证Broker是否启动成功：
```shell
$ tail -f ~/logs/rocketmqlogs/proxy.log
```
当看到类似于`The broker[broker-a,192.169.1.2:10911] boot success...`的消息时，说明Broker已成功启动。

### 5. 测试消息收发
为了测试RocketMQ是否能够正常工作，可以通过设置环境变量`NAMESRV_ADDR`来指定NameServer地址，并使用示例脚本发送接收消息：
```shell
$ export NAMESRV_ADDR=localhost:9876
$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
```
这将展示如何通过RocketMQ发送一条简单的消息以及如何接收这条消息。

### 解释
上述步骤涵盖了从下载、编译到启动RocketMQ核心组件的过程，确保了RocketMQ能够在你的Linux环境中正确地安装和运行。每个步骤都是基于官方文档提供的最佳实践，保证了系统的稳定性和性能。此外，通过最后一步的消息收发测试，可以快速验证整个配置过程是否成功，为后续开发打下坚实的基础。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：本地部署RocketMQ 5.0 
 
 * 专家经验：Docker 部署 RocketMQ 5.x 
 
 * 专家经验：Docker Compose 部署 RocketMQ 5.x 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17245)给我们反馈。
