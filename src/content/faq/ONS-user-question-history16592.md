---
id: "question-history-16592"
title: "搭建"
date: "2024-09-26"
category: "expertConsultation"
description: "根据提供的我了解的信息，我们可以了解到搭建RocketMQ集群有两种方式：一种是使用Dledger实现自动容灾切换的高可用部署；另一种是通过快速尝试脚本来体验基于Dledger的RocketMQ集群。下面将按照这两种方法分别介绍如何进行RocketMQ的搭建。### 一、使用Dledger构建高可用"
tags: ["rocketmq","搭建","教程","","配置","集群","消息队列"]
keywords: ["rocketmq","搭建","教程","","配置","集群","消息队列"]
---

## 用户问题 ： 
 搭建  

---------------
## 专家官方解答 ：

根据提供的我了解的信息，我们可以了解到搭建RocketMQ集群有两种方式：一种是使用Dledger实现自动容灾切换的高可用部署；另一种是通过快速尝试脚本来体验基于Dledger的RocketMQ集群。下面将按照这两种方法分别介绍如何进行RocketMQ的搭建。

### 一、使用Dledger构建高可用RocketMQ集群

#### 1. 源码构建
首先需要分别构建DLedger和RocketMQ项目。
- **构建DLedger**:
  ```shell
  $ git clone https://github.com/openmessaging/dledger.git
  $ cd dledger
  $ mvn clean install -DskipTests
  ```
- **构建RocketMQ**:
  ```shell
  $ git clone https://github.com/apache/rocketmq.git
  $ cd rocketmq
  $ git checkout -b develop origin/develop
  $ mvn -Prelease-all -DskipTests clean install -U
  ```

#### 2. 配置与启动Broker
- 准备至少三台机器，并为每台机器编写配置文件。参考`conf/dledger`目录下的示例文件来创建自己的配置文件。
- 启动每个节点上的Broker服务：
  ```shell
  $ nohup sh bin/mqbroker -c conf/dledger/xxx-n0.conf &
  $ nohup sh bin/mqbroker -c conf/dledger/xxx-n1.conf &
  $ nohup sh bin/mqbroker -c conf/dledger/xxx-n2.conf &
  ```

#### 3. 容灾测试
- 在集群运行时，可以尝试关闭当前的Leader节点（如端口30931所在的进程），观察其他节点是否能成功选举出新的Leader并继续提供服务。

### 二、快速体验基于Dledger的RocketMQ集群

#### 1. 构建环境
同样的，先完成DLedger及RocketMQ的源码编译步骤。

#### 2. 快速部署
- 进入RocketMQ安装包目录后执行快速启动脚本：
  ```shell
  $ cd distribution/target/rocketmq-{版本号}/rocketmq-{版本号}
  $ sh bin/dledger/fast-try.sh start
  ```
- 使用`mqadmin`工具检查集群状态：
  ```shell
  $ sh bin/mqadmin clusterList -n 127.0.0.1:9876
  ```
- 若要停止集群，则运行：
  ```shell
  $ sh bin/dledger/fast-try.sh stop
  ```

### 解释
上述两种方法都提供了从零开始到拥有一个基本功能齐全的RocketMQ集群的过程。第一种方法适用于生产环境，它通过详细的配置确保了系统的稳定性和可靠性；而第二种方法则更适合于开发或测试阶段，用于快速验证概念或功能。无论哪种情况，都需要确保网络连通性以及正确的JVM参数设置以优化性能。对于更高级的功能如Dashboard管理界面等，可以根据具体需求参照相关文档进一步配置。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq Dledger 
 
 * 专家经验：rocketmq github 地址 
 
 * 专家经验：rocketmq  Dledger集群搭建 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17267)给我们反馈。
