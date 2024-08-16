---
title: "Proxy 部署指南"
description: "Proxy 部署指南"
date: "2024-08-12"
tags: ["deploy"]
author: ""
---

<a name="UyaF5"></a>
## 前言
本部分文档主要介绍 Proxy 的两种部署方式：

1. Local 模式：

本文主要介绍如何将 Proxy 与 Broker 一起部署在同一个进程中，这能够节约一些机器资源。部署相关介绍，参见第 2 节相关内容。

2. Cluster 模式：

本文主要介绍如何将 Proxy 作为一个独立集群进行部署。这能够提供较强的横向扩展能力，且能和无 Proxy 的已有 RocketMQ 集群较好的兼容。你可以直接在一个 RocketMQ 集群中部署 Proxy 组件，并使其生效。部署相关介绍，参见第 3 节相关内容。
<a name="ZM2za"></a>
## Local 模式部署 Proxy 指南
Proxy 是一个较为独立的组件，你可以将其与 Broker 部署在一起，也可以独立进行部署。本文主要介绍如何将 Proxy 与 Broker 一起部署在同一个进程中，也就是 Local 模式部署。

![](https://img.alicdn.com/imgextra/i1/O1CN01Prqboc1P7hWULMf9G_!!6000000001794-0-tps-1464-825.jpg)

<a name="U0LfX"></a>
### 前置工作
在开始工作之前，你需要做好 [前置准备工作](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_owt0w53vl6wazv91/)，并[部署完成集群 NameServer](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_tncndnkqzud0055o/)。你需要获取之前部署完成的 NameServer IP，为你部署的 Broker 提供路由服务。

本文中，NameServer 的 IP 使用 ${NameServer IP} 代表，部署过程中需要替换为你的 NameServer IP。注意，在脚本中使用这个 NameServer IP 的大部分情况下，后面需要跟上端口号，一般是 “:9876”。
<a name="qrWbv"></a>
### 部署 Broker + Proxy
由于 Local 模式下 Proxy 和 Broker 是同进程部署，因此在启动部署之前需要确定 Broker 的相关配置，并在启动命令中指定配置文件。

在这里，我们可以直接使用conf文件夹中已有的 Broker 配置文件，conf/broker.conf：
```shell
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
```
若该配置文件已经准备完毕，则只需要一行命令即可完成部署：
```shell
nohup sh bin/mqbroker -n localhost:9876 -c conf/broker.conf --enable-proxy &

# 若部署完成，则会有如下输出：
# Mon Aug 12 15:28:27 CST 2024 rocketmq-proxy startup successfully
```
所以只需要在正常启动 broker 的命令之后跟上 “--enable-proxy ”，即可在 broker 本地启动一个 proxy 代理。

至于 broker 集群的部署，可以参考 [Broker 部署指南](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_bmpnil7eq36uy5fn/) 系列文章：

   1. 单节点的 Broker 部署，可以参考 单节点模式部署指南 章节。
   2. 集群式的部署，可以参考 主备无切换模式部署指南 章节。
   3. 同进程混布模式的部署，可以参考 基于 BrokerContainer 的混布指南 章节。
   4. 主备自动切换模式的部署，可以参考 主备自动切换模式部署指南 章节。

<a name="ojRa7"></a>
## Cluster 模式部署 Proxy 指南
Proxy 是一个较为独立的组件，你可以将其与 Broker 部署在一起，也可以独立进行部署。本文主要介绍如何独立部署 Proxy 组件。

![](https://img.alicdn.com/imgextra/i4/O1CN01ovdF6G21l2wmz59vL_!!6000000007024-0-tps-1868-813.jpg)
<a name="jc0jh"></a>
### 前置工作
在开始工作之前，你需要做好 [前置准备工作](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_owt0w53vl6wazv91/)，并[部署完成集群 NameServer](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_tncndnkqzud0055o/)。你需要获取之前部署完成的 NameServer IP，为你部署的 Broker 提供路由服务。

本文中，NameServer 的 IP 使用 ${NameServer IP} 代表，部署过程中需要替换为你的 NameServer IP。注意，在脚本中使用这个 NameServer IP的大部分情况下，后面需要跟上端口号，一般是 “:9876”。

此外，由于集群模式的 Proxy 部署与 Broker 相对独立，可以在已经部署好的 Broker 集群上额外部署 Proxy 集群。至于 broker 集群的部署，可以参考 [Broker 部署指南](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_bmpnil7eq36uy5fn/) 系列文章：

   1. 单节点的 Broker 部署，可以参考 单节点模式部署指南 章节。
   2. 集群式的部署，可以参考 主备无切换模式部署指南 章节。
   3. 同进程混布模式的部署，可以参考 基于 BrokerContainer 的混布指南 章节。
   4. 主备自动切换模式的部署，可以参考 主备自动切换模式部署指南 章节。
<a name="uHGVN"></a>
### 部署 Proxy 集群
启动 proxy 集群时，仅需直接调用 bin/mqproxy 即可。可以参照下面的命令，在多台机器启动多个 Proxy
```shell
### 在机器A，启动第一个Proxy，例如NameServer的IP为：192.168.1.1
$ nohup sh bin/mqproxy -n ${NameServer IP}:9876 &

### 在机器B，启动第二个Proxy，例如NameServer的IP为：192.168.1.1
$ nohup sh bin/mqproxy -n ${NameServer IP}:9876 &

### 在机器C，启动第三个Proxy，例如NameServer的IP为：192.168.1.1
$ nohup sh bin/mqproxy -n ${NameServer IP}:9876 &
```
若需要指定配置文件，可以使用 `-pc`或者 `--proxyConfigPath` 进行指定
```shell
### 自定义配置文件
$ nohup sh bin/mqproxy -n ${NameServer IP}:9876 -pc /path/to/proxyConfig.json &
```

至此，你的 Broker 集群便拥有了多个 proxy。该部署方式十分灵活，推荐采用。
