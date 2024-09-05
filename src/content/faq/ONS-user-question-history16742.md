---
id: "question-history-16742"
title: "RocketMQ详细安装教程 包括但不限于 安装、改默认端口、console管理后台下载和安装"
date: "2024-09-05"
category: "expertConsultation"
description: "为了帮助您完成RocketMQ的详细安装教程，包括安装、更改默认端口以及RocketMQ Console管理后台的下载与安装，我将结合已有的知识内容为您提供一个综合性的指导方案。请注意，以下步骤将涵盖从零开始部署RocketMQ到配置管理界面的过程。### 1. 安装RocketMQ#### 步骤1"
---

## 用户问题 ： 
 RocketMQ详细安装教程 包括但不限于 安装、改默认端口、console管理后台下载和安装  

---------------
## 专家官方解答 ：

为了帮助您完成RocketMQ的详细安装教程，包括安装、更改默认端口以及RocketMQ Console管理后台的下载与安装，我将结合已有的知识内容为您提供一个综合性的指导方案。请注意，以下步骤将涵盖从零开始部署RocketMQ到配置管理界面的过程。

### 1. 安装RocketMQ

#### 步骤1：环境准备
确保您的系统满足以下要求：
- **操作系统**：Linux/Unix/Mac
- **JDK版本**：64bit JDK 1.8+
- **Maven版本**：3.2.x

#### 步骤2：下载RocketMQ
从Apache RocketMQ官网下载最新版本的RocketMQ发行包，地址为：[Apache RocketMQ Download](https://rocketmq.apache.org/release_notes/downloads/)。

#### 步骤3：解压与配置
解压缩下载的RocketMQ包，进入目录，根据需要修改配置文件（如`broker.conf`或`broker-x.properties`）中的nameserver地址。例如：
```properties
namesrvAddr=127.0.0.1:9876
```

#### 步骤4：启动Nameserver
在RocketMQ的bin目录下执行启动命令：
```bash
sh mqnamesrv
```

#### 步骤5：启动Broker
同样在bin目录下，根据配置文件启动Broker，例如：
```bash
sh mqbroker -c ../conf/broker.conf
```

### 2. 更改默认端口

如果您需要修改RocketMQ各组件的默认端口，请参考以下步骤：

- **Broker端口**：在Broker的配置文件中修改`listenPort`(默认10911)、`haListenPort`(默认10912)等。
- **RocketMQ Proxy端口**：编辑`conf/rmq-proxy.json`，调整`remotingListenPort`(默认8080)和`grpcServerPort`(默认8081)。
- **RocketMQ Dashboard**：修改`spring程序的application.yml`文件，自定义Dashboard的端口（默认也是8080）。参考配置位于：[GitHub RocketMQ Dashboard配置](https://github.com/apache/rocketmq-dashboard/blob/master/src/main/resources/application.yml)

### 3. RocketMQ Console管理后台下载与安装

#### 方式一：Docker镜像安装
1. **拉取镜像**：
   ```bash
   docker pull apacherocketmq/rocketmq-dashboard:latest
   ```
2. **运行容器**：
   ```bash
   docker run -d --name rocketmq-dashboard -e "JAVA_OPTS=-Drocketmq.namesrv.addr=您的Nameserver地址:端口" -p 您希望映射的端口:8080 -t apacherocketmq/rocketmq-dashboard:latest
   ```

#### 方式二：源码安装
1. **克隆源码**：
   ```bash
   git clone https://github.com/apache/rocketmq-dashboard.git
   ```
2. **编译项目**：
   ```bash
   cd rocketmq-dashboard
   mvn clean package -Dmaven.test.skip=true
   ```
3. **运行应用**：
   ```bash
   java -jar target/rocketmq-dashboard-1.0.1-SNAPSHOT.jar
   ```
   访问地址为`http://localhost:您设定的端口`。

### 解释说明
以上步骤首先介绍了如何从零开始安装RocketMQ，包括必要的环境准备、软件下载、配置修改以及服务的启动过程。接着，提供了更改RocketMQ各组件默认端口的方法，以适应不同的部署需求。最后，针对RocketMQ Console管理后台，提供了两种便捷的安装方式，无论是通过Docker快速部署还是手动编译源码运行，都能帮助您轻松管理RocketMQ集群。请根据实际需求选择合适的安装方式，并确保按照步骤正确配置相关参数。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq端口介绍 
 
 * 专家经验：RocketMQ Dashboard 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16762)给我们反馈。
