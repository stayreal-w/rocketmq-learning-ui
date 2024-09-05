---
id: "question-history-16755"
title: "linux 环境下rocketmq-console安装和部署"
date: "2024-09-05"
category: "expertConsultation"
description: "在Linux环境下部署RocketMQ Console（现称为RocketMQ Dashboard）以便于管理和监控RocketMQ集群，可以遵循以下详细步骤。根据您的需求，我将提供基于Docker镜像安装和源码安装两种方式的指导。### Docker镜像安装**步骤1：确保Docker已安装**确"
---

## 用户问题 ： 
 linux 环境下rocketmq-console安装和部署  

---------------
## 专家官方解答 ：

在Linux环境下部署RocketMQ Console（现称为RocketMQ Dashboard）以便于管理和监控RocketMQ集群，可以遵循以下详细步骤。根据您的需求，我将提供基于Docker镜像安装和源码安装两种方式的指导。

### Docker镜像安装

**步骤1：确保Docker已安装**
确保您的Linux系统中已安装Docker。可以通过运行`docker --version`来检查Docker是否已安装及其版本。

**步骤2：拉取RocketMQ Dashboard镜像**
打开终端，执行以下命令以获取最新的RocketMQ Dashboard镜像：
```shell
$ docker pull apacherocketmq/rocketmq-dashboard:latest
```

**步骤3：运行RocketMQ Dashboard容器**
使用Docker运行RocketMQ Dashboard容器，记得替换`namesrv.addr:port`为您的RocketMQ Nameserver地址和端口：
```shell
$ docker run -d --name rocketmq-dashboard \
  -e "JAVA_OPTS=-Drocketmq.namesrv.addr=您的Nameserver地址:端口" \
  -p 8080:8080 \
  -t apacherocketmq/rocketmq-dashboard:latest
```

**步骤4：配置网络与访问**
- **云服务器**：请确保相应的端口（如8080）已在安全组规则中开放。
- **本地虚拟机**：如需外部访问，需确保端口映射正确且防火墙规则允许。

### 源码安装

**步骤1：准备环境**
确保系统满足以下要求：
- Linux/Unix/Mac操作系统
- 64位JDK 1.8+
- Maven 3.2.x

**步骤2：下载源码**
从GitHub克隆RocketMQ Dashboard的源码：
```shell
$ git clone https://github.com/apache/rocketmq-dashboard.git
```

**步骤3：编译源码**
进入项目目录并编译：
```shell
$ cd rocketmq-dashboard
$ mvn clean package -Dmaven.test.skip=true
```

**步骤4：运行RocketMQ Dashboard**
执行以下命令启动应用：
```shell
$ java -jar target/rocketmq-dashboard-1.0.1-SNAPSHOT.jar
```
提示“Started App in x.xxx seconds (JVM running for x.xxx)”表示服务已启动成功。

**步骤5：访问Dashboard**
在浏览器中输入`http://您的Nameserver地址:8080`访问RocketMQ Dashboard。

### 注意事项
- 确保RocketMQ服务本身已正确安装并配置好Nameserver地址。
- 如果使用源码安装，在Linux系统上直接编译部署通常更为稳定，避免了本地编译可能遇到的兼容性问题。
- 考虑到安全性，生产环境中应适当限制对Dashboard访问的IP范围。

以上步骤提供了在Linux环境下安装RocketMQ Dashboard的完整指南，无论是通过快速简便的Docker镜像还是更灵活的源码编译方式。[[源码地址]](https://github.com/apache/rocketmq-dashboard) 提供了更多自定义配置和开发的可能性。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：RocketMQ Dashboard 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16775)给我们反馈。
