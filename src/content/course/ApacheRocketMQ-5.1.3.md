---
title: "Apache RocketMQ 5.1.3"
description: "Apache RocketMQ 5.1.3"
date: "2023-06-24"
tags: ["version"]
author: ""
---

2023年6月24号发布
<a name="kRDxa"></a>
# Highlight 

1. 心跳优化，解决大规模共享集群，客户端连接数过多，处理连接心跳消耗大量 CPU 的问题。[#6724](https://github.com/apache/rocketmq/pull/6724)
2. SQL 过滤订阅增强，增加 Contains 表达式，支持 startsWith/endsWith 特性，满足更多精细化订阅的场景。[#6864](https://github.com/apache/rocketmq/pull/6864) [#6915](https://github.com/apache/rocketmq/pull/6915)
3. PushConsumer 性能优化，支持批量消费确认能力 batch ack，提升消费性能。[#6842](https://github.com/apache/rocketmq/pull/6842)

更多细节详见[https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.1.3](https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.1.3)

