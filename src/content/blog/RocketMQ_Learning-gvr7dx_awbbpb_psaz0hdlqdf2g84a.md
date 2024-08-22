---
title: "深度剖析 RocketMQ 5.0 之事件驱动：云时代的事件驱动有啥不同？"
description: "深度剖析 RocketMQ 5.0 之事件驱动：云时代的事件驱动有啥不同？"
date: "2024-08-09"
tags: ["explore"]
author: "隆基"
img: "https://img.alicdn.com/imgextra/i2/O1CN01wf42Ic2267AMSFv2j_!!6000000007070-2-tps-596-360.png"
---

作者｜隆基

> 简介： 本文技术理念的层面了解一下事件驱动的概念。RocketMQ 5.0 在面向云时代的事件驱动架构新推出的子产品 EventBridge，最后再结合几个具体的案例帮助大家了解云时代的事件驱动方案。


<a name="slide-0"></a>
## 1.前言

从初代开源消息队列崛起，到 PC 互联网、移动互联网爆发式发展，再到如今 IoT、云计算、云原生引领了新的技术趋势，消息中间件的发展已经走过了 30 多个年头。

目前，消息中间件在国内许多行业的关键应用中扮演着至关重要的角色。随着数字化转型的深入，客户在使用消息技术的过程中往往同时涉及交叉场景，比如同时进行物联网消息、微服务消息的处理，同时进行应用集成、数据集成、实时分析等，企业需要为此维护多套消息系统，付出更多的资源成本和学习成本。

在这样的背景下，2022 年，RocketMQ 5.0 正式发布，相对于 RocketMQ 4.0，架构走向云原生化，并且覆盖了更多的业务场景。想要掌握最新版本 RocketMQ 的应用，就需要进行更加体系化的深入了解。

<a name="slide-1"></a>
## 2.背景

今天我们要学习的课程是 RocketMQ 5.0 的事件驱动。事件驱动是一个经典的概念，通过今天这节课，我们会掌握云时代的事件驱动和之前有哪些不同

这是今天我们要学习的内容，第一部分先从技术理念的层面了解一下事件驱动的概念。第二部分会讲，RocketMQ 5.0 在面向云时代的事件驱动架构新推出的子产品 EventBridge，最后再结合几个具体的案例帮助大家了解云时代的事件驱动方案。

<a name="slide-2"></a>
## 3. 事件驱动架构

<a name="slide-3"></a>
### 3.1. 事件驱动架构定义

首先我们来学习一下什么是事件驱动。先从事件驱动的定义来看，事件驱动本质上是一种软件设计模式。它能够最大化降低不同模块以及不同系统之间的耦合度。

这里有一个典型的事件驱动架构图，首先是事件生产者发送事件到 EventBroker，然后 EventBroker 会把事件路由到对应的消费者进行事件处理。事件处理能够灵活扩展，随时增减事件消费者，事件生产者对此透明。

为什么说事件驱动是个很经典的设计模式呢，因为早在几十年前，就出现过多种事件驱动的技术，比如桌面客户端编程框架，点击按钮就可以触发 onclick 事件，开发者编写业务逻辑响应事件。在编程语言上，也经常会采用事件驱动的代码模式，比如 callback、handler 这类的函数。进入分布式系统的时代，系统之间的通信协同也会采用事件驱动的方式。

你有没有发现，这里的图和之前 RocketMQ 的消息应用解耦图很像。没错，无论是消息的发布订阅，还是事件的生产消费都是为了进行代码解耦、系统解耦。消息队列更偏技术实现，大部分的 EventBroker 都是基于消息队列实现的，而事件驱动更偏向于架构理念。

![](https://img.alicdn.com/imgextra/i2/O1CN01Q8oGgL1EphnlGW7z7_!!6000000000401-0-tps-750-422.jpg)

<a name="slide-4"></a>
### 3.2. 事件的特征

从技术角度来看，消息队列是和 RPC 对应的，一个是同步通信，一个是异步通信。消息队列并不会规定消息的内容，只负责传输二进制内容。如果从技术实现来看，的确，EDA 需要的核心技术就是消息队列的技术。事件驱动跟消息驱动最大的区别就是，事件是一种特殊的消息，只有消息满足了某些特征，才能把它叫做事件。

我打个比方，来看左边这个图。消息就像是一个抽象类，有多种子类，最主要的就是 Command 和 Event 两种。以信号灯为例，向信号灯发送打开的消息，这就是一种 Command，信号灯接受这个 Command 并开灯。开灯后，信号灯对外发出信号灯变成绿色的消息，这个就是一种 Event。

对于 Event 来说，有四个主要的特征：
第一，它是一个不可变的，事件就是表示已经发生了的事情，已经成为事实。
第二，事件有时间概念，并且对同一个实体来说事件的发送是有序的。如信号灯按顺序发送了绿、黄、红等事件。
第三，事件是无预期的，这个就是EDA架构之所以能够实现最大化解耦的特点，事件的产生者对于谁是事件消费者，怎么消费这个事件是不关心的。
第四，由于事件驱动是彻底解耦的，并且对于下游怎么去消费事件没有预期，所以事件是具象化的，应该包括尽可能详尽的信息，让下游消费者各取所需。比如像交通交通信号灯事件，包含多个字段，包括它的来源是谁、它的类型是什么？它的主题是什么？是具体的哪一个信号灯，另外它会包含唯一的ID，便于跟踪？它会有事件发生时间，事件的内容。

![](https://img.alicdn.com/imgextra/i1/O1CN01MgAeAi1XyO0Vbj6pW_!!6000000002992-0-tps-750-422.jpg)

<a name="slide-5"></a>
### 3.3. 云时代的事件驱动

在全行业数字化转型的时代，事件驱动架构应用范围扩大，成为 Gartner 年度十大技术趋势。在新型的数字化商业解决方案里，会有 60% 采纳 EDA 架构。

事件驱动作为一个经典的架构模式，为什么会在云时代再度成为焦点呢？主要有两个原因：

首先是云原生技术带来的，其中之一是微服务。微服务是云原生应用架构的核心，引入微服务架构，数字化企业能够按照小型化的业务单元和团队划分，以“高内聚、低耦合”的方式高效协作。但是微服务架构也会带来新的问题，比如大量同步微服务会面临延迟增大、可用性降低等风险，采用事件驱动的微服务体系，可提高微服务的韧性，降低延迟，实现更彻底的解耦。

另外一个云原生代表技术 Serverless 架构范式本身也是事件驱动的。现在主要的 Serverless 产品形态，无论是阿里云的函数计算、还是 AWS 的 Lambda，它们的主要触发源都是各种形态的事件，比如云产品事件，OSS 文件上传，触发用户基于函数进行文件加工处理计算；用户业务事件，EventBroker 触发函数运行消费逻辑；云产品运维事件，用户通过响应事件，在云平台的基础上扩展自己的自动化运维体系。事件驱动架构的大规模使用，能够帮助数字化企业释放云计算 Serverless 的技术红利。

IoT 也是事件驱动架构的重要推动力，有大量的 IoT 应用构建都是基于事件驱动的，比如传感器上报设备事件，温度变化事件、地址位置变化事件等等，云端应用订阅这些事件触发对应的业务流程。

在全行业大规模数字化转型后，跨业务、跨组织的业务合作会从线下搬到线上，在数字经济时代，数字化商业生态规模会持续扩大，跨组织业务协同更需要彻底解耦。而 EDA 天然具备的异步、解耦的特性就可以解决这一系列的问题。比如阿里聚石塔业务就是事件驱动的模式，聚石塔实时发布交易事件，合作伙伴包括ISV、软件服务商、品牌商家订阅消费交易事件，建设个性化的 CRM、商家运营、后台管理系统等等，形成一个庞大的电子商务数字化生态。

<a name="slide-6"></a>
## 4. EventBridge

<a name="slide-7"></a>
### 4.1. 云时代的事件驱动能力抽象

接下来进入第二个部分的内容，一起学习一下 RocketMQ 5.0 的 EventBridge。在了解这个系统的技术实现之前，我们先来了解一下 EventBridge 对事件驱动的通用能力抽象，从这里也可以了解到 EventBridge 的领域模型。

![](https://img.alicdn.com/imgextra/i4/O1CN01TNxK951l8XnUUICmd_!!6000000004774-0-tps-750-422.jpg)

我们从左往右看这张图。最左边是事件源，因为这个事件是希望被跨平台消费的，所以我们希望采用业界标准来作为事件的格式。同时，事件是有可能被跨组织消费的，所以我们需要一个统一的事件中心，让这些不同的事件源都注册到这个事件中心。对消费者来说就好比是一个事件商店，能够选择自己感兴趣的事件订阅。在事件消费者开始编写消费逻辑的时候，他还需要对这个事件的格式有更清楚的了解，需要知道这个事件有哪些内容，有哪些字段，分别是什么含义，才能编写正确的消费业务逻辑。所以，EventBridge 还提供了 schema 中心，有这个 schema 中心后，消费者对于事件的格式也就一目了然，不用跟事件源的发起者进行沟通了，整个效率也得到了大幅度的提升。再往后面看，就到了事件消费的环节，因为事件的消费者种类很多，不同消费者关注不同的事件类型，EventBridge 需要提供丰富的过滤规则。即便多个消费者对同一个事件感兴趣，但是可能只需要事件的部分内容，EventBridge 还提供了事件转换的能力。这就是 RocketMQ 5.0 对事件驱动的能力抽象。

<a name="slide-8"></a>
### 4.2. 统一事件标准

在云计算的时代、大规模数字化转型时代，我们强调事件驱动架构往往跨越了不同的组织，不同的平台。所以事件驱动架构需要一个统一的事件标准。在 EventBridge 这个产品里，我们采纳了 CNCF 基金会中的 CloudEvents 标准，这个是业界事件的事实标准，这个标准就是为了简化事件声明，提升事件在跨服务、跨平台的互操作性。

CloudEvents 带来了很多价值：
第一，它提供了一种规范，使得跨组织、跨平台的事件集成，有了共同语言，加速更多的事件集成。然后也因为有的规范，所以它可以加速跨服务，跨平台的事件的集成。
第二，随着 Serverless 的普及，各大云厂商都提供函数计算的服务，有了 CloudEvents 规范，用户在函数计算的使用上就可以实现无厂商绑定。
第三，webhook 是一种通用的集成模式，有了 CloudEvents 规范作为统一格式，不同系统的 webhook 能实现更好的互操作性。
最后，基于这样统一的规范，其实是更有利于沉淀事件驱动的基础软件设施的，比如跨服务的事件 Tracing 链路追踪。

![](https://img.alicdn.com/imgextra/i2/O1CN010k0TIN1DDrSXJUHgJ_!!6000000000183-0-tps-750-422.jpg)

<a name="slide-9"></a>
### 4.3. RocketMQ - EventBridge

如下图是 RocketMQ 面向 EDA 场景全新推出的产品形态 EventBridge。

它的核心技术都是基于 RocketMQ，但是在产品界面上面向事件驱动的业务进行一层抽象，核心领域对象从消息变成 CloudEvents。基于统一事件标准来构建事件驱动的数字生态。它的事件源也很多样，可以是云产品事件，可以是 SaaS 平台事件，应用自定义事件、通用的 WebHook。当然，它的事件目标更是多样化的，通过事件规则引擎把事件路由到不同的消费者，典型的消费者，比如函数计算，也可以是存储系统，消息通知如钉钉短信，还有通用的的 webhook。通过事件驱动这种彻底解耦的架构，更适合建设混合云、多云的数字化系统。

![](https://img.alicdn.com/imgextra/i2/O1CN01KDlagP1l0l437rlsQ_!!6000000004757-0-tps-750-422.jpg)

为了提升事件驱动的研发效率，EventBridge 也支持 Schema 的特性，支持事件信息的解释、预览，甚至还可以自动化的生成代码，让开发者以低代码、0 代码的方式完成事件集成。

![](https://img.alicdn.com/imgextra/i2/O1CN018U16RX1c5x0oofhZS_!!6000000003550-0-tps-750-422.jpg)

EventBridge 的另一个比较重要的特性是事件规则引擎。因为不同的事件消费者，他们对于事件的兴趣是不一样的。所以我们提供了七种事件过滤模式，包括前缀匹配、后缀匹配、除外匹配、数值匹配等等，可以进行各种复杂的组合逻辑过滤，只推送消费者感兴趣的事件。

当然，就算都关心同一个事件，不同消费者对事件内部的信息关注点也会有所不同。为了提升事件消费效率，我们也提供了四种事件转化器，可以只推送给消费者它关心的事件字段。还可以对事件进行自定义的模板转化，满足更灵活的业务诉求。

![](https://img.alicdn.com/imgextra/i1/O1CN01EirNGk1Vv3tqYKwtD_!!6000000002714-0-tps-750-422.jpg)

作为 RocketMQ 的子项目，在 EventBridge 里也同样提供了完整的可观测的能力。能够根据事件的时间、类型查询事件列表。每个事件都会生成唯一 ID。用户可以根据唯一 ID 去精确的定位事件的内容、发生时间、对应的事件规则，下游的消费状况，精准排查问题。

![](https://img.alicdn.com/imgextra/i2/O1CN01Vd9ATY271bL9CjGzS_!!6000000007737-0-tps-750-422.jpg)

<a name="slide-10"></a>
## 5. 典型案例

接下来结合几个典型案例来看 EventBridge 的使用场景。

第一个案例适用于使用大量云产品的公司。C 客户是一家以智能消费终端为核心的科技公司，希望收集账号里全部的云上事件，方便后续做分析或故障处理。公共云的 EventBridge 汇聚了所有的云产品事件，通过 EventBridge，客户能收集全量的事件对齐进行自定义的业务处理。还能够配置事件规则，过滤异常事件推送给监控系统或者钉钉，及时关注处理。

![](https://img.alicdn.com/imgextra/i1/O1CN01P0YMyQ24OYL9cSlot_!!6000000007381-0-tps-750-422.jpg)

第二个案例是 SaaS 事件的集成。现在随着整个云计算生态的繁荣，有不少企业不仅使用了公共云的 IaaS、PaaS 产品，也会同时使用三方的 SaaS 产品，比如各种 ERP、CRM 等系统。基于 EventBridge 标准的 HTTP、webhook 的集成能力，能够无缝连接三方 SaaS 系统作为事件源，企业能够收集到他所关心的所有 SaaS 事件，方便后续管理，比如申请单，入职单，报销单，订单等等这些场景。

![](https://img.alicdn.com/imgextra/i4/O1CN01ToqqGz1ELw3cEmfsN_!!6000000000336-0-tps-750-422.jpg)

第三个案例是 SaaS 平台集成，以钉钉为例，钉钉是典型的 SaaS 平台，他有繁荣的生态，拥有 4000+ 家的生态伙伴，包括 ISV 生态伙伴、硬件生态伙伴、服务商、咨询生态和交付生态伙伴等等。通过 EventBridge 把公共云的 Paas 层生态和钉钉的 SaaS 层生态连接起来，而且依赖 EventBridge 完成整体事件生命周期的管理，以 WebHook 的形式推送给下游 ISV 接收端。比如钉钉的官方事件源包括视频会议、日程、通讯录、审批流、钉盘、宜搭等，企业和 SaaS 厂商可以充分利用这些官方应用的事件构建企业级的应用系统，也可以把钉钉的官方数据流和其他系统做深度集成。

![](https://img.alicdn.com/imgextra/i4/O1CN01crdicg1TRAp8hkUZC_!!6000000002378-0-tps-750-422.jpg)