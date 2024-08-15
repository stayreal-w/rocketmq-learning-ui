---
id: "question-history-16594"
title: "rocketmq dledger介绍"
date: "2024-08-09"
category: "expertConsultation"
description: "内容：**RocketMQ Dledger介绍**RocketMQ Dledger是一种基于Raft协议设计的分布式日志存储组件，它被用于增强RocketMQ的高可用性和容灾能力。通过将RocketMQ的副本存储机制替换为DLedger，可以实现自动的容灾切换，保障消息系统的稳定运行。以下是关于Dl"
---

## 用户问题 ： 
 rocketmq dledger介绍  

---------------
## 专家官方解答 ：

内容：

**RocketMQ Dledger介绍**

RocketMQ Dledger是一种基于Raft协议设计的分布式日志存储组件，它被用于增强RocketMQ的高可用性和容灾能力。通过将RocketMQ的副本存储机制替换为DLedger，可以实现自动的容灾切换，保障消息系统的稳定运行。以下是关于Dledger快速搭建及集群搭建的详细步骤与说明：

### 1. Dledger快速搭建
#### 步骤解析：
- **源码构建DLedger与RocketMQ**
  1. 先克隆DLedger的GitHub仓库并使用Maven构建。
  2. 接着，克隆RocketMQ仓库，切换到develop分支并构建。
  
- **快速部署**
  1. 在RocketMQ构建成功后，进入特定目录并使用`fast-try.sh start`脚本启动快速集群。
  2. 使用`mqadmin clusterList`命令检查集群状态，确认部署成功。
  
- **操作说明**：这一步骤旨在让你快速体验DLedger支持的RocketMQ集群，包括启动与基本的健康检查。

### 2. Dledger集群搭建
#### 步骤与说明：
- **配置准备**
  1. 至少准备三台服务器，并分别为每个节点编写配置文件，确保配置中`dLegerGroup`、`dLegerPeers`、`dLegerSelfId`等关键参数设置正确。
  
- **启动Broker**
  1. 分别在每台服务器上使用`mqbroker`命令配合各自的配置文件启动Broker服务。
  
- **部署策略**：此步骤详细介绍了如何从零开始构建一个具备自动容灾能力的RocketMQ集群，强调了配置的细节与启动流程。

### 3. 旧集群升级
#### 升级步骤简述：
- **停运旧Broker**
  1. 优雅地关闭现有的RocketMQ实例。
  
- **数据一致性检查**
  2. 确保所有Master或Master-Slave组的CommitLog数据一致。
  
- **配置修改与重启**
  3. 对旧集群进行必要的配置调整以适应DLedger模式，然后按照新集群的方式重新启动Broker。

- **升级策略**：升级流程特别注意了数据一致性的维护，确保在切换至DLedger架构时数据的完整与一致性。

### 总结
Dledger的引入极大提升了RocketMQ集群的高可用性和容错能力。通过快速搭建和正式集群部署的指南，你可以根据实际需求选择合适的部署方案，无论是初步尝试还是大规模生产环境的部署升级，Dledger都能提供坚实的支持。请参照上述步骤操作，并结合[DLedger快速搭建](https://github.com/openmessaging/dledger)及[RocketMQ中文学习社区](https://rocketmq-learning.com)的资源进行深入学习与实践。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq Dledger 
 
 * 专家经验：rocketmq中文学习社区介绍 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>，在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。您也可以访问 : [全局专家答疑](https://answer.opensource.alibaba.com/docs/intro) 。 咨询其他产品的的问题

### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=16601)给我们反馈。
