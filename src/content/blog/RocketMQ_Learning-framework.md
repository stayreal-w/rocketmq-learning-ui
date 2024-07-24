---
title: "RocketMQ 原理和架构"
description: "介绍 Apache RocketMQ 的核心原理和架构"
date: "2024-07-24"
img: "https://img.alicdn.com/imgextra/i2/O1CN01b7uXIi1bmFycwldBc_!!6000000003507-2-tps-498-220.png"
tags: ["baseLearn"]
author: "隆基"
---

Apache RocketMQ 是用于消息传递场景中高吞吐量、低延迟、高可用的分布式中间件，被广泛应用于搜索、社交网络活动流、数据管道、交易流程、物联网等领域，本文介绍 Apache RocketMQ 的核心原理和架构。
<a name="bODIv"></a>
## 领域模型
以下是 RocketMQ 中的术语，更多细节请参考 [基本概念](https://rocketmq.apache.org/zh/docs/introduction/02concepts)。

- Message（消息）：Message 是 RocketMQ 传输的基本单元，包含了具体的业务数据以及一些元数据（如消息 ID、主题、标签、发送时间等）。消息可以是文本、二进制数据或其他任何序列化后的对象形式。
- Topic（主题）：Topic 是一类消息的逻辑分类名，是 Apache RocketMQ 中消息传输和存储的顶层容器。类似于邮件系统中的邮箱地址或发布/订阅模式中的“频道”。生产者向特定的 Topic 发送消息，消费者则根据 Topic 订阅并接收消息。一个 Topic 可以被多个生产者写入，同时也能被多个消费者订阅。
- Queue（队列）：每个 Topic 被划分为多个 Queue（队列），或称 MessageQueue，这些队列用于存储消息。生产者发送到 Topic 的消息会被分配到其下的各个 Queue 中；消费者则是从这些 Queue 中拉取消息进行消费。
- Subscription（订阅）：Subscription 表示消费者对某个 Topic 消息的兴趣表达。订阅关系由消费者分组动态注册到服务端系统，并在消息传输中按照订阅关系定义的过滤规则进行消息匹配和消费进度的维护。
- Producer（生产者）：生产者是消息产生的源头，将消息发送到服务端指定 Topic。
- Consumer（消费者）：消费者负责从服务端中拉取消息并进行处理。
- ProducerGroup（生产者组）：ProducerGroup 是一组生产者的逻辑分组，共享同样的 Topic 发送配置，实现发送端的负载均衡和容错。如果组内某个生产者失败，其他生产者可以继续工作，保证消息发送的连续性。
- ConsumerGroup（消费者组）：消费者分组是 Apache RocketMQ 系统中承载多个消费行为一致的消费者的负载均衡分组。和消费者不同，消费者分组并不是运行实体，而是一个逻辑资源。分组中的消费者共同订阅同一个 Topic 并以某种策略（如广播、集群消费）消费消息。在 Apache RocketMQ 中，通过消费者分组内初始化多个消费者实现消费性能的水平扩展以及高可用容灾。
<a name="c7zM7"></a>
## 技术架构
Apache RocketMQ 服务端基础组件包括 NameServer，Broker，Proxy，推荐使用存储计算分离模式部署。
<a name="Uuyso"></a>
### 直连模式部署
RocketMQ 是一个典型的发布订阅系统，通过 Broker 节点中转和持久化数据，解耦上下游。Broker 是真实存储数据的节点，由多个水平部署但不一定完全对等的副本组构成，单个副本组的不同节点上的数据会达到最终一致。单个副本组同一时间只有一个可读写的 Master 和若干个只读的 Slave，主故障时会进行选举来容忍故障，此时单个副本组可读不可写。NameServer 是独立的一个无状态组件，接受 Broker 的元数据注册并动态维护着一些映射关系，同时为客户端提供服务发现的能力。在这个模型中，我们使用不同主题 (Topic) 来区分不同类别的信息流，为消费者设置订阅组 (Group) 进行更好的管理与负载均衡。如下图中间部分所示：

1. 服务端 Broker Master1 和 Slave1 构成其中的一个副本组。
2. 服务端 Broker 1 和 Broker 2 两个副本组以负载均衡的形式共同为客户端提供读写。

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/213145/1721631114262-daa5ff53-c0f9-4dc9-8764-a56abb6f807e.png#clientId=uafe8ad5f-3e17-4&from=paste&height=370&id=udc4ee774&originHeight=740&originWidth=1510&originalType=binary&ratio=2&rotation=0&showTitle=false&size=532783&status=done&style=none&taskId=ufe2c8bbe-64d0-4784-99bf-1f1050fc013&title=&width=755)<br />注：Producer 和 Consumer 会和 NameServer，Broker 都维持长连接。Producer 只会向 Master 副本发送消息，Consumer 可以从 Master 或者 Slave 消费消息。
<a name="xdWNn"></a>
### 存储计算分离部署
存储和计算分离是一种良好的模块化设计。无状态的 Proxy 集群是数据流量的入口，提供签名校验与权限控制、计量与可观测、客户端连接管理、消息编解码处理、流量控制、多协议接入等能力。原 Broker 节点演化为以存储为核心的有状态集群，支持读写多类型消息，它们的底层是多模态存储和多元化的高效索引。存储计算分离的形态利于不同业务场景下单独调整存储或计算节点的数量，来实现扩容和缩容。网关模式接入还能带来升级简单，组网便利等好处。Proxy 和 Broker 都属于服务端组件，内网通信的延迟不会显著增加客户端收发消息的延迟。![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/213145/1721631074159-71767188-3103-41a5-80dd-a2a28e5d0adb.png#clientId=uafe8ad5f-3e17-4&from=paste&height=384&id=u9b19a098&originHeight=768&originWidth=2056&originalType=binary&ratio=2&rotation=0&showTitle=false&size=692097&status=done&style=none&taskId=u87be9002-2f21-43ff-9bbe-f581fa98911&title=&width=1028)<br />注：Proxy 自身会向 NameServer 和 Broker 都建立长连接，Producer 和 Consumer 仅连接到 Proxy。
<a name="aPwMj"></a>
## 通信机制
Apache RocketMQ 客户端使用 TCP 访问服务端，根据传输的数据格式分为 Remoting 协议和 gRPC 协议。

- Remoting 协议诞生较早，是组件间通信默认的私有协议。其中 Remoting Java 客户端和主仓库同步演进和迭代，而多语言客户端（以下简称 SDK）归属于 Apache 社区多个独立仓库。
- gRPC 协议自 RocketMQ 5.0 版本推出，以 Protobuf 定义了底层传输的数据格式（详见 [RocketMQ API](https://github.com/apache/rocketmq-apis)），旨在以云原生主流技术演进轻量、标准、易扩展的客户端服务端通信协议。使用 gRPC 协议的 SDK 是以独立仓库 [RocketMQ Clients](https://github.com/apache/rocketmq-clients) 方式演进，支持 Java/C++/.NET/Go/Rust 等众多语言。
- RocketMQ 5.0 在服务端内部也提供了基于 Protobuf + gRPC 的管控 API 实现。

RocketMQ 的接入点是什么？为了简化客户端配置的复杂度，以直连模式部署的集群，客户端需要和服务端的 NameServer，Broker 进行点对点直连通信，客户端需要配置 NameServer 集群的负载均衡地址。对于以代理模式部署的集群，无论客户端使用 Remoting 还是 gRPC 协议，客户端仅需和 Proxy 进行通信，需要将配置接入点为 Proxy 的负载均衡地址。服务端会使用[协议协商技术](https://github.com/apache/rocketmq/wiki/RIP-55-Support-remoting-protocol-in-rocketmq-proxy-module)，自动区分 Remoting 和 gRPC 协议并处理客户端的请求。在受限网络环境中，客户端需要同时放通接入点的 8080 和 8081 端口。
<a name="fnSaF"></a>
## 存储机制
<a name="bzUeC"></a>
### 元数据管理
为了提升整体的吞吐量与提供跨副本组的高可用能力，RocketMQ 服务端一般会为单个 Topic 创建多个逻辑分区，即在多个副本组上各自维护部分分区 (Partition)，我们把它称为队列 (MessageQueue)。同一个副本组上同一个 Topic 的队列数相同并从 0 开始连续编号，不同副本组上的 MessageQueue 数量可以不同。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/213145/1721629257518-f3778041-c94a-4b79-91fe-c0cd2088b7de.png#clientId=u84497c1f-c419-4&from=paste&height=320&id=u18094212&originHeight=640&originWidth=1348&originalType=binary&ratio=2&rotation=0&showTitle=false&size=522102&status=done&style=none&taskId=udc5ec960-d65a-4064-9983-03777139db3&title=&width=674)<br />例如 topic-a 可以在 broker-1 主副本上有 4 个队列，编号 (queueId) 是 0-3，在 broker-1 备副本上完全相同，但是 broker-2 上可能就只有 2 个队列，编号 0-1。在 Broker 上元数据的组织管理方式是与上述模型匹配的，每一个 Topic 的 TopicConfig，包含了几个核心的属性，名称，读写队列数，权限与许多元数据标识，这个模型类似于 K8s 的 StatefulSet，队列从 0 开始编号，扩缩队列都在尾部操作（例如 24 个队列缩分区到 16，是留下了编号为 0-15 的分区）。Broker 还管理着当前节点上 Group 的相关信息和消费进度（位点），当消费进度更新时 并不会像 Topic Group 那样立刻持久化，而是使用一个定时任务做 CheckPoint。这个周期默认是 5 秒，所以当客户端有上下线，服务端主备切换或者正常发布时，可能会有秒级的消息重复，并观察到堆积量的短暂上升。
<a name="N2peE"></a>
### 高效的存储层实现
RocketMQ 存储的核心是极致优化的顺序写盘，数据以 append only 的形式不断的将新的消息追加到文件末尾。RocketMQ 使用了一种称为 MappedByteBuffer 的内存映射文件的办法，将一个文件映射到进程的地址空间，实现文件的磁盘地址和进程的一段虚拟地址关联，实际上是利用了NIO 中的 FileChannel 模型。在进行这种绑定后，用户进程就可以用指针（偏移量）的形式写入磁盘而不用进行 read / write 的系统调用，减少了数据在缓冲区之间来回拷贝的开销。当然这种内核实现的机制有一些限制，单个 mmap 的文件不能太大 (RocketMQ 选择了 1G)，此时再把多个 mmap 的文件用一个链表串起来构成一个逻辑队列 (称为 MappedFileQueue)，就可以在逻辑上实现一个无需考虑长度的存储空间来保存全部的消息。

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/webp/213145/1721184070944-36721724-720e-4fbc-8b86-26226bbcb3fb.webp#clientId=u68b4bc55-b6a6-4&from=paste&id=u94190ef1&originHeight=388&originWidth=1600&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uc976b6c8-eed8-4736-8fb4-0a519c6e1af&title=)
<a name="4ever-bi-139"></a>
### 单条消息的存储格式
RocketMQ 有一套相对复杂的消息存储编码用来将消息对象序列化，随后再将非定长的数据落到上述的真实的写入到文件中，存储格式中包括了索引队列的编号和位置。单条消息的存储格式如下：<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/213145/1721373261326-06836245-30da-4fd3-8f01-f468521f7f27.png#clientId=u033b11ca-bdcc-4&from=paste&height=291&id=ucf0b3609&originHeight=582&originWidth=2204&originalType=binary&ratio=2&rotation=0&showTitle=false&size=544924&status=done&style=none&taskId=u5233aef5-a42b-4be7-93a4-98a710c65fe&title=&width=1102)<br />可以发现，单条消息本身元数据占用的存储空间为固定的描述信息和变长的 body 和 properties 部分，而消息的 payload 通常大于 2K，也就是说元数据带来的额外存储开销只增加了 5%-10% 左右。很明显，单条消息越大，存储本身额外的开销（比例）就相对的越少。
<a name="4ever-bi-158"></a>
### 构建消息的索引
在数据写入 CommitLog 后，有一个后端的 ReputMessageService 服务 (也被称为 dispatch 线程) 会异步的构建多种索引（例如 ConsumeQueue 和 Index），满足不同形式的读取和查询诉求。在 RocketMQ 的模型下，消息本身存在的逻辑队列称为 MessageQueue，而对应的物理索引文件称为 ConsumeQueue。其中 dispatch 线程会源源不断的将消息从 CommitLog 取出，再拿出消息在 CommitLog 中的物理偏移量，消息长度以及 Tag Hash 等信息作为单条消息的索引，分发到对应的消费队列，构成了对 CommitLog 的引用 (Reference)。ConsumeQueue 中单条消息占用的索引空间只有 20B。当客户端尝试从服务端拉取消息时，会先读取索引并进行过滤，随后根据索引从 CommitLog 中获得真实的消息并返回。<br />![](https://intranetproxy.alipay.com/skylark/lark/0/2024/webp/213145/1721184071581-7c02ef2d-0aaa-41f0-96a1-c5e977b50642.webp#clientId=u68b4bc55-b6a6-4&from=paste&height=896&id=u8c114b07&originHeight=896&originWidth=1142&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u78d8a251-5f36-4d12-b249-c03f1ea2c88&title=&width=1142)
<a name="yplKH"></a>
## 高可用机制
<a name="SOIMw"></a>
### 架构演进
最早的时候，RocketMQ 基于 Master-Slave 模式提供了主备部署的架构，这种模式提供了一定的高可用能力，在 Master 节点负载较高情况下，读流量可以被重定向到备机。备机在正常工作场景下资源使用率较低，造成一定的资源浪费。为了解决这个问题，社区提出了在一个 Broker 进程内运行多个 BrokerContainer，通过在单节点主备交叉部署来同时承担多份流量，该方案无外部依赖，自愈能力强。这种方式下隔离性弱于使用原生容器方式进行隔离，同时由于架构的复杂度增加导致了自愈流程较为复杂。<br />另一条演进路线则是基于可切换的，RocketMQ 也尝试过依托于 Zookeeper 的分布式锁和通知机制进行 HA 状态的管理。引入外部依赖的同时给架构带来了复杂性，不容易做小型化部署，部署运维和诊断的成本较高。DLedger 方案是基于 Raft 的日志实现，集群内同一个副本组下的 Broker 会自动选主，Raft 中的副本身份被透出和复用到 Broker Role 层面，无外部依赖，然而强一致的 Raft 设计并未支持灵活的降级策略，无法在 C 和 A 之间灵活调整。<br />而 RocketMQ DLedger 融合模式是 RocketMQ 5.0 演进中结合上述两条路线后的一个系统的解决方案，推荐使用两副本进行部署，很好的权衡了整体拥有成本和运维复杂度，其核心流程如下：

1. 利用可内嵌于 NameServer 的 Controller 进行选主，无外部依赖，对两副本支持友好。
2. 引入 Epoch-StartOffset 机制来计算日志分叉位点。
3. 消息在进行写入时，提供了灵活的配置来协调系统对于可用性还是一致性优先的诉求。
4. 简化日志复制协议使得日志复制为高效。

利用任期 Epoch 和偏移量 StartOffset 实现一个新的截断算法。这种 Epoch-StartOffset 满足如下原则：

1. 通过共识协议保证给定的一个任期 Epoch 只有一个Leader。
2. 只有 Leader 可以写入新的数据流，满足一定条件才会被提交。
3. Follower 只能从 Leader 获取最新的数据流，Follower 上线时按照选举算法进行截断。

下面是一个选举截断的具体案例，选举截断算法思想和流程如下:

- 主 CommitLog Min = 300，Max = 2500，EpochMap = {<6, 200>, <7, 1200>, <8,2500>}
- 备 CommitLog Min = 300，Max = 2500，EpochMap = {<6, 200>, <7, 1200>, <8,2250>}

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/213145/1721375364707-2a95206b-ee55-4520-8eab-9313c788ce7e.png#clientId=u8e0dbd4e-8e0f-4&from=paste&height=525&id=u32e55a00&originHeight=1050&originWidth=2200&originalType=binary&ratio=2&rotation=0&showTitle=false&size=508565&status=done&style=none&taskId=u0895623e-c9cf-45bc-8187-535663699c9&title=&width=1100)

1. 备节点连接到主节点进行 HA 协商，获取主节点的 Epoch-StartOffset 信息并比较
2. 备从后向前找到任期-起始点相同的那个点作为分叉任期，在上述案例里是 <8, 2250>
3. 选择这个任期里主备结束位点的最小值（如果主副本没有切换且为最大任期，则主副本的结束位点是无穷大）
<a name="RopBr"></a>
### 实现对比
|  | 模式 | 优点 | 缺点 |
| --- | --- | --- | --- |
| 无切换架构 | Master-Slave 模式 | 实现简单，适用于中小型用户，人为干预管控力强。 | 冷备浪费资源，故障需要人工介入，由于故障的副本组不可写入消息，还会导致一些二级消息消费暂停，整体运维成本高。 |
| 无切换架构 | Broker Container 模式 | 无需选主，无外部依赖，自愈能力强，故障转移时间从 ~30 秒级降低为 < 3 秒。 | 由于数据是从主到备单向复制的，Container 模式会增加一定的运维复杂度，二级消息场景自愈流程较为复杂，还需要长期生产检验。 |
| 切换架构 | 利用 Raft 实现 | 自动主备切换。 | 故障转移时间较长，强一致的  Raft 并未支持灵活的降级，无法在 C 和 A 之间做灵活的调整；三副本成本压力较高。复制链路没有有效展现原生存储的优势。 |
| 融合架构 | 基于 Dledger Controller 实现 | 一套代码同时支持无切换和切换架构，且两种模式可以转换。复制协议相比于其他几种要显著简化，灵活降级。 | 提高了 NameServer 部署复杂度，仍需大规模生产环境验证。 |

<a name="UOuhA"></a>
## 客户端
RocketMQ 提供了灵活的负载均衡机制，主要体现在消费者如何均衡地从消息队列中获取消息。<br />主要分为三种消费模式：Push（推送模式)，Pull（拉取模式），Pop（无状态消费模式）。
<a name="aseIJ"></a>
### Push 和 Pull 消费
RocketMQ 中的 Push 并不是指传统意义上的客户端完全被动接收，底层是基于长轮询机制实现。

1. 长轮询：客户端与 Broker 建立长连接，并发送拉取消息的请求。如果当前没有新消息，Broker 不会立即响应，而是等待一段时间或直到有新消息到达再返回。
2. 消费位点：每个消费者维护自己的消费进度（消费位点），Broker 根据这些位点信息，只推送消费者尚未消费的消息。
3. 重平衡：当消费者组内的消费者实例发生变化时（如增加或减少消费者实例），RocketMQ会触发一次重平衡（Rebalance）操作，重新分配消息队列到各个消费者实例，以实现负载均衡。这个过程确保了消息的均匀消费，避免了消息积压或某些消费者空闲的情况。

Pull 模式更加主动，消费者根据自己的消费能力和需求，主动从 Broker 拉取消息。

1. 主动拉取：消费者主动向Broker发送拉取请求，指定要拉取的消息数量和偏移量（或时间戳），Broker 响应包含消息或空结果。
2. 位点管理和重平衡：与Push模式类似，每个消费者维护自己的消费进度，并在消费者实例变化时进行重平衡。但是，在Pull模式下，重平衡的逻辑更依赖于消费者的主动参与，消费者需要根据新的队列分配情况调整自己的拉取策略。

Push / Pull 消费模式的负载均衡是在客户端完成的，性能较高，但也有一些缺陷。

1. 客户端代码逻辑复杂，客户端要实现完整的负载均衡，拉消息，位点管理，消费失败后将消息发回 Broker 重试等逻辑。这给多语言客户端的支持造成很大的阻碍。
2. 消费者无法无限扩展，当消费者数量扩大到大于队列数量时，有的消费者将无法分配到队列。
3. 当某些消费者僵死（hang 住）时，会造成其消费的队列的消息堆积。
<a name="wbzkW"></a>
### Pop 消费
在 RocketMQ 5.0 中，Pop 消费模式借助 gRPC 封装的接口，促进了轻量化多语言客户端的实现，无需在各客户端重复实现重平衡逻辑，显著提升了系统的灵活性和扩展性。该设计核心在于将重平衡、位点管理及消息重试等任务转移至服务端处理，有效避免单点故障引起的消息积压，优化了整体消息处理效率和系统的水平扩展能力。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/213145/1721378379272-87a4eb8c-393b-4c84-ab15-b91a7a96cc85.png#clientId=u430ffe7f-4b30-4&from=paste&height=532&id=ToUbG&originHeight=1064&originWidth=2838&originalType=binary&ratio=2&rotation=0&showTitle=false&size=648812&status=done&style=none&taskId=uf8b08cce-e6eb-4812-95ac-51cc545196c&title=&width=1419)<br />Push / Pull 模式下队列中有慢任务会阻塞整个队列。例如有位点为 34567 的 5 条消息，消费 offset = 5 时业务逻辑耗时非常久，并发消费模式下 67 两条消息消费较快，而观察到的堆积一直为 3 造成误判。消费者或者服务端宕机，业务对产生几秒的消费重复依然敏感，影响用户体验，例如短信推送场景。甚至，我们还有更有代表性的场景来命中这些 “缺陷”，例如渲染业务，队列中每一条消息代表一个渲染任务。

1. 消费者数量较多，同一个订阅组可能有成百上千台机器同时消费。
2. 该场景下单条数据的处理耗时较长，需要几秒至几个小时不等。
3. 由于消费者负载高和大量使用竞价实例，导致消费方进程假死和宕机率远高于一般业务。

传统的消息队列会遇到很经典的 “Work-Stealing” 难题，任务的负载无法均衡的分配到所有消费方，单条消息的阻塞会影响后续消费成功消息位点的提交。此时我们想要的是一个基于不可见时间的投递算法，该算法大致的工作流程如下：

1. 客户端设置一个不可见时间，例如 5 分钟，并向服务端拉取一批消息。
2. 服务端返回一批消息，并在后台开始倒计时 5 分钟，消息上会附加一个字段用来标识，也称为 handle。
3. 如果客户端 5 分钟内没有提交消费成功（ack by handle），5 分钟后客户端再次可以获取到这批消息。

![](https://intranetproxy.alipay.com/skylark/lark/0/2024/webp/213145/1721617811766-2d41d32f-8b87-4a5d-a067-bb8a503f2336.webp#clientId=u63929812-ba1e-4&from=paste&id=u4a50b48b&originHeight=982&originWidth=1600&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=u83f3e008-8b3f-49eb-9ccd-e5730aebc8e&title=)<br />很快我们就会发现这个模型还是有缺陷的，假如消费者拉取消息 1 分钟后立刻宕机了，业务不得不忍受 4 分钟的延迟才能再次处理，哪怕此时其他消费者还是空闲状态。这个时候就可以选择将消息的不可见时间设置为 1 分钟，在客户端处理业务的同时不停的 refresh 不可见时间，例如每隔 30 秒就调用 change invisible time，使剩余的不可见时间更新为 1 分钟，此时无论客户端何时宕机，消息的延迟时间会控制在 1 分钟之内。在 RocketMQ 中，这种基于区间和单条消息进行消费的方式被称为 “pop 消费”，对应的客户端实现是 SimpleConsumer，它的简单性在于客户端不再需要关心复杂的负载均衡和位点管理，也更容易适配多语言。
<a name="QX5n3"></a>
## 高级特性
<a name="TZstV"></a>
### 顺序消息
顺序消息是 Apache RocketMQ 提供的一种高级消息类型，支持消费者按照发送消息的先后顺序获取消息，从而实现业务场景中的顺序处理。 相比其他类型消息，顺序消息在发送、存储和投递的处理过程中，更多强调多条消息间的先后顺序关系。Apache RocketMQ 顺序消息的顺序关系通过消息组（MessageGroup）判定和识别。（注：这个概念在旧版本中被称为 ShardingKey）发送顺序消息时需要为每条消息设置归属的消息组，相同消息组的多条消息之间遵循先进先出的顺序关系，不同消息组、无消息组的消息之间不涉及顺序性。Apache RocketMQ 的消息的顺序性分为两部分，生产顺序性和消费顺序性：<br />如需保证消息生产的顺序性，则必须满足以下条件：

- 单一生产者：消息生产的顺序性仅支持单一生产者，不同生产者分布在不同的系统，即使设置相同的消息组，不同生产者之间产生的消息也无法判定其先后顺序。
- 串行发送：Apache RocketMQ 生产者客户端支持多线程安全访问，但如果生产者使用多线程并行发送，则不同线程间产生的消息将无法判定其先后顺序。
- Apache RocketMQ 通过消费者和服务端的协议保障消息消费严格按照存储的先后顺序来处理。

如需保证消息消费的顺序性，则必须满足投递顺序和有限重试两个条件，详情请参考[顺序消息设计](https://rocketmq.apache.org/zh/docs/featureBehavior/03fifomessage)。
<a name="XYKKh"></a>
### 定时消息
在分布式场景中会有定时调度、任务超时处理等场景，使用 Apache RocketMQ 的定时消息可以简化定时调度任务的开发逻辑，实现高性能、可扩展、高可靠的定时触发能力。RocketMQ 服务端有两种实现延时的机制，一种是延时队列，另一种是可持久化单层时间轮。下面简述这两种方案的服务端实现：

1. 延时队列实现：通过划分延时 Level 的方式，将不同延迟级别的消息放入不同的延迟队列，排序操作转换为了 O(1) 的 ConsumeQueue 的 append 操作。broker 有一个定时任务不断从各个延迟队列 “消费消息”，如果到达预期时间，就取出消息并重新放入到 commitLog 中。这样设计的延时队列已经满足大部分场景下的需求，如 15 分钟的特定场景，消息会堆积于单个队列中。该方案性能非常高，缺陷是只支持固定间隔的定时消息，例如 1 分钟，15 分钟，60 分钟等。
2. 可持久化单层时间轮：每一秒为一个定时的 Slot，将所有消息通过持久化的方式放入 TimerLog 中，当定时时间有冲突时使用哈希拉链的方式解决冲突，后一条消息保存对前一条消息的引用。同时对时间轮提供内存锁定，对于所有追加只在 TimerLog 尾部操作。当取出消息的线程从时间轮中扫描 Slot 时，从 TimerLog 中以此拉取所有定时消息并出队。
<a name="SpMlU"></a>
### 事务消息
RocketMQ 提供了事务消息的功能，采用 2PC (两段式协议) + 补偿机制（事务回查）的分布式事务功能，通过这种方式能达到分布式事务的最终一致。这里需要先来理解两个概念：

- 半事务消息：暂不能投递的消息，发送方已经成功地将消息发送到了消息队列 RocketMQ 版服务端，但是服务端未收到生产者对该消息的二次确认，此时该消息被标记成“暂不能投递”状态，处于该种状态下的消息即半事务消息。
- 消息回查： 由于网络闪断、生产者应用重启等原因，导致某条事务消息的二次确认丢失，消息队列 RocketMQ 版服务端通过扫描发现某条消息长期处于“半事务消息”时，需要主动向消息生产者组（ProducerGroup）的任意一个客户端询问该消息的最终状态（Commit 或是 Rollback），即消息回查。

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/213145/1721630512582-bc0b9b98-92d3-458b-8d4d-7a9125c0668a.png#clientId=u84497c1f-c419-4&from=paste&height=317&id=u6ac86d71&originHeight=634&originWidth=2000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=79028&status=done&style=none&taskId=u4b15121d-4a4a-4f60-85a5-c1e08f8db77&title=&width=1000)<br />事务消息发送步骤如下：

1. 发送方将半事务消息发送至消息队列 RocketMQ 版服务端。
2. 消息队列 RocketMQ 版服务端将消息持久化成功之后，向发送方返回 Ack 确认消息已经发送成功，此时消息为半事务消息。
3. 发送方开始执行本地事务逻辑。
4. 发送方根据本地事务执行结果向服务端提交二次确认（Commit 或是 Rollback），服务端收到 Commit 状态则将半事务消息标记为可投递，订阅方最终将收到该消息；服务端收到 Rollback 状态则删除半事务消息，订阅方将不会接受该消息。 事务消息回查步骤如下：
5. 在断网或者是应用重启的特殊情况下，上述步骤 4 提交的二次确认最终未到达服务端，经过固定时间后服务端将对该消息发起消息回查。
6. 发送方收到消息回查后，需要检查对应消息的本地事务执行的最终结果。
7. 发送方根据检查得到的本地事务的最终状态再次提交二次确认，服务端仍按照步骤 4 对半事务消息进行操作。
<a name="EPjVo"></a>
### 消息查询
RocketMQ 作为业务消息的首选，除了上文中 ReputMessageService 线程除了构建消费队列的索引外，还同时为每条消息根据 id, key 构建了索引到 IndexFile。这是方便快速快速定位目标消息而产生的，当然这个构建随机索引的能力是可以降级的，IndexFile文件结构如下：<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2024/png/213145/1721378192280-4f512f14-f103-4405-97a7-1d2190bf5b0c.png#clientId=u430ffe7f-4b30-4&from=paste&height=445&id=uc9aaf85c&originHeight=890&originWidth=3368&originalType=binary&ratio=2&rotation=0&showTitle=false&size=471315&status=done&style=none&taskId=u96456528-50a0-4ea8-af5f-5687ab592b7&title=&width=1684)<br />IndexFile 也是定长的，从单个文件的数据结构来说，这是实现了一种简单原生的哈希拉链机制。当一条新的消息索引进来时，首先使用 hash 算法命中黄色部分 500w 个 slot 中的一个，如果存在冲突就使用拉链解决，将最新索引数据的 next 指向上一条索引位置。同时将消息的索引数据 append 至文件尾部（绿色部分），这样便形成了一条当前 slot 按照时间存入的倒序的链表。这里其实也是一种 LSM compaction 在消息模型下的改进，降低了写放大。当用户按照 UniqueKey（MsgId）或者业务 Key 来进行查询时，会先从索引查询消息报存在 CommitLog 中的位置并取回数据返回客户端。
