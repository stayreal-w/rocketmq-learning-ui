---
id: "question-history-16442"
title: "rocketmq dledger 使用说明"
date: "2024-09-26"
category: "expertConsultation"
description: "### RocketMQ Dledger 使用说明#### 1. 源码构建首先，需要分别构建 DLedger 和 RocketMQ。DLedger 是基于 Raft 协议的分布式日志存储组件，用于替换 RocketMQ 的原生副本存储机制。- **构建 DLedger**  ```shell  $ "
tags: ["rocketmq","dledger","使用","说明"]
keywords: ["rocketmq","dledger","使用","说明"]
---

## 用户问题 ： 
 rocketmq dledger 使用说明  

---------------
## 专家官方解答 ：

### RocketMQ Dledger 使用说明

#### 1. 源码构建
首先，需要分别构建 DLedger 和 RocketMQ。DLedger 是基于 Raft 协议的分布式日志存储组件，用于替换 RocketMQ 的原生副本存储机制。

- **构建 DLedger**
  ```shell
  $ git clone https://github.com/openmessaging/dledger.git
  $ cd dledger
  $ mvn clean install -DskipTests
  ```

- **构建 RocketMQ**
  ```shell
  $ git clone https://github.com/apache/rocketmq.git
  $ cd rocketmq
  $ git checkout -b develop origin/develop
  $ mvn -Prelease-all -DskipTests clean install -U
  ```

#### 2. 快速部署
在完成源码构建后，可以快速启动一个基于 DLedger 的 RocketMQ 集群。

- **启动集群**
  ```shell
  # 替换 {rocketmq-version} 为实际版本号，例如 5.1.0
  $ cd distribution/target/rocketmq-{rocketmq-version}/rocketmq-{rocketmq-version}
  $ sh bin/dledger/fast-try.sh start
  ```
  成功启动后，可以通过 `mqadmin` 命令检查集群状态：
  ```shell
  $ sh bin/mqadmin clusterList -n 127.0.0.1:9876
  ```

- **关闭集群**
  如果需要停止快速部署的集群，可以执行以下命令：
  ```shell
  $ sh bin/dledger/fast-try.sh stop
  ```

#### 3. 容灾切换
当集群中的 Leader 节点发生故障时，DLedger 会自动进行容灾切换，选举出新的 Leader。例如，如果要模拟 Leader 故障，可以杀死端口 30931 上运行的进程（假设这是 Leader 的监听端口）。等待约 10 秒后，再次使用 `clusterList` 命令查看，可以看到 Leader 已经切换到其他节点。

#### 4. 新集群部署
对于生产环境，建议按照以下步骤搭建完整的 DLedger 集群：

- **编写配置文件**
  每个 RocketMQ-on-DLedger Group 至少需要三台机器。每个 Broker 需要一个配置文件，关键参数包括 `enableDLegerCommitLog`, `dLegerGroup`, `dLegerPeers`, `dLegerSelfId` 等。示例配置如下：
  ```properties
  brokerClusterName = RaftCluster
  brokerName=RaftNode00
  listenPort=30911
  namesrvAddr=127.0.0.1:9876
  storePathRootDir=/tmp/rmqstore/node00
  storePathCommitLog=/tmp/rmqstore/node00/commitlog
  enableDLegerCommitLog=true
  dLegerGroup=RaftNode00
  dLegerPeers=n0-127.0.0.1:40911;n1-127.0.0.1:40912;n2-127.0.0.1:40913
  dLegerSelfId=n0
  sendMessageThreadPoolNums=16
  ```

- **启动 Broker**
  在每台机器上启动对应的 Broker 实例：
  ```shell
  $ nohup sh bin/mqbroker -c conf/dledger/broker-n0.conf &
  $ nohup sh bin/mqbroker -c conf/dledger/broker-n1.conf &
  $ nohup sh bin/mqbroker -c conf/dledger/broker-n2.conf &
  ```

#### 5. 旧集群升级
将现有的 RocketMQ 集群升级到支持 DLedger 的模式，主要分为以下几个步骤：

- **停止旧 Broker**
  可以通过 `kill` 命令或 `bin/mqshutdown broker` 来停止正在运行的 Broker。
  
- **检查 Commitlog**
  确保所有 Broker 的 Commitlog 文件一致。如果不一致，需要手动对齐这些文件。对于 Master-Slave 部署，可能还需要准备第三台机器，并从 Master 复制 Commitlog 到新机器上。

- **修改配置**
  根据新集群部署的要求调整配置文件。

- **重新启动 Broker**
  按照新集群部署的方式重新启动 Broker。

通过上述步骤，您可以成功地构建和管理一个高可用且具备自动容灾能力的 RocketMQ 集群。更多详细信息请参考 [RocketMQ官方文档](https://github.com/apache/rocketmq) 和 [DLedger GitHub项目](https://github.com/openmessaging/dledger)。


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
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17230)给我们反馈。
