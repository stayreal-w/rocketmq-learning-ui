---
id: "question-history-16470"
title: "windows rocketmq 部署安装运行"
date: "2024-09-26"
category: "expertConsultation"
description: "根据我了解的信息中的信息，RocketMQ 5.0 可以在 Windows 上进行本地部署。以下是详细的步骤来帮助您完成 RocketMQ 的安装、启动以及简单的消息收发测试。### 系统要求- 64位操作系统（Windows）- 64位 JDK 1.8+### 1. 下载并解压 RocketMQ首"
tags: ["rocketmq","windows","部署","安装","运行"]
keywords: ["rocketmq","windows","部署","安装","运行"]
---

## 用户问题 ： 
 windows rocketmq 部署安装运行  

---------------
## 专家官方解答 ：

根据我了解的信息中的信息，RocketMQ 5.0 可以在 Windows 上进行本地部署。以下是详细的步骤来帮助您完成 RocketMQ 的安装、启动以及简单的消息收发测试。

### 系统要求
- 64位操作系统（Windows）
- 64位 JDK 1.8+

### 1. 下载并解压 RocketMQ
首先，从 Apache 官方网站下载 RocketMQ 的二进制包或源码包。对于初学者来说，推荐直接使用二进制包，因为这避免了编译过程。
- 二进制包下载链接: [这里](https://dist.apache.org/repos/dist/release/rocketmq/5.3.0/rocketmq-all-5.3.0-bin-release.zip)
- 源码包下载链接: [这里](https://dist.apache.org/repos/dist/release/rocketmq/5.3.0/rocketmq-all-5.3.0-source-release.zip)

下载完成后，解压到一个合适的位置，例如 `C:\rocketmq-5.3.0`。

### 2. 启动 NameServer
打开命令提示符窗口，并切换到 RocketMQ 解压目录下的 `bin` 文件夹，然后执行以下命令来启动 NameServer：
```shell
cd C:\rocketmq-5.3.0\bin
start mqnamesrv.cmd
```
检查是否成功启动，可以通过查看日志文件 `logs\namesrv.log` 来确认，寻找 "The Name Server boot success..." 字样。

### 3. 启动 Broker 和 Proxy
继续在同一命令提示符窗口中执行下面的命令来启动 Broker 和 Proxy：
```shell
start mqbroker.cmd -n localhost:9876 --enable-proxy
```
同样地，通过查看 `logs\proxy.log` 日志文件来验证 Broker 是否已经正确启动，查找 "The broker[brokerName,ip:port] boot success..." 相关的信息。

### 4. 测试消息收发
为了能够发送和接收消息，需要设置环境变量 `NAMESRV_ADDR` 指向 NameServer 地址。在命令行中输入:
```shell
set NAMESRV_ADDR=localhost:9876
```

接着，可以使用提供的工具脚本来快速测试消息的发送与接收功能：
- 发送消息:
  ```shell
  tools.cmd org.apache.rocketmq.example.quickstart.Producer
  ```
- 接收消息:
  ```shell
  tools.cmd org.apache.rocketmq.example.quickstart.Consumer
  ```

如果一切正常，你应该能看到发送的消息被消费者成功接收到的相关输出。

### 总结
以上步骤详细介绍了如何在 Windows 平台上安装和运行 RocketMQ 服务，并进行了基础的功能验证。如果您遇到任何问题，请确保您的 JDK 版本符合要求，并且所有相关配置都已正确设置。此外，也可以参考官方文档获取更多高级配置选项和支持信息。

请注意，虽然上述指南基于 Windows 平台，但大部分操作逻辑也适用于 Linux 或 macOS 环境，只需相应调整路径和命令格式即可。


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
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17244)给我们反馈。
