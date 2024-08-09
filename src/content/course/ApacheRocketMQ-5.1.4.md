---
title: "Apache RocketMQ 5.1.4"
description: "Apache RocketMQ 5.1.4"
date: "2024-08-08"
tags: ["version"]
author: ""
---

1. 多级存储模块优化，支持删除过期或者损坏的存储文件，优化消息读取、Topic 删除代码。 [#6952](https://github.com/apache/rocketmq/pull/6952)
2. 顺序消息 Pop 消费模式优化，可重入优化[#6755](https://github.com/apache/rocketmq/pull/6755)。
3. Pop 消费模式，支持批量消费确认，提升性能 [#7206](https://github.com/apache/rocketmq/pull/7206)
4. 在 gRPC 和 Remoting 基础上支持 Proxy 协议，获取客户端网络层信息。[#6958](https://github.com/apache/rocketmq/pull/6958) [#7062](https://github.com/apache/rocketmq/pull/7062)
5. 生产者 autobatch 消息发送，使得 RocketMQ 的消息吞吐量提升一个数量级，和 kafka 同一个水位。[#3718](https://github.com/apache/rocketmq/pull/3718)
6. 支持百万级队列元数据，基于 rocksdb 存储 RocketMQ 元数据。 [#7092](https://github.com/apache/rocketmq/pull/7092)

更多细节详见[https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.1.4](https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.1.4)
