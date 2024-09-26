---
id: "question-history-16742"
title: "RocketMQ详细安装教程 包括但不限于 安装、改默认端口、console管理后台下载和安装"
date: "2024-09-26"
category: "expertConsultation"
description: "根据您提出的需求，即获取RocketMQ的详细安装教程，包括但不限于安装、修改默认端口以及console管理后台（这里指`RocketMQ Dashboard`）的下载与安装步骤，我们可以依据提供的知识来构建一个全面的指南。以下是按照您的要求整理的具体步骤。### 1. RocketMQ 安装###"
tags: ["rocketmq","安装","教程","端口","console","管理","下载"]
keywords: ["rocketmq","安装","教程","端口","console","管理","下载"]
---

## 用户问题 ： 
 RocketMQ详细安装教程 包括但不限于 安装、改默认端口、console管理后台下载和安装  

---------------
## 专家官方解答 ：

根据您提出的需求，即获取RocketMQ的详细安装教程，包括但不限于安装、修改默认端口以及console管理后台（这里指`RocketMQ Dashboard`）的下载与安装步骤，我们可以依据提供的知识来构建一个全面的指南。以下是按照您的要求整理的具体步骤。

### 1. RocketMQ 安装

#### 系统要求
- 操作系统：Linux/Unix/Mac
- JDK版本：64位JDK 1.8或以上
- Maven版本：3.2.x
- 需要先启动[RocketMQ](https://rocketmq.apache.org/docs/quick-start/)服务本身

#### 下载RocketMQ
访问[官方下载页面](https://rocketmq.apache.org/zh/download/)选择合适的版本进行下载。例如，最新稳定版为5.3.0，您可以直接从那里获取二进制包或者源代码包。

```shell
# 示例命令，实际链接请替换为最新的
wget https://dist.apache.org/repos/dist/release/rocketmq/5.3.0/rocketmq-all-5.3.0-bin-release.zip
unzip rocketmq-all-5.3.0-bin-release.zip -d /opt/
```

#### 启动Name Server
进入解压后的目录并启动Name Server。
```shell
cd /opt/rocketmq-all-5.3.0-bin-release
nohup sh bin/mqnamesrv &
```

#### 启动Broker
同样在上述目录中，使用如下命令启动Broker。
```shell
nohup sh bin/mqbroker -n localhost:9876 &
```

### 2. 修改RocketMQ默认端口

- **修改Broker端口**：编辑`conf/broker.conf`文件中的`listenPort`和`haListenPort`字段值。
- **修改Dashboard端口**：对于RocketMQ Dashboard，可以通过修改其`application.yml`配置文件内的`server.port`属性来自定义监听端口。具体位置取决于您的安装方式（Docker镜像安装或源码安装）。

### 3. RocketMQ Dashboard 安装

#### Docker镜像安装
1. 安装Docker，并拉取`rocketmq-dashboard`镜像。
   ```shell
   docker pull apacherocketmq/rocketmq-dashboard:latest
   ```
2. 使用Docker运行`rocketmq-dashboard`容器。
   ```shell
   docker run -d --name rocketmq-dashboard -e "JAVA_OPTS=-Drocketmq.namesrv.addr=127.0.0.1:9876" -p 8080:8080 -t apacherocketmq/rocketmq-dashboard:latest
   ```

#### 源码安装
1. 访问GitHub仓库[apache/rocketmq-dashboard](https://github.com/apache/rocketmq-dashboard)克隆项目到本地。
2. 在项目根目录执行以下命令编译打包。
   ```shell
   mvn clean package -Dmaven.test.skip=true
   ```
3. 执行生成的jar包以启动服务。
   ```shell
   java -jar target/rocketmq-dashboard-1.0.1-SNAPSHOT.jar
   ```

通过以上步骤，您应该能够成功地完成RocketMQ及其Dashboard的安装与基本配置调整。如果有任何进一步的问题或需要更详细的说明，请随时告知。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：RocketMQ Dashboard 
 
 * 专家经验：rocketmq端口介绍 
 
 * 专家经验：RocketMQ 版本说明 下载 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17282)给我们反馈。
