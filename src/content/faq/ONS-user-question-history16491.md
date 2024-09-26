---
id: "question-history-16491"
title: "rocketmq metrics 怎么用？"
date: "2024-09-26"
category: "expertConsultation"
description: "要使用 RocketMQ 的 Metrics 来监控您的集群，首先需要了解 RocketMQ 提供了哪些指标以及这些指标的含义。根据我了解的信息中的信息，RocketMQ 支持 Prometheus 格式的 Metrics，并且从 5.1.0 版本开始提供了丰富的服务端、生产者和消费者相关指标。##"
tags: ["rocketmq","metrics","使用","方法"]
keywords: ["rocketmq","metrics","使用","方法"]
---

## 用户问题 ： 
 rocketmq metrics 怎么用？  

---------------
## 专家官方解答 ：

要使用 RocketMQ 的 Metrics 来监控您的集群，首先需要了解 RocketMQ 提供了哪些指标以及这些指标的含义。根据我了解的信息中的信息，RocketMQ 支持 Prometheus 格式的 Metrics，并且从 5.1.0 版本开始提供了丰富的服务端、生产者和消费者相关指标。

### 一、理解 Metrics 指标

RocketMQ 定义的 Metrics 包括 Counter（计数器）、Gauge（仪表盘）和 Histogram（直方图）三种类型，具体如下：

- **Counter**：用于记录累计数量或总量，例如 `rocketmq_messages_in_total` 表示消息生产数量。
- **Gauge**：反映当前状态值，如 `rocketmq_consumer_ready_messages` 显示已就绪的消息量。
- **Histogram**：提供数据分布情况，比如 `rocketmq_message_size` 描述了不同大小区间内的消息分布。

每个 Metric 都有其特定的标签（Labels），用以区分不同的维度，例如集群名 (`cluster`)、节点类型 (`node_type`) 等。

### 二、配置并启动 RocketMQ Exporter

为了能够通过 Prometheus 获取 RocketMQ 的 Metrics，您需要设置并运行一个 RocketMQ Exporter 项目。以下是基于提供的我了解的信息的具体步骤：

#### 2.1 配置 `application.yml`

在 `application.yml` 文件中，至少需要配置以下几项：
- `server.port`: 设置 Prometheus 监听 RocketMQ Exporter 的端口，默认为 5557。
- `rocketmq.config.webTelemetryPath`: Prometheus 获取 Metrics 的路径，默认为 `/metrics`。
- 如果 RocketMQ 开启了 ACL 验证，还需设置 `rocketmq.config.enableACL=true` 并填写相应的 `accessKey` 和 `secretKey`。
- `rocketmq.config.outOfTimeSeconds`: 存储指标的有效期，建议设置为 60 秒或更长，确保不低于 Prometheus 抓取间隔。
- `task..cron`: 定时任务的时间间隔，控制 Exporter 从 Broker 拉取指标的频率，默认每分钟一次。

#### 2.2 启动 Exporter 项目

按照常规 Java 应用的方式启动 RocketMQ Exporter 项目。

#### 2.3 配置 Prometheus

编辑 Prometheus 配置文件，添加静态目标配置，指向刚刚启动的 Exporter 地址，例如 `localhost:5557`。

#### 2.4 访问 Prometheus 页面

启动 Prometheus 后，在浏览器访问 `http://localhost:9090` 可查看收集到的 Metrics 值。推荐结合 Grafana 使用以获得更好的可视化效果。

### 三、解释

上述步骤帮助您完成了从 RocketMQ 到 Prometheus 的数据链路搭建。首先，通过修改配置文件让 Exporter 正确连接到 RocketMQ 集群并定期采集数据；然后，通过调整 Prometheus 设置来抓取这些公开的数据点；最后，利用 Prometheus 或 Grafana 等工具对收集的数据进行展示与分析。这样，您可以实时监测 RocketMQ 的运行状况，及时发现潜在问题并采取相应措施。


<font color="#949494">---------------</font> 


## 参考链接 ：

* 专家经验：rocketmq Metrics 
 
 * 专家经验：RocketMQ Promethus Exporter 
 
 * 专家经验：RocketMQ版本 


 <font color="#949494">---------------</font> 
 


## <font color="#FF0000">答疑服务说明：</font> 

本内容经由技术专家审阅的用户问答的镜像生成，我们提供了<font color="#FF0000">专家智能答疑服务</font>,使用方法：
用法1： 在<font color="#FF0000">页面的右下的浮窗”专家答疑“</font>。
用法2： 点击[专家答疑页](https://answer.opensource.alibaba.com/docs/intro)（针对部分网站不支持插件嵌入的情况）
### 另：


有其他开源产品的使用问题？[点击访问阿里AI专家答疑服务](https://answer.opensource.alibaba.com/docs/intro)。
### 反馈
如问答有错漏，欢迎点：[差评](https://ai.nacos.io/user/feedbackByEnhancerGradePOJOID?enhancerGradePOJOId=17250)给我们反馈。
