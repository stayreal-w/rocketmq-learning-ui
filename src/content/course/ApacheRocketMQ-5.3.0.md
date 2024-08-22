---
title: "Apache RocketMQ 5.3.0"
description: "Apache RocketMQ 5.3.0"
date: "2024-07-17"
tags: ["version"]
author: ""
---

2024年7月17号发布
<a name="kRDxa"></a>
# Highlight 

1. RocketMQ 权限认证体系进行了全面升级，发布了 ACL 2.0。
2. 在多级存储基础上提供了 Topic 粒度的 TTL，支持更细粒度的消息保留时长。
3. 消费链路性能优化，针对多级存储的 tag 消费模式、长轮询 Pop 的 SQL 过滤消费模式进行性能优化，提升数倍性能。
4. 增强可观测能力，新增多项 metric，包括 rocketmq_topic_number、rocketmq_consumer_group_number、rocketmq_topic_create_execution_time 等


更多细节详见 [https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.3.0](https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.3.0)
