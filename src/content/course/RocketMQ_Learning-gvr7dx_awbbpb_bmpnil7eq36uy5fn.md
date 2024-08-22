---
title: "Broker 部署指南"
description: "Broker 部署指南"
date: "2024-08-13"
tags: ["deploy"]
author: ""
---

<a name="Q8QWN"></a>
## 前言
本部分主要介绍如何通过传统命令行的方式，部署各式各样的 Broker 集群。

主要有如下几种部署方式：

**1. 单节点模式部署：**

本文将介绍单节点 Broker 的部署，通过本文，你能够在一台机器上启动一个 Broker 进程，提供基本的消息收发服务。若你需要多个 Broker 节点一起服务，则可以通过在不同机器上重复本文档所述的部署工作，部署不同 BrokerName 的 Broker 节点实现。部署相关介绍，参见第 2 节相关内容。

**2. 主备无切换模式部署：**

本文将介绍如何部署一个主备无切换模式的 RocketMQ 集群，这样的集群能够让集群拥有一定容灾能力：例如，在主节点宕机时，备节点仍然能够承担消息消费的任务。部署相关介绍，参见第 3 节相关内容。

**3. 基于BrokerContainer的混布：**

BrokerContainer 是一种 RocketMQ 特有的混布手段，它能够在单个进程中同时启动多个 Broker。部署相关介绍，参见第 4 节相关内容。

**4. 主备自动切换模式部署：**

主备自动切换模式和此前提到的无切换模式的区别在于，用该模式部署的 Broker 具备选主能力。被选为主节点的 Broker 拥有完全的 Broker Master 功能，而非仅仅作为备读节点工作。部署相关介绍，参见第 5 节相关内容。
<a name="X6BxE"></a>
## 单节点模式部署指南
本文将介绍单节点 Broker 的部署，通过本文，你能够在一台机器上启动一个 Broker 进程，提供基本的消息收发服务。若你需要多个 Broker 节点一起服务，则可以通过在不同机器上重复本文档所述的部署工作，部署不同 BrokerName 的 Broker 节点实现。
<a name="Cci26"></a>
### 准备工作
进行本文部署工作前，你需要先完成[前置准备工作](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_owt0w53vl6wazv91/)，以及[NamerServer 部署](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_tncndnkqzud0055o/)。你需要获取之前部署完成的 NameServer IP，为你部署的 Broker 提供路由服务。<br />本文中，NameServer 的 IP 使用 ${NameServer IP} 代表，部署过程中需要替换为你的 NameServer IP。
<a name="mt9d8"></a>
### 启动一个 Broker
在 RocketMQ 文件夹下，你可以运行下面的指令启动一台 Broker:
```bash
nohup sh bin/mqbroker -n ${NameServer IP}:9876 -c conf/broker.conf &
```
在该指令中，我们调用了 bin 目录下的 mqbroker 脚本，其中需要我们通过 -n 参数指定 NamerServer 的 IP 和端口号，并通过 -c 参数指定 broker 的配置文件。在本文档中，我们采用了 conf 文件夹下已有的 broker.conf 文件，无需你自行准备，你可以基于这个文件添加你需要的 broker 参数，并进行修改。

该配置文件中指定了 broker 名称，集群名等信息，相当于 Broker 的“户口本”：
```bash
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
```
<a name="EMbmE"></a>
### 进一步拓展
通过上一章节，我们启动了一台 Broker 进行服务，但是这样启动的 Broker 集群中只有一个节点——这种集群无疑是极为脆弱的。我们可以通过本章节，启动多个不同名称的 Broker 来组成一个集群，这个集群可以在某个节点宕机时仍然提供服务。

需要注意的是，**多个不同名称的 Broker 并不是主备关系**，它们彼此之间的数据都是完全独立的。

若要启动多个不同名称的 Broker，只需在**不同节点**（均需完成前置准备工作）上分别执行如下指令：
```bash
nohup sh bin/mqbroker -n ${NameServer IP}:9876 -c conf/2m-noslave/broker-a.properties &
```
```bash
nohup sh bin/mqbroker -n ${NameServer IP}:9876 -c conf/2m-noslave/broker-b.properties &
```
这里我们演示了启动两个不同的 Broker 节点，它们分别基于已有的 conf/2m-noslave/broker-a.properties 以及 conf/2m-noslave/broker-b.properties 文件启动。这两个文件的主要差别就是**配置项 brokerName 不同**。

因此，你可以基于本章内容，手动启动多个 Broker 节点，**只需要确保它们的 BrokerName 不相同即可**。
<a name="HF8PT"></a>
## 主备无切换模式部署指南
本文将介绍如何部署一个主备无切换模式的 RocketMQ 集群，这样的集群能够让集群拥有一定容灾能力：例如，在主节点宕机时，备节点仍然能够承担消息消费的任务。

需要注意的是，这个和单节点模式不同的地方在于，通过本文档部署的 Broker 共享同一个 BrokerName，它们仅仅是角色不同。而通过单节点模式部署多个 Broker，它们之间的 BrokerName 是不同的，彼此之间没有灾备能力。
<a name="CfESO"></a>
### 准备工作
在开始工作之前，你需要做好 [前置准备工作](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_owt0w53vl6wazv91/)，并[部署完成集群 NameServer](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_tncndnkqzud0055o/)。你需要获取之前部署完成的 NameServer IP，为你部署的 Broker 提供路由服务。

本文中，NameServer 的 IP 使用 ${NameServer IP} 代表，部署过程中需要替换为你的 NameServer IP。注意，在脚本中使用这个 NameServer IP 的大部分情况下，后面需要跟上端口号，一般是 “:9876”。

此外，主备 Broker 需要两个不同的配置文件，我们在这里直接选用 RocketMQ 已经准备好的 conf/2m-2s-sync 文件夹中的四个配置文件。

![](https://img.alicdn.com/imgextra/i1/O1CN01ko777C1D3mkQ6sQpm_!!6000000000161-0-tps-1380-522.jpg)

另外，你需要至少两个运行环境，以分别运行不同配置的 Broker，实现主备。
<a name="HxRPi"></a>
### 启动 Broker
在启动 Broker时，需要指定配置文件，若你有至少两个环境，则可以运行如下命令，分别启动 broker-a 的主备节点。
```bash
### 在机器A，启动第一个Master，例如NameServer的IP为：192.168.1.1:9876
$ nohup sh bin/mqbroker -n ${NameServer IP:9876} -c conf/2m-2s-sync/broker-a.properties &
 
### 在机器B，启动第一个Slave，例如NameServer的IP为：192.168.1.1:9876
$ nohup sh bin/mqbroker -n ${NameServer IP:9876} -c conf/2m-2s-sync/broker-a-s.properties &
```
若你有额外两台机器，则可以部署另一组 Broker 节点，即 Broker B：
```bash
### 在机器C，启动第二个Master，例如NameServer的IP为：192.168.1.1:9876
$ nohup sh bin/mqbroker -n ${NameServer IP:9876} -c conf/2m-2s-sync/broker-b.properties &
 
### 在机器D，启动第二个Slave，例如NameServer的IP为：192.168.1.1:9876
$ nohup sh bin/mqbroker -n ${NameServer IP:9876} -c conf/2m-2s-sync/broker-b-s.properties &
```
<a name="D6QR6"></a>
### 进一步拓展
为进一步说明部署细节，并强化你的自定义配置能力，我们来说一说这几个配置中最核心的配置项差别。

下面是 broker-a 的主节点配置文件：
```bash
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=MASTER
flushDiskType=SYNC_FLUSH
```
下面是  broker-a 的备节点的配置文件：
```bash
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=SYNC_FLUSH
```
可以看出，两者主要是两项差别：brokerId 以及 brokerRole。

针对主节点来说，brokerId 需要是 0，且 brokerRole 是 MASTER。而对于备节点来说，brokerId 是除 0 以外的任意自然数，且 brokerRole 是 SLAVE。只要遵循这样的配置，你就可以配置任意备节点的主备容灾集群了。

通过改变 brokerName，brokerId，brokerRole，你已经能够配置无切换模式下的横向扩展能力的 RocketMQ 集群了。这样的部署模式已经能够应对大部分的业务场景及容灾需求。

<a name="wUSSt"></a>
## 基于 BrokerContainer 的混布指南
BrokerContainer 是一种 RocketMQ 特有的混布手段，它能够在单个进程中同时启动多个 Broker。
<a name="JwUYM"></a>
### 准备工作
在使用 BrokerContainer 启动 Broker 时，请先完成[前置准备工作](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_owt0w53vl6wazv91/)，安装好 Java，并准备好 RocketMQ 的相关文件。此外，请先按照[NameServer 部署指南](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_tncndnkqzud0055o/)，启动完成 NameServer 节点。你需要获取之前部署完成的 NameServer IP，为你部署的 Broker 提供路由服务。

本文中，NameServer 的 IP 使用 ${NameServerIP} 代表，部署过程中需要替换为你的 NameServer IP。注意，在脚本中使用这个 NameServer IP 的大部分情况下，后面需要跟上端口号，一般是“:9876”。

<a name="jFBi1"></a>
### 启动 BrokerContainer
BrokerContainer 实际是在一个进程中启动多个 Container，且这个过程是可以动态进行的。你可以直接启动一个 BrokerContainer，然后使用指令添加或者删除其中的 Broker。

在这里，我们先准备一个 container 的配置文件，**注意，你需要将 ${NameServerIP} 替换为你的真实 nameserver ip + 端口号，如 127.0.0.1:9876**
```shell
echo "#配置端口,用于接收mqadmin命令" > conf/container-1.conf
echo "listenPort=10811" >> conf/container-1.conf
echo "#指定namesrv,请修改为你的namesrv地址,如127.0.0.1:9876" >> conf/container-1.conf
echo "namesrvAddr=${NameServerIP}" >> conf/container-1.conf
```
或者你可以直接将这些内容粘贴进入 conf/container-1.conf 中：
```shell
#配置端口,用于接收mqadmin命令
listenPort=10811
#指定namesrv,请修改为你的namesrv地址,如127.0.0.1:9876
namesrvAddr=${NameServerIP}
```
然后我们根据该配置文件，启动一个空的 Container：
```shell
nohup sh bin/mqbrokercontainer -c conf/container-1.conf > container.log &
```
启动完成后，我们可以通过下面的命令检查 container 是否启动正确：
```shell
$ cat container.log

# 若启动正常，则会有如下输出：
# The broker container boot success. serializeType=JSON and name server is xxxx:9876
```
然后我们使用 mqadmin 工具，在该 container 中添加 broker。这里我们需要使用 -b 指定新增 broker 的配置文件路径，并需要使用 -c 指定接收这个 mqadmin 指令的 container IP 和端口号。

我们在 RocketMQ 安装包中已经准备了两主两备的 broker 配置文件，位于 conf/container/2container-2m-2s 文件夹中：

![](https://img.alicdn.com/imgextra/i2/O1CN01ARaxW323nuemzTyrD_!!6000000007301-0-tps-660-442.jpg)

其中，broker-a-in-container1 和 broker-b-in-container1 是在第一个 container 上启动的，分别是 broker-a 的主节点以及 broker-b 的备节点。另外两个配置文件则是在第二个 container 上启动的，分别是 broker-a 的备节点和 broker-b 的主节点。

此处我们是在启动 container-1 的机器上原地使用 mqadmin 工具，故 container 便是 "127.0.0.1:10811"：
```shell
# 增加broker-a的主节点
sh bin/mqadmin addBroker -c 127.0.0.1:10811 -b conf/container/2container-2m-2s/broker-a-in-container1.conf

# 增加broker-b的备节点
sh bin/mqadmin addBroker -c 127.0.0.1:10811 -b conf/container/2container-2m-2s/broker-b-in-container1.conf
```
此时若添加成功，则会输出：
```shell
$ sh bin/mqadmin addBroker -c 127.0.0.1:10811 -b conf/container/2container-2m-2s/broker-a-in-container1.conf

# 输出如下：
# add broker to 127.0.0.1:10811 success

$ sh bin/mqadmin addBroker -c 127.0.0.1:10811 -b conf/container/2container-2m-2s/broker-b-in-container1.conf

# 输出如下：
# add broker to 127.0.0.1:10811 success
```
通过该方法，我们便能在一个 container 中添加多个 Broker 的主备节点。
<a name="iW1tq"></a>
### 启动第二个 BrokerContainer
接下去，你可以在另一个机器上启动另外一个 container，并指定相同的 NameServer 地址，启动剩下两个 Broker 节点：
```shell
# 准备container-2.conf
echo "#配置端口,用于接收mqadmin命令" > conf/container-2.conf
echo "listenPort=10811" >> conf/container-2.conf
echo "#指定namesrv,请修改为你的namesrv地址,如127.0.0.1:9876" >> conf/container-2.conf
echo "namesrvAddr=${NameServerIP}" >> conf/container-2.conf

# 启动container-2
nohup sh bin/mqbrokercontainer -c conf/container-2.conf > container.log &
# 若启动正常，则会有如下输出：
# The broker container boot success. serializeType=JSON and name server is xxxx:9876

# 增加broker-a的备节点
$ sh bin/mqadmin addBroker -c 127.0.0.1:10811 -b conf/container/2container-2m-2s/broker-a-in-container2.conf
# 输出如下：
# add broker to 127.0.0.1:10811 success

# 增加broker-b的主节点
$ sh bin/mqadmin addBroker -c 127.0.0.1:10811 -b conf/container/2container-2m-2s/broker-b-in-container2.conf
# 输出如下：
# add broker to 127.0.0.1:10811 success
```

至此，你已经灵活掌握了使用 BrokerContainer 的基本方法，可以在有限的机器上进行多样化的 Broker 混布。

<a name="ybR2x"></a>
## 主备自动切换模式部署指南
主备自动切换模式和此前提到的无切换模式的区别在于，用该模式部署的 broker 具备选主能力。被选为主节点的 Broker 拥有完全的 Broker Master 功能，而非仅仅作为备读节点工作。

![](https://img.alicdn.com/imgextra/i4/O1CN01WJRu9U1E39oxHdj3F_!!6000000000295-0-tps-1517-826.jpg)

该文档主要介绍如何部署支持自动主从切换的 RocketMQ 集群，其架构如上图所示，主要增加支持自动主从切换的 Controller 组件，其可以独立部署也可以内嵌在 NameServer 中。Controller 组件提供选主能力，若需要保证 Controller 具备容错能力，Controller 部署需要三副本及以上（遵循 Raft 的多数派协议）。
> **注意**
> Controller 若只部署单副本也能完成 Broker Failover，但若该单点 Controller 故障，会影响切换能力，但不会影响存量集群的正常收发。

Controller 部署有两种方式。一种是嵌入于 NameServer 进行部署，可以通过配置 enableControllerInNamesrv 打开（可以选择性打开，并不强制要求每一台 NameServer 都打开），在该模式下，NameServer 本身能力仍然是无状态的，也就是内嵌模式下若 NameServer 挂掉多数派，只影响切换能力，不影响原来路由获取等功能。另一种是独立部署，需要单独部署 Controller 组件。
<a name="FXYAA"></a>
### 前置工作
在进行本文的部署工作前，你需要完成[前置准备工作](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_owt0w53vl6wazv91/)。完成 JDK 的安装，以及 RocketMQ 安装包的获取。
<a name="aHsbE"></a>
### Controller 部署
<a name="akqcX"></a>
#### Controller 嵌入 NameServer 部署
![](https://img.alicdn.com/imgextra/i1/O1CN01Wqe89M1xwGBLhkq0r_!!6000000006507-0-tps-2590-632.jpg)

嵌入 NameServer 部署时只需要在 NameServer 的配置文件中设置 enableControllerInNamesrv=true，并填上 Controller 的配置即可。注意，如果需要使用该方式部署，则需要准备三台 NameServer 机器进行部署，并将这三台机器的 IP、端口填入 NameServer 的 controllerDLegerPeers 参数中。

在本章节中，我们参考 RocketMQ 安装包中自带的 controller/cluster-3n-namesrv-plugin/ 中的配置文件进行部署，如：
```properties
#Namesrv config
listenPort = 9876
enableControllerInNamesrv = true

#controller config
controllerDLegerGroup = group1
controllerDLegerPeers = n0-127.0.0.1:9878;n1-127.0.0.1:9868;n2-127.0.0.1:9858
controllerDLegerSelfId = n0
```
参数解释：

- enableControllerInNamesrv：Nameserver 中是否开启 controller，默认 false。
- controllerDLegerGroup：DLedger Raft Group 的名字，同一个 DLedger Raft Group 保持一致即可。
- controllerDLegerPeers：DLedger Group 内各节点的端口信息，同一个 Group 内的各个节点配置必须要保证一致。
- controllerDLegerSelfId：节点 id，必须属于 controllerDLegerPeers 中的一个；同 Group 内各个节点要唯一。
- controllerStorePath：controller 日志存储位置。controller 是有状态的，controller 重启或宕机需要依靠日志来恢复数据，该目录非常重要，不可以轻易删除。
- enableElectUncleanMaster：是否可以从 SyncStateSet 以外选举 Master，若为 true，可能会选取数据落后的副本作为 Master 而丢失消息，默认为 false。
- notifyBrokerRoleChanged：当 Broker 副本组上角色发生变化时是否主动通知，默认为 true。

参数设置完成后，指定配置文件启动 Nameserver 即可。由于 RocketMQ 安装包中的配置文件是为了在单机部署测试准备的，**因此若是需要测试、学习，可以直接在同一台机器下分别执行以下三个指令进行启动**：
```bash
nohup sh bin/mqnamesrv -c conf/controller/cluster-3n-namesrv-plugin/namesrv-n0.conf &
```
```bash
nohup sh bin/mqnamesrv -c conf/controller/cluster-3n-namesrv-plugin/namesrv-n1.conf &
```
```bash
nohup sh bin/mqnamesrv -c conf/controller/cluster-3n-namesrv-plugin/namesrv-n2.conf &
```

**若你需要在生产环境嵌入部署 controller，则需要修改这三个 namesrv 的配置文件**——在这三个配置文件的 controllerDLegerPeers 中填入相同的 NameServer-IP-0/1/2。然后分别指定 controllerDLegerSelfId，如：
```bash
#Namesrv config
listenPort = 9876
enableControllerInNamesrv = true

#controller config
controllerDLegerGroup = group1
controllerDLegerPeers = n0-{NameServer-IP-0}:9878;n1-{NameServer-IP-1}:9878;n2-{NameServer-IP-2}:9878
controllerDLegerSelfId = n0
```
将这三个文件分别保存为 conf/namesrv-n0.conf，conf/namesrv-n1.conf，vnamesrv-n2.conf，然后分别在三台 NameServer 机器中执行如下命令，进行启动：
```bash
# 在namesrv 0 执行：
nohup sh bin/mqnamesrv -c conf/namesrv-n0.conf&

# 在namesrv 1 执行：
nohup sh bin/mqnamesrv -c conf/namesrv-n1.conf&

# 在namesrv 2 执行：
nohup sh bin/mqnamesrv -c conf/namesrv-n2.conf&
```

<a name="QHe13"></a>
#### 单独部署 Controller
![](https://img.alicdn.com/imgextra/i3/O1CN0158JVJR1IdZmFZwKVr_!!6000000000916-0-tps-2970-892.jpg)

Controller 可以单独被部署在集群中。不过独立部署 Controller 后，仍然需要单独部署 NameServer 提供路由发现能力。也就是说，你需要遵循[NameServer 部署指南](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_tncndnkqzud0055o/)完成部署工作。

在启动 Controller 前，你需要准备好其配置文件。在这里，我们需要填入三台 Controller 的 IP 地址、端口号，并指定本配置文件的角色 id。我们默认在 9878 端口运行 controller，则 n0 节点 controller 的配置文件如下：
```bash
controllerDLegerGroup = group1
controllerDLegerPeers = n0-{Controller-IP-0}:9878;n1-{Controller-IP-1}:9878;n2-{Controller-IP-2}:9878
controllerDLegerSelfId = n0
```
注意，三个节点的 controllerDLegerPeers 参数应当是一致的，只有它们的 controllerDLegerSelfId 各不相同。请确保三个 controller 的 controllerDLegerSelfId 指代正确。

准备好 Controller 的配置文件，即可直接使用如下的指令进行启动：
```bash
$ nohup sh bin/mqcontroller -c controller.conf &
```
<a name="AYxAk"></a>
### Broker 部署
Broker 启动方法与之前的模式相同，不过在 Controller 模式下，Broker 配置必须设置两个参数才可启动：

- enableControllerMode：Broker controller 模式的总开关，只有该值为 true，自动主从切换模式才会打开。默认为 false。
- controllerAddr：controller 的地址，多个 controller 中间用分号隔开。例如你的Controller IP 分别为 {Controller-IP-0/1/2}:9878，则`controllerAddr = {Controller-IP-0}:9878;{Controller-IP-1}:9878;{Controller-IP-2}:9878`

作为示例，一个标准的配置文件如下：
```shell
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = -1
brokerRole = SLAVE
deleteWhen = 04
fileReservedTime = 48
enableControllerMode = true
controllerAddr = {Controller-IP-0}:9878;{Controller-IP-1}:9878;{Controller-IP-2}:9878
namesrvAddr = 127.0.0.1:9876
```

在完成配置后，如果该配置文件路径为当前目录下的 broker.conf 文件，则可以使用下面命令启动：
```shell
$ nohup sh bin/mqbroker -c broker.conf &
```
通过在不同机器执行该命令，你可以得到一个具备自动切换能力的 broker 集群，其 brokerName 均为 broker-a。

当然，你也可以通过该教程所说，部署具备自动切换能力的 broker-b/c/d，共同组成一个高可用 RocketMQ 集群。

> **注意**
> 自动主备切换模式下 Broker 无需指定 brokerId 和 brokerRole，其由 Controller 组件进行分配

在该模式下，我们为 Broker 增加了以下参数，提供自定义配置能力：

- syncBrokerMetadataPeriod：向 controller 同步 Broker 副本信息的时间间隔。默认 5000（5s）。
- checkSyncStateSetPeriod：检查 SyncStateSet 的时间间隔，检查 SyncStateSet 可能会 shrink SyncState。默认5000（5s）。
- syncControllerMetadataPeriod：同步 controller 元数据的时间间隔，主要是获取 active controller 的地址。默认10000（10s）。
- haMaxTimeSlaveNotCatchup：表示 Slave 没有跟上 Master 的最大时间间隔，若在 SyncStateSet 中的 slave 超过该时间间隔会将其从 SyncStateSet 移除。默认为 15000（15s）。
- storePathEpochFile：存储 epoch 文件的位置。epoch 文件非常重要，不可以随意删除。默认在 store 目录下。
- allAckInSyncStateSet：若该值为 true，则一条消息需要复制到 SyncStateSet 中的每一个副本才会向客户端返回成功，可以保证消息不丢失。默认为 false。
- syncFromLastFile：若 slave 是空盘启动，是否从最后一个文件进行复制。默认为 false。
- asyncLearner：若该值为 true，则该副本不会进入 SyncStateSet，也就是不会被选举成 Master，而是一直作为一个 learner 副本进行异步复制。默认为false。
- inSyncReplicas：需保持同步的副本组数量，默认为1，allAckInSyncStateSet=true 时该参数无效。
- minInSyncReplicas：最小需保持同步的副本组数量，若 SyncStateSet 中副本个数小于 minInSyncReplicas 则 putMessage 直接返回 PutMessageStatus.IN_SYNC_REPLICAS_NOT_ENOUGH，默认为1。

<a name="MA8PQ"></a>
### 兼容性
该模式未对任何客户端层面 API 进行新增或修改，不存在客户端的兼容性问题。

Nameserver 本身能力未做任何修改，Nameserver 不存在兼容性问题。如开启 enableControllerInNamesrv 且 controller 参数配置正确，则开启 controller 功能。

Broker 若设置 enableControllerMode=false，则仍然以之前方式运行。若设置 enableControllerMode=true，则需要部署 controller 且参数配置正确才能正常运行。<br />具体行为如下表所示：

|  | 旧版 Nameserver | 旧版 Nameserver + 独立部署 Controller | 新版 Nameserver 开启 controller 功能 | 新版 Nameserver 关闭 controller 功能 |
| --- | --- | --- | --- | --- |
| 旧版 Broker | 正常运行，无法切换 | 正常运行，无法切换 | 正常运行，无法切换 | 正常运行，无法切换 |
| 新版 Broker 开启 Controller 模式 | 无法正常上线 | 正常运行，可以切换 | 正常运行，可以切换 | 无法正常上线 |
| 新版 Broker 不开启 Controller 模式 | 正常运行，无法切换 | 正常运行，无法切换 | 正常运行，无法切换 | 正常运行，无法切换 |

<a name="dVTDx"></a>
### 升级注意事项
从上述兼容性表述可以看出，NameServer 正常升级即可，无兼容性问题。在不想升级 Nameserver 情况，可以独立部署 Controller 组件来获得切换能力。

针对 Broker 升级，分为两种情况：

（1）Master-Slave 部署升级成 Controller 切换架构<br />可以带数据进行原地升级，对于每组 Broker，停机主、备 Broker，**保证主、备的 CommitLog 对齐**（可以在升级前禁写该组 Broker 一段时间，或则通过拷贝方式保证一致），升级包后重新启动即可。
> **注意**
> 若主备 CommitLog 不对齐，需要保证主上线以后再上线备，否则可能会因为数据截断而丢失消息。