---
title: "前置准备工作"
description: "前置准备工作"
date: "2024-08-15"
tags: ["deploy"]
author: ""
---

在开始部署 RocketMQ 之前，需要做好如下准备：

1. 运行机器，64 位操作系统，推荐 Linux/Unix/macOS
2. 64 位 JDK 1.8+
3. 下载好 RocketMQ 二进制包，或者自行编译源码包
4. 进入 RocketMQ 目录，执行后续所有部署指令

本文将对上述流程作简要介绍，以便你能够顺利完成后续部署工作。
<a name="imcIh"></a>
## 安装 JDK
<a name="NA046"></a>
### Ubuntu 安装 JDK 11

**1. 更新软件包索引：**
```bash
sudo apt update
```

**2. 安装 OpenJDK 11：**
```bash
sudo apt install openjdk-11-jdk
```

**3. 验证安装：**
```bash
java -version
```
<a name="gEF5p"></a>
## CentOS 安装 JDK 11

**1. 更新软件包索引：**
```bash
sudo yum update
```

**2. 安装 OpenJDK 11：**
```bash
sudo yum install java-11-openjdk-devel
```

**3. 验证安装：**
```bash
java -version
```

若安装后无法使用 java，可能需要你设置 JAVA_HOME 路径，可以自行检索解决方法。
<a name="J26Eu"></a>
## 准备 RocketMQ
RocketMQ 的安装包分为两种，二进制包和源码包。二进制包是已经编译完成后可以直接运行的，源码包是需要编译后运行的。为提升体验过程的流畅性，建议下载二进制包，直接运行 RocketMQ。本教程以 5.2.0 版本为例，介绍 RocketMQ 的准备过程。你也可以选择其它版本的 RocketMQ 进行下载，微调命令进行部署。

**您可以点击**[**这里**](https://dist.apache.org/repos/dist/release/rocketmq/5.2.0/rocketmq-all-5.2.0-bin-release.zip)**下载二进制包（强烈推荐）**。也可以点击[这里](https://dist.apache.org/repos/dist/release/rocketmq/5.2.0/rocketmq-all-5.2.0-source-release.zip) 下载 Apache RocketMQ 5.2.0 的源码包。

若您想从源码包开始进行上手体验，需要您[安装 maven](https://maven.apache.org/install.html)，进入下载源码的目录，执行如下编译命令：
```bash
$ unzip rocketmq-all-5.2.0-source-release.zip
$ cd rocketmq-all-5.2.0-source-release/
$ mvn -Prelease-all -DskipTests -Dspotbugs.skip=true clean install -U
```
若你选择直接下载 RocketMQ 的二进制包，则可以直接进入二进制包的目录中：
```bash
$ cd rocketmq-all-5.2.0-bin-release
```
若你选择从源码开始部署，且已经在本地自行编译完成了二进制文件，则可进入源码目录下的 distribution/target 中的二进制文件目录：
```bash
$ cd distribution/target/rocketmq-5.2.0/rocketmq-5.2.0
```
