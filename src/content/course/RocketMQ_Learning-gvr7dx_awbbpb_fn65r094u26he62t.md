---
title: "DashBoard 部署指南"
description: "DashBoard 部署指南"
date: "2024-08-11"
category: "article"
tags: ["deploy"]
author: ""
---

 `RocketMQ Dashboard` 是 RocketMQ 的管控利器，为用户提供客户端和应用程序的各种事件、性能的统计信息，支持以可视化工具代替 Topic 配置、Broker 管理等命令行操作。
<a name="61a3ec66"></a>
## 介绍
<a name="3e3d01e7"></a>
### 功能概览
| 面板 | 功能 |
| --- | --- |
| 运维 | 修改 nameserver 地址; 选用 `VIPChannel` |
| 驾驶舱 | 查看 broker, topic 消息量 |
| 集群 | 集群分布，broker 配置、运行信息 |
| 主题 | 搜索、筛选、删除、更新/新增主题，消息路由，发送消息，重置消费位点 |
| 消费者 | 搜索、删除、新增/更新消费者组，终端，消费详情，配置 |
| 消息 | 消息记录，私信消息，消息轨迹等消息详情 |

操作面板：

![](https://img.alicdn.com/imgextra/i4/O1CN0111yFs91MHN2PXADcN_!!6000000001409-2-tps-1241-831.png)
<a name="c182e73c"></a>
## 快速开始
系统要求：

1. Linux/Unix/Mac
2. 64bit JDK 1.8+
3. Maven 3.2.x
4. 启动 [RocketMQ](https://rocketmq.apache.org/docs/quick-start/)

网络配置：

1. 云服务器可远程访问或本地虚拟机可 PING 通外网
2. `rocketmq` 配置文件 `broker.conf / broker-x.properties` 设置 nameserver 地址和端口号
3. 用配置文件启动 broker
<a name="3be975ce"></a>
### 1. docker 镜像安装
① 安装 docker，拉取 `rocketmq-dashboard` 镜像
```shell
$ docker pull apacherocketmq/rocketmq-dashboard:latest
```
② docker 容器中运行 `rocketmq-dashboard`
```shell
$ docker run -d --name rocketmq-dashboard -e "JAVA_OPTS=-Drocketmq.namesrv.addr=127.0.0.1:9876" -p 8080:8080 -t apacherocketmq/rocketmq-dashboard:latest
```
`namesrv.addr:port` 替换为 `rocketmq` 中配置的 nameserver 地址：端口号<br />开放端口号：8080，9876，10911，11011 端口

- 云服务器：设置安全组访问规则
- 本地虚拟机：关闭防火墙，或 `-add-port`
<a name="bd55804c"></a>
### 2. 源码安装
源码地址：[apache/rocketmq-dashboard](https://github.com/apache/rocketmq-dashboard)<br />下载并解压，切换至源码目录 `rocketmq-dashboard-master/`<br />① 编译 `rocketmq-dashboard`
```shell
$ mvn clean package -Dmaven.test.skip=true
```
② 运行 `rocketmq-dashboard`
```shell
$ java -jar target/rocketmq-dashboard-1.0.1-SNAPSHOT.jar
```
提示：**Started App in x.xxx seconds (JVM running for x.xxx)** 启动成功

浏览器页面访问：namesrv.addr:8080

关闭 `rocketmq-dashboard` : ctrl + c

再次启动：执行 ②

**tips**：下载后的源码需要上传到 Linux 系统上编译，本地编译可能会报错。
<a name="658b4cfd"></a>
## 使用教程
<a name="4bdda76f"></a>
### 1. 创建主题 Topic
主题 `>` 新增/更新

![](https://img.alicdn.com/imgextra/i3/O1CN01Edjg1h1s7M54O8Ouu_!!6000000005719-2-tps-897-729.png)
<a name="945901c5"></a>
### 2. 创建消费者组 consumer
消费者 `>` 新增/更新

![](https://img.alicdn.com/imgextra/i2/O1CN019Ah6Jo2A72tl58W8Z_!!6000000008155-2-tps-897-413.png)
<a name="51b921b2"></a>
### 3. 重置消费位点
主题 `>` 重置消费位点

![](https://img.alicdn.com/imgextra/i2/O1CN01xkDItk1YT4XxVXzhG_!!6000000003059-2-tps-914-482.png)

**tips**:

- 集群消费支持重置消费位点, 广播模式不支持.
- 消费者不在线不能重置消费位点
<a name="c744f0f2"></a>
### 4. 扩容 Topic 队列
主题 `>` TOPIC 配置

![](https://img.alicdn.com/imgextra/i3/O1CN01PW61dJ1qK3pwVuWgd_!!6000000005476-2-tps-898-492.png)
<a name="4eb964c6"></a>
### 5. 扩容 Broker

- 安装部署一个新的 broker, nameserver 地址和当前集群一样

![](https://img.alicdn.com/imgextra/i4/O1CN01QPiQiC1DmDALv0EQG_!!6000000000258-2-tps-1043-372.png)

- 更新 Topic 的 BROKER_NAME

主题 `>` 新增/更新 `>` BROKER_NAME

![](https://img.alicdn.com/imgextra/i3/O1CN01awdigk2AEpd9gvY2S_!!6000000008172-2-tps-1028-415.png)
<a name="808498fe"></a>
### 6. 发送消息

- 向指定 Topic 发送消息

主题 `>` 发送消息

![](https://img.alicdn.com/imgextra/i1/O1CN01tWN8W91ZM24dC9MPa_!!6000000003179-2-tps-899-550.png)

- 发送结果

![](https://img.alicdn.com/imgextra/i2/O1CN01O6nuuL1Gi2P4q4TLh_!!6000000000655-2-tps-959-643.png)