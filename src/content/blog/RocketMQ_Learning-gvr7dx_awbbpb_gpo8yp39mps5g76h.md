---
title: "深度剖析 RocketMQ 5.0 之架构解析：云原生架构如何支撑多元化场景？"
description: "深度剖析 RocketMQ 5.0，架构解析：云原生架构如何支撑多元化场景？"
date: "2024-08-09"
tags: ["explore"]
author: "隆基"
img: "https://img.alicdn.com/imgextra/i1/O1CN01KYPkWt1Eg5U8v02i9_!!6000000000380-2-tps-596-360.png"
---

作者 | 隆基

> 简介： 了解 RocketMQ 5.0 的核心概念和架构概览；然后我们会从集群角度出发，从宏观视角学习 RocketMQ 的管控链路、数据链路、客户端和服务端如何交互；学习 RocketMQ 如何实现数据的存储，数据的高可用，如何利用云原生存储进一步提升竞争力。

<a name="slide-0"></a>
## 1.前言
从初代开源消息队列崛起，到 PC 互联网、移动互联网爆发式发展，再到如今 IoT、云计算、云原生引领了新的技术趋势，消息中间件的发展已经走过了 30 多个年头。

目前，消息中间件在国内许多行业的关键应用中扮演着至关重要的角色。随着数字化转型的深入，客户在使用消息技术的过程中往往同时涉及交叉场景，比如同时进行物联网消息、微服务消息的处理，同时进行应用集成、数据集成、实时分析等，企业需要为此维护多套消息系统，付出更多的资源成本和学习成本。

在这样的背景下，2022 年，RocketMQ 5.0 正式发布，相对于 RocketMQ 4.0，架构走向云原生化，并且覆盖了更多的业务场景。想要掌握最新版本 RocketMQ 的应用，就需要进行更加体系化的深入了解。

<a name="slide-1"></a>
## 2.背景
本节课的内容是 RocketMQ 5.0 的架构解析。前面的课程中，我们了解到 RocketMQ 5.0 可以支撑多样化的业务场景，不仅仅是业务消息，它还会支持流处理、物联网、事件驱动等场景。在进入具体的业务领域场景之前，我们先从技术的角度来了解 RocketMQ 的云原生架构，看它是如何基于这一套统一的架构支撑多元化场景的。

首先，我们会了解 RocketMQ 5.0 的核心概念和架构概览；然后我们会从集群角度出发，从宏观视角学习 RocketMQ 的管控链路、数据链路、客户端和服务端如何交互；最后，我们将回到消息队列最重要的模块存储系统，学习 RocketMQ 如何实现数据的存储，数据的高可用，如何利用云原生存储进一步提升竞争力。

<a name="FbEpk"></a>
## 3. 概览
<a name="slide-3"></a>
### 3.1. RocketMQ 领域模型

在学习 RocketMQ 的架构之前，我们先从用户视角来来看 RocketMQ 的关键概念以及领域模型。如下图，我们按照消息的流转顺序来介绍。

![](https://img.alicdn.com/imgextra/i4/O1CN01dFrTsK1vvE3F1Q2wW_!!6000000006234-0-tps-750-422.jpg)

最左边是消息生产者，一般对应业务系统的上游应用，在某个业务动作触发后发送消息到 Broker。Broker 是消息系统数据链路的核心，负责接收消息、存储消息、维护消息状态、消费者状态。多个 Broker 组成一个消息服务集群，共同服务一个或者多个 Topic 。刚才提到生产者生产消息并发送到 Broker，消息是业务通信的载体，每个消息包含消息 ID、消息 Topic、消息体内容、消息属性、消息业务 key 等。每条消息都属于某个 Topic，表示同一个业务语义，在阿里内部，我们交易消息的 Topic 叫做 Trade，购物车消息叫 Cart，生产者应用会把消息发送到对应的 Topic 上。Topic 里面还有 MessageQueue，这个用于消息服务的负载均衡和数据存储分片，每个 Topic 会包含一个或者多个 Message Queue 分布在不同的消息 Broker。生产者发送消息，Broker 存储消息，下一步就是消费者消费消息。消费者一般对应业务系统的下游应用，同一个消费者应用集群会共用一个 Consumer  Group。消费者会和某个 Topic 产生订阅关系，订阅关系是 Consumer Group + Topic + 过滤表达式的三元组，符合订阅关系的消息就会被对应的消费者集群消费。接下来我们从技术实现角度进一步深入了解 RocketMQ 。

<a name="slide-4"></a>
### 3.2. RocketMQ 5.0 架构概览

这是 RocketMQ 5.0 的架构概览图，从上往下看，可分为 SDK、NameServer、Proxy 和 Store 层。

![](https://img.alicdn.com/imgextra/i2/O1CN01XpWeHO1PV3ij1oot5_!!6000000001845-0-tps-750-422.jpg)

我们首先来看 SDK 层，包括了 RocketMQ 的 SDK ，用户基于 RocketMQ 自身的领域模型来使用这个 SDK 。除了 RocketMQ 自身的 SDK 之外，还包括了细分领域场景的业界标准 SDK 。面向事件驱动的场景，RocketMQ 5.0 支持 CloudEvents 的 SDK；面向 IoT 的场景，RocketMQ 支持物联网 MQTT 协议的 SDK；为了方便更多的传统应用迁移到 RocketMQ，我们还支持了 AMQP 协议，未来也会开源到社区版本里。另外一个组件是是 NameServer，它承担服务发现和负载均衡的职责。通过 NameServer，客户端能获取 Topic 的数据分片和服务地址，链接消息服务器进行消息收发。

消息服务包含计算层 Proxy 和存储层 RocketMQ Store。RocketMQ 5.0 是存算分离的架构，这里的存算分离强调的是模块的分离，职责的分离。Proxy 和 RocketMQ Store 面向不同的业务场景可以合并部署，也可以分开部署。计算层 Proxy 主要承载的消息的上层业务逻辑，尤其是面向多场景、多协议的支持，比如承载 CloudEvents、MQTT、AMQP 的领域模型的实现逻辑和协议转换。面向不同的业务负载，还可以把 Proxy 分离部署，独立弹性，比如在物联网场景，Proxy 层独立部署可以面向海量物联网设备连接数进行弹性伸缩，和存储流量扩缩容解耦。RocketMQ Store 层则是负责核心的消息存储，这里包括基于 Commitlog 的存储引擎、多元索引、多副本技术和云存储集成扩展。消息系统的状态都下沉到 RocketMQ Store，其他组件全部实现无状态化。

<a name="slide-5"></a>
## 4. 服务发现

<a name="slide-6"></a>
### 4.1. 服务发现


第二部分我们来详细看一下 RocketMQ 的服务发现。RocketMQ 的服务发现是通过 NameServer（简称NS） 来实现的。

我们通过下方这个图来了解服务发现的机制，这个是 Proxy 和 Broker 合并部署的模式，也是 RocketMQ 最常见的模式。前面提到每个 Broker 集群会负责某些 Topic 的服务，每个 Broker 都会把自身服务哪些 Topic 注册到 NameServer 集群，和每个 NameServer 进行通信，并定时和 NS 通过心跳机制来维持租约。服务注册的数据结构包含 Topic 和 Topic 分片 MessageQueue。

![](https://img.alicdn.com/imgextra/i2/O1CN017fyjmL1hHvRopC7YX_!!6000000004253-0-tps-750-422.jpg)

在示例中 Broker1 和 Broker2 分别承载 TopicA 的一个分片。在 NS 机器上会维护全局视图，TopicA 有两个分片分别在 Broker1 和 Broker2 。RocketMQ SDK 在对 TopicA 进行正式的消息收发之前，它会随机访问一个 NameServer 机器，从而知道这个 TopicA 有哪些分片，每个数据的分片在哪个 Broker 上面，它会跟这些 Broker 建立好长连接，然后再进行消息的收发。大部分的项目的服务发现机制会通过 zookeeper 或者 etcd 等强一致的分布式协调组件来担任注册中心的角色，而 RocketMQ 有自己的特点，如果从 CAP 的角度来看，它的注册中心采用的是 AP 的模式，NameServer 节点无状态，是 ShareNothing 的架构，有更高的可用性。

再看下方这个图，我们说 RocketMQ 的存算分离是可分可合，这里采用的就是分离的部署模式，RocketMQ  SDK 直接访问无状态的 Proxy 集群。这个模式可以应对更加复杂的网络环境，支持多网络类型的访问，如公网访问，实现更好的安全控制。

![](https://img.alicdn.com/imgextra/i1/O1CN01qSGRrp1pmAWk7Rk2V_!!6000000005402-0-tps-750-422.jpg)

在整个服务发现机制中，NameServer、Proxy 都是无状态的，可以随时进行节点增减。有状态节点 Broker 的增减基于 NS 的注册机制，客户端可以实时感知、动态发现。在缩容过程中，RocketMQ Broker 还可以进行服务发现的读写权限控制，对缩容的节点禁写开读，待未读消息全消费，实现无损平滑下线。

<a name="slide-7"></a>
### 4.2. 负载均衡

刚才我们已经知道 SDK 如何通过 NameServer 来发现 Topic 的分片信息 MessageQueue，以及 Broker 地址。基于这些服务发现的元数据，我们再来详细看看消息流量是如何在生产者、RocketMQ Broker 和消费者集群进行负载均衡的。

![](https://img.alicdn.com/imgextra/i4/O1CN01o6jtTN1CApDbXPzqe_!!6000000000041-0-tps-750-422.jpg)

先来看生产链路的负载均衡，生产者通过服务发现机制，知道了 Topic 的数据分片以及对应的 Broker 地址。它的服务发现机制是比较简单的，在默认情况下采用 Round Robin 的方式轮询发送到各个 Topic 队列，保证了 Broker 集群的流量均衡。在顺序消息的场景下会略有特殊，会基于消息的业务主键 Hash 到某个队列发送，这样一来，如果有热点业务主键，那 Broker 集群也可能出现热点。除此之外，我们基于这些元数据还能根据业务需要扩展更多的负载均衡算法，比如同机房优先算法，可以降低多机房部署场景下的延迟，提升性能。

![](https://img.alicdn.com/imgextra/i2/O1CN01PiV7jQ22FH5NxUXkI_!!6000000007090-0-tps-750-422.jpg)

再看消费者的负载均衡，相对来说会比生产者更复杂，它有两种类型的负载均衡方式。最经典的模式是队列级负载均衡，消费者知道 Topic 的队列总数，也知道同一个 Consumer Group 下的实例数，就可以按照统一的分配算法，类似一致性 hash 的方式，让每个消费者实例绑定对应的队列，只消费绑定队列的消息，每个队列的消息也只会被一个消费者实例消费。

这种模式最大的缺点就是负载不均衡，消费者实例要绑定队列、有临时状态。如果我们有三个队列，有两个消费者实例，那就必然有一个消费者需要消费三分之二的数据，如果我们有四个消费者，那么第四个消费者就要空跑。所以在 RocketMQ 5.0 里面，我们引入了消息粒度的负载均衡机制，无需绑定队列，消息在消费者集群随机分发，这样就可以保障消费者集群的负载均衡。更重要的是这种模式更加符合未来 Serverless 化的趋势，Broker 的机器数、Topic 的队列数和消费者实例数完全解耦，可以独立扩缩容。

![](https://img.alicdn.com/imgextra/i1/O1CN01oLKjMi1bYyUlJpmbF_!!6000000003478-0-tps-750-422.jpg)

<a name="slide-8"></a>
## 5. 存储系统

前面通过架构概览和服务发现机制，我们已经对 RocketMQ 有比较全局性的了解。接下来我们将深入 RocketMQ 的存储系统，这个模块对 RocketMQ 的性能、成本、可用性有决定性作用。

<a name="slide-9"></a>
### 5.1. 存储核心

先来看一下 RocketMQ 的存储核心。存储核心由 Commitlog、Consumequeue 和 Index 文件组成。消息存储首先写到 Commitlog，刷盘并复制到 slave 节点来完成持久化，Commitlog 是 RocketMQ 存储的 source of true，通过它可以构建完整的消息索引。相比于 Kafka 而言，RocketMQ 把所有 Topic 的数据都写到 Commitlog 文件，最大化顺序 io，使得 RocketMQ 单机可以支撑万级的 Topic。

在写完 Commitlog 之后，RocketMQ 会异步分发出多个索引，首先是 ConsumeQueue 索引，这个和 MessageQueue 是对应的，基于这个索引可以实现消息的精准定位，可以按照 Topic、队列 id 和位点定位到消息，消息回溯功能也是基于这个实现的。另外一个很重要的索引是哈希索引，它是消息可观测的基础。通过持久化的 hash 表来实现消息业务主键的查询能力，消息轨迹主要是基于这个来实现的。

除了消息本身的存储之外，Broker 还承载了消息元数据的存储。包括 topics 的文件，表示该 Broker 会对哪些 Topic 提供服务，还维护了每个 Topic 队列数、读写权限、顺序性等属性。还有一个 Subscription、ConsumerOffset 文件，这两个维护了 Topic 的订阅关系以及每个消费者的消费进度。还有 Abort、Checkpoint 文件则是用于完成重启后的文件恢复，保障数据完整性。

![](https://img.alicdn.com/imgextra/i4/O1CN01tBuTwZ1NHetwiZIyf_!!6000000001545-0-tps-750-422.jpg)

<a name="slide-10"></a>
### 5.2. Topic 高可用

上面的内容中，我们站在单机的视角，从功能的层面学习 RocketMQ 的存储引擎，包括 Commitlog 和索引。现在我们重新跳出来，再从集群视角看 RocketMQ 的高可用。我们先定义一下 RocketMQ 的高可用，指当 RocketMQ 集群出现 NameServer、Broker 局部不可用的时候，指定的 Topic 依然是可读可写的。

![](https://img.alicdn.com/imgextra/i2/O1CN01Nj6zJm274o763otMo_!!6000000007744-0-tps-750-422.jpg)

RocketMQ 可以应对三类故障场景。

第一种 case，某对主备单机不可用。如下方这个图，当 Broker2 主宕机，备可用。TopicA 依然可读可写，其中分片1可读可写，分片 2 可读不可写，Topic A 在分片 2 的未读消息依然可以消费。总结起来就是 Broker 集群里，只要任意一组 Broker 存活一个节点，Topic 的读写可用性不受影响。如果某组 Broker 主备全部宕机，那么 Topic 新数据的读写也不受影响，未读消息会延迟，待任意主备启动才能继续消费。

![](https://img.alicdn.com/imgextra/i1/O1CN014kEgPt1SO8aBRb4tl_!!6000000002236-0-tps-750-422.jpg)

接下来，再看 NameServer 集群的故障情况，由于 NameServer 是 ShareNothing 的架构，每个节点都是无状态的，并且是 AP 模式，不需要依赖多数派算法，所以只要有一台 NameServer 存活，整个服务发现机制都是正常的，Topic 的读写可用性不受影响。

![](https://img.alicdn.com/imgextra/i3/O1CN01zsVyqZ1m12vfwOdOU_!!6000000004893-0-tps-750-422.jpg)

甚至在更极端的情况下，整个 NS 都不可用，由于 RocketMQ 的 SDK 对服务发现元数据有缓存，只要 SDK 不重启，它依然可以按照当下的 topic 元数据，继续进行消息收发。

![](https://img.alicdn.com/imgextra/i4/O1CN01UOVs7v1yTh5ldiRV0_!!6000000006580-0-tps-750-422.jpg)

<a name="slide-11"></a>
### 5.3. MessageQueue 高可用

从 Topic 高可用的实现中我们发现，虽然 Topic 持续可读可写，但是 Topic 的读写队列数会发生变化。队列数变化，会对某些数据集成的业务有影响，比如说异构数据库 Binlog 同步，同一个记录的变更 Binlog 会写入不同的队列，重放 Binlog 可能会出现乱序，导致脏数据。所以我们还需要对现有的高可用进一步增强，要保障局部节点不可用时，不仅 Topic 可读可写，并且 Topic 的可读写队列数量不变，指定的队列也是可读可写的。

如下图，NameServer 或 Broker 任意出现单点不可用，Topic A 依然保持 2 个队列，每个队列都具备读写能力。

![](https://img.alicdn.com/imgextra/i2/O1CN01Nf3uTt1tylt7EuvVR_!!6000000005971-0-tps-750-422.jpg)

为了解决 MessageQueue 高可用的场景，RocketMQ 5.0 引入全新的高可用机制。我们先来了解其中的核心概念：

- Dledger Controller，这是一个基于 raft 协议的强一致元数据组件，来执行选主命令、维护状态机信息。
- SynStateSet，如图，它维护了处于同步状态的副本组集合，这个集合里的节点都有完整的数据，当主节点宕机后，就从这个集合中选择新的主节点。
- Replication，用于不同副本之间的数据复制、数据校验、截断对齐等事项。

![](https://img.alicdn.com/imgextra/i3/O1CN01yX5Ote1a6kTJtCYvI_!!6000000003281-0-tps-750-422.jpg)

下图是 RocketMQ 5.0 HA 的架构全景图，这个高可用架构具有多个优势。

![](https://img.alicdn.com/imgextra/i1/O1CN01u0RAS91C5KSt4Zskw_!!6000000000029-0-tps-750-422.jpg)

**一是**在消息存储引入了朝代和开始位点，基于这两个数据，完成数据校验、截断对齐，在构建副本组的过程中简化数据一致性逻辑。

**二是**基于 Dledger Controller，我们不需要引入 zk、etcd 等外部分布式一致性系统，并且 Dledger Controller 还可以和 NameServer 合并部署，简化运维、节约机器资源。

**三是** RocketMQ 对 Dledger Controller 是弱依赖，即便 Dledger 整体不可用了，也只会影响选主，不影响正常的消息收发流程。

**四是**可定制，用户可以根据业务对数据可靠性、性能、成本综合选择，比如副本数可以是2、3、4、5，副本直接可以是同步复制、异步复制。如 2-2 模式表示，2 副本、并且数据同步复制；2-3 模式表示3副本，2副本多数派完成复制，才算成功。用户还可以将其中的一个副本部署在异地机房，异步复制实现容灾。

![](https://img.alicdn.com/imgextra/i1/O1CN01YQ09gK1c8EzVXxhBI_!!6000000003555-0-tps-750-422.jpg)

<a name="slide-12"></a>
### 5.4. 云原生存储

前面我们讲的存储系统都是 RocketMQ 面向本地文件系统的实现。但是在云原生时代，当我们把 RocketMQ 部署到云环境，可以进一步利用云原生基础设施，如云存储来进一步增强 RocketMQ 的存储能力。在 RocketMQ 5.0 里面我们提供了多级存储的特性，它是内核级的存储扩展，我们面向对象存储扩展了对应的 Commitlog、ConsumeQueue 和 IndexFile；我们采用了插件化的设计，多级存储可以有多种实现，在阿里云上，我们基于 OSS 对象服务实现，在 AWS 上我们则可以面向 S3 的接口来实现。

通过引入了这个云原生的存储，RocketMQ 释放了很多红利：

- 无限存储能力，消息存储空间不受本地磁盘空间的限制，原来是保存几天，现在可以几个月、甚至存一年。另外对象存储也是业界成本最低的存储系统，特别适合冷数据存储。
- Topic 的 TTL，原来多个 Topic 的生命周期是和 Commitlog 绑定，统一的保留时间。现在每个 Topic 都会使用独立的对象存储 Commitlog 文件，可以有独立的 TTL。
- 存储系统进一步的存算分离，能把存储吞吐量的弹性和存储空间的弹性分离。
- 冷热数据隔离，分离了冷热数据的读链路，能大幅度提升冷读性能，不会影响在线业务。

![](https://img.alicdn.com/imgextra/i3/O1CN01tUbqw81diFkg1yPlE_!!6000000003769-0-tps-750-422.jpg)