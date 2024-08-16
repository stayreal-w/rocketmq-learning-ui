---
title: "NameServer 部署指南"
description: "NameServer 部署指南"
date: "2024-08-14"
tags: ["deploy"]
author: ""
---

NameServer 是 RocketMQ 集群的核心组件，由于其无状态、轻量的特点，它的部署过程也非常简单。

若你已经完成 [**前置准备工作**](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_owt0w53vl6wazv91/)，则可以直接开始 NameServer 的部署。

安装完 RocketMQ 包后，我们在 RocketMQ 根目录下，执行下面的指令启动 NameServer：
```shell
### 启动namesrv
$ nohup sh bin/mqnamesrv &
 
### 验证namesrv是否启动成功
$ cat ~/logs/rocketmqlogs/namesrv.log
```
若一切正常，则会在执行完上述命令后，输出如下内容：
```shell
The Name Server boot success...
```
