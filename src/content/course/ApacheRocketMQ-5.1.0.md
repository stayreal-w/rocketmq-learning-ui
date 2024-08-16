---
title: "Apache RocketMQ 5.1.0"
description: "Apache RocketMQ 5.1.0"
date: "2023-02-20"
tags: ["version"]
author: ""
---

2023年2月20号发布
<a name="kRDxa"></a>
# Highlight 

1. Proxy 支持 Remoting 协议。 [#5575](https://github.com/apache/rocketmq/pull/5575)
2. Pop 顺序消息模式能力增强，支持动态修改不可见时间[#5367](https://github.com/apache/rocketmq/pull/5367)，增加通知机制，提升消费实时性[#5387](https://github.com/apache/rocketmq/pull/5387)。
3. Broker 启动加速，ConsumeQueue 并发加载。[#5093](https://github.com/apache/rocketmq/pull/5093)
4. 性能提升，事务消息 Compaction 批量优化。[#5386](https://github.com/apache/rocketmq/pull/5386)
5. 位点重置特性优化，支持服务端位点重置，支持广播消费模式、Pop 消费模式的重置场景。[#5293](https://github.com/apache/rocketmq/pull/5293)
6. 可观测增强，增加客户端连接、消费延迟、定时器、Pop、网络、存储指标
7. 兼容 RocketMQ MQTT 位点保存。[#5208](https://github.com/apache/rocketmq/pull/5208)
8. 增加 CompactTopic 删除策略。[#5260](https://github.com/apache/rocketmq/pull/5260)
9. 5.0 客户端增加 BrokerName 字段传输，弱化 RocketMQ 架构的 Broker 状态。[#5334](https://github.com/apache/rocketmq/pull/5334)
10. 网络模块优化，网络拥塞背压，快速失败；在 remoting 协议支持 RPC 响应时间分布统计，并输出日志； 网络异常日志输出优化；
11. 消息生命周期定义富化，增加就绪、inflight 状态。[#5357](https://github.com/apache/rocketmq/pull/5357)

更多细节详见[https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.1.0](https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.1.0)

