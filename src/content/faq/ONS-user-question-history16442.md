---
id: "question-history-16442"
title: "rocketmq dledger 使用说明"
date: "2024-08-07"
category: "expertConsultation"
description: "内容要有条理，分步骤：### RocketMQ Dledger 使用说明概览#### 1. **Dledger快速搭建**   - **前言**: Dledger是一个基于Raft协议的分布式日志存储组件，能够增强RocketMQ的容灾能力。以下是快速构建和部署基于Dledger的RocketMQ集"
---

## 用户问题 ： 
 rocketmq dledger 使用说明  

---------------
## 专家官方解答 ：

内容要有条理，分步骤：

### RocketMQ Dledger 使用说明概览

#### 1. **Dledger快速搭建**
   - **前言**: Dledger是一个基于Raft协议的分布式日志存储组件，能够增强RocketMQ的容灾能力。以下是快速构建和部署基于Dledger的RocketMQ集群步骤。
   
   - **源码构建**:
     1. **构建DLedger**: 
        ```shell
        $ git clone https://github.com/openmessaging/dledger.git
        $ cd dledger
        $ mvn clean install -DskipTests
        ```
     2. **构建RocketMQ**: 
        ```shell
        $ git clone https://github.com/apache/rocketmq.git
        $ cd rocketmq
        $ git checkout -b develop origin/develop
        $ mvn -Prelease-all -DskipTests clean install -U
        ```
        
   - **快速部署**: 
     ```shell
     #{rocketmq-version} 替换为实际的RocketMQ版本，例如: 5.1.0
     $ cd distribution/target/rocketmq-{rocketmq-version}/rocketmq-{rocketmq-version}
     $ sh bin/dledger/fast-try.sh start
     ```
     成功后，可通过`mqadmin clusterList`命令查看集群状态。

   - **停止集群**: 
     ```shell
     $ sh bin/dledger/fast-try.sh stop
     ```

#### 2. **Dledger集群搭建**
   - **新集群部署**:
     1. **编写配置**: 配置示例包括`enableDLegerCommitLog`, `dLegerGroup`, `dLegerPeers`, `dLegerSelfId`, 等关键参数。
     2. **启动Broker**: 使用`nohup sh bin/mqbroker -c conf/dledger/xxx-nX.conf &`命令分别启动不同节点的Broker。
     
   - **旧集群升级**:
     1. **杀掉旧Broker**.
     2. **检查并确保旧Commitlog一致性**.
     3. **准备第三台机器**，确保数据对齐。
     4. **修改配置**，参照新集群配置。
     5. **重新启动Broker**，按照新集群部署步骤操作。

#### 解释与补充
以上步骤详细介绍了如何从零开始搭建基于Dledger的RocketMQ集群以及如何将现有的RocketMQ集群升级至支持Dledger模式，确保了数据的一致性和服务的高可用性。Dledger通过Raft协议实现了自动容灾切换，提升了RocketMQ集群的可靠性。在部署过程中，务必注意配置文件的正确填写以及网络、存储等基础设施的稳定，以避免部署失败或运行时异常。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq Dledger 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16443)给我们反馈。
