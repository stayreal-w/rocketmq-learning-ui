---
id: "question-history-16594"
title: "rocketmq dledger介绍"
date: "2024-09-26"
category: "expertConsultation"
description: "### 问题：rocketmq dledger介绍#### 分析根据提供的我了解的信息，DLedger是一套基于Raft协议的分布式日志存储组件。它被设计用来增强RocketMQ的高可用性和容灾能力。通过使用DLedger，可以实现自动选举Leader和数据复制，从而在节点故障时能够快速进行容灾切换"
tags: ["rocketmq","dledger","介绍"]
keywords: ["rocketmq","dledger","介绍"]
---

## 用户问题 ： 
 rocketmq dledger介绍  

---------------
## 专家官方解答 ：

### 问题：rocketmq dledger介绍

#### 分析
根据提供的我了解的信息，DLedger是一套基于Raft协议的分布式日志存储组件。它被设计用来增强RocketMQ的高可用性和容灾能力。通过使用DLedger，可以实现自动选举Leader和数据复制，从而在节点故障时能够快速进行容灾切换，并确保集群中的数据一致性。这种机制特别适合需要高度可靠性的应用场景。

#### 具体步骤（基于快速搭建）
1. **源码构建**
   - 首先构建DLedger：
     ```shell
     $ git clone https://github.com/openmessaging/dledger.git
     $ cd dledger
     $ mvn clean install -DskipTests
     ```
   - 接着构建RocketMQ：
     ```shell
     $ git clone https://github.com/apache/rocketmq.git
     $ cd rocketmq
     $ git checkout -b develop origin/develop
     $ mvn -Prelease-all -DskipTests clean install -U
     ```

2. **快速部署**
   - 构建完成后，进入RocketMQ版本目录并启动DLedger：
     ```shell
     # {rocketmq-version}替换为实际的RocketMQ版本号，例如5.1.0
     $ cd distribution/target/rocketmq-{rocketmq-version}/rocketmq-{rocketmq-version}
     $ sh bin/dledger/fast-try.sh start
     ```
   - 使用`mqadmin`命令检查集群状态：
     ```shell
     $ sh bin/mqadmin clusterList -n 127.0.0.1:9876
     ```
   - 如果一切正常，你将看到集群中有一个Master (BID 为 0) 和多个Follower。
   - 若要停止集群，执行以下命令：
     ```shell
     $ sh bin/dledger/fast-try.sh stop
     ```

3. **容灾测试**
   - 在集群运行期间，可以通过杀死当前的Leader进程来模拟故障情况（例如，如果Leader位于端口30931上，则可以终止该进程）。
   - 等待大约10秒后，再次使用`clusterList`命令查看集群状态，此时应该可以看到一个新的Leader被选出来了。

#### 解释
上述步骤提供了一个从零开始到成功部署一个基于DLedger的RocketMQ集群的方法。首先通过源码构建得到最新的DLedger与RocketMQ，然后利用提供的脚本快速启动一个小型集群用于测试或学习目的。最后，通过简单的操作演示了如何触发并观察集群的容灾切换过程，这有助于理解DLedger如何提高系统的可靠性。对于生产环境下的部署，建议进一步参考官方文档以获取更详细的配置指导和支持。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq Dledger 
 
 * 专家经验：rocketmq  Dledger集群搭建 
 
 * 专家经验：rocketmq  Dledger快速搭建 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17269)给我们反馈。
