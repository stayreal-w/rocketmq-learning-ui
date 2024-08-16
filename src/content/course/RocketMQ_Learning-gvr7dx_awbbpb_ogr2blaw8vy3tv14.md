---
title: "Apache RocketMQ 权威部署指南"
description: "Apache RocketMQ 权威部署指南"
date: "2024-08-16"
tags: ["deploy"]
author: ""
---

<a name="ZDHfC"></a>
## 一、前言
RocketMQ 5.0 包含多种组件，不仅包含如 Broker, NameServer 等 4.0 经典组件，还新增了多种功能组件，以支持多样的部署形态，适应复杂的部署场景。

本文将对 RocketMQ 5.0 中常用的部署组件作介绍，并提供多种部署方式的详细指南。若需要了解迅速部署的 quick start 系列文章，可以直接阅读 **[**Quick Start迅速部署**](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_nse74d9gfuq3c5f0/)** 系列文章。
<a name="JcDZC"></a>
## 二、常用部署组件
<a name="J781w"></a>
### 2.1 Broker
**Broker** 是 RocketMQ 的核心服务节点，负责接收来自 Producer（生产者）的消息、存储消息以及将消息转发给 Consumer（消费者）。在 RocketMQ 架构中，Broker 分为 Master Broker 和 Slave Broker（或称为 Replica）两种角色：

- **Master Broker**：处理消息的读写请求，是生产者发送消息和消费者消费消息的主要目标。
- **Slave Broker**：作为 Master Broker 的副本存在，主要用于备读或者故障切换，以提高系统的可用性。在 Master Broker出故障时，Slave Broker 里的消息依然可以被消费，或者直接被提升为新的 Master 继续提供服务。

Broker 的部署可以是单节点的，也可以是集群式的，甚至支持同机器、同进程混布的模式。<br />其具体部署形式可以参考 [**Broker 部署指南**](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_bmpnil7eq36uy5fn/)。
<a name="Mk0pV"></a>
### 2.2 NameServer
**NameServer** 是 RocketMQ 的路由服务集群，它不保存任何消息数据，而是维护着所有 Broker 的路由信息（包括 Broker 存活情况、各 topic 的路由信息等）。Producer 和 Consumer 在发送或订阅消息前，都需要先从 NameServer 获取 Broker 的路由信息，之后才能正确地与 Broker 通信。NameServer 支持集群部署，以确保高可用。

该组件的部署相对简单，可以参见文档 [**NameServer 部署指南**](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_tncndnkqzud0055o/)。
<a name="AYI26"></a>
### 2.3 Proxy
**Proxy** 是 RocketMQ 为了提高性能和简化客户端接入而引入的一个可选组件。它作为一个轻量级的代理服务器，位于客户端与 Broker 之间，主要职责包括：

- **负载均衡**：自动为客户端分配合适的 Broker，减轻客户端的负担。
- **协议转换**：支持多种协议接入，使得不同语言的客户端能够更容易地与 RocketMQ 集成。
- **安全控制**：可以作为一层安全网关，实现访问控制、鉴权等功能。
- **网络优化**：通过缓存、连接复用等技术优化网络通信效率。

该组件的部署相对独立，可以选用不同模式进行部署，如 Local 模式以及 Cluster 模式，你可以参考[**Proxy 部署指南**](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_xa4fgvpbunrvehbf/)进行部署。
<a name="nUMfI"></a>
### 2.4 Controller
**Controller**是 RocketMQ 5.0 新增的组件，它充当控制平面的角色，负责管理和协调系统的整体状态。它主要出现在可切换架构的部署过程中，如 Broker 宕机时，它将进行调度，对 Broker 集群进行选举管理。Controller 本身也支持集群部署，基于 Raft 实现容灾。

该组件的部署可以参考[**Broker 部署指南**](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_bmpnil7eq36uy5fn/)中的主备自动切换模式部署指南章节。
<a name="GYq1q"></a>
### 2.5 Dashboard
**RocketMQ Dashboard** 是 RocketMQ 提供的一款可视化管理界面，它允许用户通过网页浏览器直观地监控和管理 RocketMQ 集群的状态，包括但不限于查看消息队列、消费进度、Broker 健康状况、主题配置、消费组详情等信息。Dashboard 的存在大大提升了运维人员对 RocketMQ 集群的监控能力和管理效率。

该组件的部署可以参考[**DashBoard 部署指南**](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_fn65r094u26he62t/)。

<a name="pjloM"></a>
## 三、常见部署形态介绍
基于上述组件，RocketMQ 可以满足多样化的部署形态。如以下几种：
<a name="oJLh7"></a>
### 3.1 直连模式（单节点）
单节点直连模式是最简单的部署形态，单节点指单 Broker 节点。这种部署方式能满足基本的消息收发需求，但无法提供高可用、存算分离等能力。

其基本部署结构如下图所示：

![](https://img.alicdn.com/imgextra/i1/O1CN011d3wBr1muvFd12fjG_!!6000000005015-0-tps-1320-554.jpg)
<a name="YorZS"></a>
### 3.2 主备无切换模式
基于单节点模式，我们可以启动多台相同名字的 Broker，并设置其角色为备机，则可以实现最简单的主备模式。这种模式不具有切换能力，但是备节点可以在主节点宕机时提供一定读能力。经过合理配置，也可以提供一些备代理主、二级消息逃逸的能力。

这种部署模式的结构图如下所示：

![](https://img.alicdn.com/imgextra/i2/O1CN0102DqWQ1KUXBltN4B3_!!6000000001167-0-tps-1320-681.jpg)
<a name="alTNz"></a>
### 3.3 Proxy 本地代理模式
此前两种部署模式主要面向 Broker，如果不希望客户端直接与 Broker 进行交互，或者需要更先进的存算分离、协议转换、负载均衡等能力，则需要部署 Proxy 组件。

若部署了 Proxy，则生产者、消费者将直接与 Proxy 进行交互，避免直接与 Broker/NameServer 进行交互。Proxy 内置的生产、消费逻辑会优雅地处理路由感知、负载均衡等事宜。

Proxy 的部署模式支持本地部署与集群部署，本地代理模式的结构图如下所示：

![](https://img.alicdn.com/imgextra/i2/O1CN016e6MZr27D3F7UeVNU_!!6000000007762-0-tps-1464-825.jpg)
<a name="DLTt4"></a>
### 3.4 Proxy 集群代理模式
若希望 Proxy 与 Broker 部署分离，则可以采用如下的部署结构：

![](https://img.alicdn.com/imgextra/i4/O1CN01aakmw91z2VC8QjG3t_!!6000000006656-0-tps-1868-813.jpg)

这种部署方式能够提供非常灵活的弹性能力。因为 Proxy 是“无状态”的，这意味着它能够直接扩容与缩容，而不用像 Broker 一样考虑数据面的写入、消费问题。
<a name="JNio7"></a>
### 3.5 基于 Controller 的自动切换模式
上述的部署形态均不提供切换能力，因此在 Broker 遇到故障时，只有原 Broker 自动恢复且启动成功才能彻底解决问题，否则该 Broker 只能通过备节点提供一定读能力。由于可写分区少了，可能导致集群的写入能力受损。为避免这种问题，可以部署 Controller 组件，从而使 Broker 具备自动的主备切换能力。

带有 Controller 组件的部署形态一般如下所示：

![](https://img.alicdn.com/imgextra/i3/O1CN01dwznex1XdJn85tp6w_!!6000000002946-2-tps-1432-619.png)

这个部署过程仅涉及到 Broker 的配置，不影响其它组件（如 Proxy/NameServer）部署。您依然可以选择使用直连模式，或者采用 Proxy 代理模式进行部署。
<a name="lQ6Wi"></a>
### 3.6 BrokerContainer 模式
BrokerContainer 是一种特殊的部署形态，它不同于传统的主备模式，它是一种独属于 RocketMQ 的混布形式。通过这种部署方式，我们能够实现一台机器上混布多台 Broker。这种混布是在同一个进程内实现的，因此能够非常高效地完成 Broker 间的数据共享。

BrokerContainer 的部署结构一般如下所示：

![](https://img.alicdn.com/imgextra/i2/O1CN01u6PerA1WVhaZa2CuS_!!6000000002794-0-tps-1738-931.jpg)

在这个部署方式下，等于一个机器的两个不同端口号分别扮演了两个 Broker。且拥有 Broker 的全量能力。这种部署方式能够节约资源，且避免备机资源利用率低的问题。
