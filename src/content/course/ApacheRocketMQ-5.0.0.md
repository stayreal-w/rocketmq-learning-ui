---
title: "Apache RocketMQ 5.0.0"
description: "Apache RocketMQ 5.2.0"
date: "2022-09-22"
tags: ["version"]
author: ""
---

2022年9月22号发布
# Highlight 

1. Pop consumer发布，支持无队列负载均衡模式，是5.0轻量SDK的基础。[RIP19](https://github.com/apache/rocketmq/wiki/%5BRIP-19%5D-Server-side-rebalance,--lightweight-consumer-client-support)
2. 秒级定时消息发布。[RIP43](https://docs.google.com/document/d/1D6XWwY39p531c2aVi5HQll9iwzTUNT1haUFHqMoRkT0/edit)
3. 5.0轻量gRPC sdk首发 [RIP37](https://shimo.im/docs/m5kv92OeRRU8olqX)。
4. 三层存算分离架构发布，Proxy+Broker+云存储-对象存储。多协议网关Proxy首发[RIP39](https://shimo.im/docs/gXqmeEPYgdUw5bqo)；基于云存储-对象存储的存算分离[RIP57](https://github.com/apache/rocketmq/wiki/RIP-57-Tiered-storage-for-RocketMQ)
5. 5.0新HA发布，数据面和控制面HA分离，引入Dledger Controller。[RIP44](https://github.com/apache/rocketmq/wiki/RIP-44-Support-DLedger-Controller)
6. 物联网消息[MQTT](https://github.com/apache/rocketmq-mqtt)首发。[RIP33](https://github.com/apache/rocketmq/wiki/RIP-33-RocketMQ-MQTT)
7. 事件总线[EventBridge](https://github.com/apache/rocketmq-eventbridge)首发。[RIP38](https://docs.google.com/document/d/1RWPeORHY_-ukq8qs1a1lH80fH8vSQ44U1R9xbxgEX_c/)
8. 逻辑队列发布，支持static topic，固定分区数弹性伸缩。[RIP21](https://github.com/apache/rocketmq/wiki/RIP-21-logical-queue-abstraction-for-static-topic-and-fast-scale-out)

更多细节详见[https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.0.0](https://github.com/apache/rocketmq/releases/tag/rocketmq-all-5.0.0)
