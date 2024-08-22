---
title: "Apache RocketMQ 5.2.0"
description: "Apache RocketMQ 5.2.0"
date: "2024-02-14"
tags: ["version"]
author: ""
---

2024年2月14号发布
<a name="kRDxa"></a>
# Highlight 

1. 发布百万级队列实现，基于 rocksdb 实现 consume queue，RocketMQ 单机 topic、队列数从十万级提升到百万级。更适用于物联网海量队列、移动 APP 的场景。[详见](https://github.com/apache/rocketmq/issues/7064)
2. 在多级存储的基础上，实现面向海量消息规模的消息索引能力。使得多级存储的消息和本地消息都具备同样的索引能力，消息查询能力。[详见](https://github.com/apache/rocketmq/issues/7545)
3. 5.0 HA 升级，基于 Jraft 的 HA controller，具备更完整、更成熟的高可用能力。[详见](https://github.com/apache/rocketmq/issues/7300)
4. 支持多级存储更丰富的消费能力，新增 tag 过滤和 sql 过滤消费模式。
5. 无损变更能力增强，客户端增加优雅断开和重连机制。

更多细节详见 [https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.2.0](https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.2.0)
