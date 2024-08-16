---
title: "Quick Start 快速部署"
description: "Quick Start 快速部署"
date: "2024-08-10"
tags: ["deploy"]
author: ""
---

<a name="hQld2"></a>
## 前言
本部分内容主要介绍如何快速拉起 RocketMQ 集群，我们提供了如下一些快速拉起的指南：

1. 通过 Docker 部署 RocketMQ 集群
2. 通过 Docker Compose 迅速拉起 RocketMQ 集群
3. 迅速部署具备自动切换能力的 RocketMQ 集群
<a name="iUA2W"></a>
## 通过 Docker 部署 RocketMQ 集群
这一节介绍如何使用 Docker 快速部署一个单节点单副本 RocketMQ 服务，并完成简单的消息收发。

系统要求：

1. 64 位操作系统
2. 64 位 JDK 1.8+
<a name="UXs6p"></a>
### 1. 拉取 RocketMQ 镜像
这里以[dockerhub](https://hub.docker.com/r/apache/rocketmq/tags)上 RocketMQ 5.3.0 版本的镜像为例，介绍部署过程。
```shell
docker pull apache/rocketmq:5.3.0
```
<a name="zGBnD"></a>
### 2. 创建容器共享网络
RocketMQ 中有多个服务，需要创建多个容器，创建 docker 网络便于容器间相互通信。
```shell
docker network create rocketmq
```
<a name="thgGq"></a>
### 3. 启动 NameServer
```shell
# 启动 NameServer
docker run -d --name rmqnamesrv -p 9876:9876 --network rocketmq apache/rocketmq:5.3.0 sh mqnamesrv

# 验证 NameServer 是否启动成功
docker logs -f rmqnamesrv
```

如果我们看到 **'The Name Server boot success..'，** 那表示 NameServer 已成功启动。
<a name="gTscf"></a>
### 4. 启动 Broker + Proxy
NameServer 成功启动后，我们启动 Broker 和 Proxy。

- Linux
- Windows
```
# 配置 Broker 的IP地址
echo "brokerIP1=127.0.0.1" > broker.conf

# 启动 Broker 和 Proxy
docker run -d \
--name rmqbroker \
--network rocketmq \
-p 10912:10912 -p 10911:10911 -p 10909:10909 \
-p 8080:8080 -p 8081:8081 \
-e "NAMESRV_ADDR=rmqnamesrv:9876" \
-v ./broker.conf:/home/rocketmq/rocketmq-5.3.0/conf/broker.conf \
apache/rocketmq:5.3.0 sh mqbroker --enable-proxy \
-c /home/rocketmq/rocketmq-5.3.0/conf/broker.conf

# 验证 Broker 是否启动成功
docker exec -it rmqbroker bash -c "tail -n 10 /home/rocketmq/logs/rocketmqlogs/proxy.log"
```
如果我们可以看到 **'The broker boot success..'，** 表示 Broker 已成功启动。

至此，一个单节点副本的 RocketMQ 集群已经部署起来了，我们可以利用脚本进行简单的消息收发。
<a name="fad72304"></a>
### 5. SDK 测试消息收发
工具测试完成后，我们可以尝试使用 SDK 收发消息。这里以 Java SDK 为例介绍一下消息收发过程，可以从 [rocketmq-clients](https://github.com/apache/rocketmq-clients) 中参阅更多细节。

1. 在 IDEA 中创建一个 Java 工程。
2. 在 _pom.xml_ 文件中添加以下依赖引入 Java 依赖库，将 `rocketmq-client-java-version` 替换成 [最新的版本](https://search.maven.org/search?q=g:org.apache.rocketmq%20AND%20a:rocketmq-client-java).
```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client-java</artifactId>
    <version>${rocketmq-client-java-version}</version>
</dependency>
```

3. 进入 broker 容器，通过 mqadmin 创建 Topic。
```shell
$ docker exec -it rmqbroker bash
$ sh mqadmin updatetopic -t TestTopic -c DefaultCluster
```

4. 在已创建的 Java 工程中，创建发送普通消息程序并运行，示例代码如下：
```java
import org.apache.rocketmq.client.apis.ClientConfiguration;
import org.apache.rocketmq.client.apis.ClientConfigurationBuilder;
import org.apache.rocketmq.client.apis.ClientException;
import org.apache.rocketmq.client.apis.ClientServiceProvider;
import org.apache.rocketmq.client.apis.message.Message;
import org.apache.rocketmq.client.apis.producer.Producer;
import org.apache.rocketmq.client.apis.producer.SendReceipt;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ProducerExample {
    private static final Logger logger = LoggerFactory.getLogger(ProducerExample.class);

    public static void main(String[] args) throws ClientException {
        // 接入点地址，需要设置成Proxy的地址和端口列表，一般是xxx:8080;xxx:8081
        // 此处为示例，实际使用时请替换为真实的 Proxy 地址和端口
        String endpoint = "localhost:8081";
        // 消息发送的目标Topic名称，需要提前创建。
        String topic = "TestTopic";
        ClientServiceProvider provider = ClientServiceProvider.loadService();
        ClientConfigurationBuilder builder = ClientConfiguration.newBuilder().setEndpoints(endpoint);
        ClientConfiguration configuration = builder.build();
        // 初始化Producer时需要设置通信配置以及预绑定的Topic。
        Producer producer = provider.newProducerBuilder()
            .setTopics(topic)
            .setClientConfiguration(configuration)
            .build();
        // 普通消息发送。
        Message message = provider.newMessageBuilder()
            .setTopic(topic)
            // 设置消息索引键，可根据关键字精确查找某条消息。
            .setKeys("messageKey")
            // 设置消息Tag，用于消费端根据指定Tag过滤消息。
            .setTag("messageTag")
            // 消息体。
            .setBody("messageBody".getBytes())
            .build();
        try {
            // 发送消息，需要关注发送结果，并捕获失败等异常。
            SendReceipt sendReceipt = producer.send(message);
            logger.info("Send message successfully, messageId={}", sendReceipt.getMessageId());
        } catch (ClientException e) {
            logger.error("Failed to send message", e);
        }
        // producer.close();
    }
}
```

5. 在已创建的 Java 工程中，创建订阅普通消息程序并运行。Apache RocketMQ 支持[SimpleConsumer](https://rocketmq.apache.org/zh/docs/featureBehavior/06consumertype)和[PushConsumer](https://rocketmq.apache.org/zh/docs/featureBehavior/06consumertype)两种消费者类型，您可以选择以下任意一种方式订阅消息。
```java
import java.io.IOException;
import java.util.Collections;
import org.apache.rocketmq.client.apis.ClientConfiguration;
import org.apache.rocketmq.client.apis.ClientException;
import org.apache.rocketmq.client.apis.ClientServiceProvider;
import org.apache.rocketmq.client.apis.consumer.ConsumeResult;
import org.apache.rocketmq.client.apis.consumer.FilterExpression;
import org.apache.rocketmq.client.apis.consumer.FilterExpressionType;
import org.apache.rocketmq.client.apis.consumer.PushConsumer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class PushConsumerExample {
    private static final Logger logger = LoggerFactory.getLogger(PushConsumerExample.class);

    private PushConsumerExample() {
    }

    public static void main(String[] args) throws ClientException, IOException, InterruptedException {
        final ClientServiceProvider provider = ClientServiceProvider.loadService();
        // 接入点地址，需要设置成Proxy的地址和端口列表，一般是xxx:8080;xxx:8081
        // 此处为示例，实际使用时请替换为真实的 Proxy 地址和端口
        String endpoints = "localhost:8081";
        ClientConfiguration clientConfiguration = ClientConfiguration.newBuilder()
            .setEndpoints(endpoints)
            .build();
        // 订阅消息的过滤规则，表示订阅所有Tag的消息。
        String tag = "*";
        FilterExpression filterExpression = new FilterExpression(tag, FilterExpressionType.TAG);
        // 为消费者指定所属的消费者分组，Group需要提前创建。
        String consumerGroup = "YourConsumerGroup";
        // 指定需要订阅哪个目标Topic，Topic需要提前创建。
        String topic = "TestTopic";
        // 初始化PushConsumer，需要绑定消费者分组ConsumerGroup、通信参数以及订阅关系。
        PushConsumer pushConsumer = provider.newPushConsumerBuilder()
            .setClientConfiguration(clientConfiguration)
            // 设置消费者分组。
            .setConsumerGroup(consumerGroup)
            // 设置预绑定的订阅关系。
            .setSubscriptionExpressions(Collections.singletonMap(topic, filterExpression))
            // 设置消费监听器。
            .setMessageListener(messageView -> {
                // 处理消息并返回消费结果。
                logger.info("Consume message successfully, messageId={}", messageView.getMessageId());
                return ConsumeResult.SUCCESS;
            })
            .build();
        Thread.sleep(Long.MAX_VALUE);
        // 如果不需要再使用 PushConsumer，可关闭该实例。
        // pushConsumer.close();
    }
}
```
<a name="3b62a4fa"></a>
### 5. 停止容器
完成实验后，我们可以通过以下方式停止容器。
```shell
# 停止 NameServer 容器
docker stop rmqnamesrv

# 停止 Broker 容器
docker stop rmqbroker
```
<a name="Rz9Zq"></a>
## 通过 Docker Compose 迅速拉起 RocketMQ 集群
这一节介绍如何使用 Docker-compose 快速部署一个单节点单副本 RocketMQ 服务，并完成简单的消息收发。

系统要求：

   1. 64 位操作系统
   2. 64 位 JDK 1.8+

具体可参照 [前置准备工作指南](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_owt0w53vl6wazv91/) 进行配置。
<a name="fe642368"></a>
### 1. 编写 docker-compose
为了快速启动并运行 RockerMQ 集群，您可以使用以下模板通过修改或添加环境部分中的配置来创建 docker-compose.yml 文件。
```
version: '3.8'
services:
  namesrv:
    image: apache/rocketmq:5.3.0
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    networks:
      - rocketmq
    command: sh mqnamesrv
  broker:
    image: apache/rocketmq:5.3.0
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
      - 10912:10912
    environment:
      - NAMESRV_ADDR=rmqnamesrv:9876
    depends_on:
      - namesrv
    networks:
      - rocketmq
    command: sh mqbroker
  proxy:
    image: apache/rocketmq:5.3.0
    container_name: rmqproxy
    networks:
      - rocketmq
    depends_on:
      - broker
      - namesrv
    ports:
      - 8080:8080
      - 8081:8081
    restart: on-failure
    environment:
      - NAMESRV_ADDR=rmqnamesrv:9876
    command: sh mqproxy
networks:
  rocketmq:
    driver: bridge
```
<a name="f094af8c"></a>
### 2. 启动 RocketMQ 集群
根据 docker-compose.yml 文件启动所有定义的服务。

- Linux
- Windows
```
docker-compose up -d
```
<a name="936be376"></a>
### 3. SDK 测试消息收发
工具测试完成后，我们可以尝试使用 SDK 收发消息。这里以 Java SDK 为例介绍一下消息收发过程，可以从[ rocketmq-clients](https://github.com/apache/rocketmq-clients) 中参阅更多细节。

1. 在 IDEA 中创建一个 Java 工程。
2. 在 _pom.xml_ 文件中添加以下依赖引入 Java 依赖库，将 `rocketmq-client-java-version` 替换成 [最新的版本](https://search.maven.org/search?q=g:org.apache.rocketmq%20AND%20a:rocketmq-client-java).
```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client-java</artifactId>
    <version>${rocketmq-client-java-version}</version>
</dependency>
```

3. 进入 broker 容器，通过 mqadmin 创建 Topic。
```shell
$ docker exec -it rmqbroker bash
$ sh mqadmin updatetopic -t TestTopic -c DefaultCluster
```

4. 在已创建的 Java 工程中，创建发送普通消息程序并运行，示例代码如下：
```java
import org.apache.rocketmq.client.apis.ClientConfiguration;
import org.apache.rocketmq.client.apis.ClientConfigurationBuilder;
import org.apache.rocketmq.client.apis.ClientException;
import org.apache.rocketmq.client.apis.ClientServiceProvider;
import org.apache.rocketmq.client.apis.message.Message;
import org.apache.rocketmq.client.apis.producer.Producer;
import org.apache.rocketmq.client.apis.producer.SendReceipt;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ProducerExample {
    private static final Logger logger = LoggerFactory.getLogger(ProducerExample.class);

    public static void main(String[] args) throws ClientException {
        // 接入点地址，需要设置成Proxy的地址和端口列表，一般是xxx:8080;xxx:8081
        String endpoint = "localhost:8081";
        // 消息发送的目标Topic名称，需要提前创建。
        String topic = "TestTopic";
        ClientServiceProvider provider = ClientServiceProvider.loadService();
        ClientConfigurationBuilder builder = ClientConfiguration.newBuilder().setEndpoints(endpoint);
        ClientConfiguration configuration = builder.build();
        // 初始化Producer时需要设置通信配置以及预绑定的Topic。
        Producer producer = provider.newProducerBuilder()
            .setTopics(topic)
            .setClientConfiguration(configuration)
            .build();
        // 普通消息发送。
        Message message = provider.newMessageBuilder()
            .setTopic(topic)
            // 设置消息索引键，可根据关键字精确查找某条消息。
            .setKeys("messageKey")
            // 设置消息Tag，用于消费端根据指定Tag过滤消息。
            .setTag("messageTag")
            // 消息体。
            .setBody("messageBody".getBytes())
            .build();
        try {
            // 发送消息，需要关注发送结果，并捕获失败等异常。
            SendReceipt sendReceipt = producer.send(message);
            logger.info("Send message successfully, messageId={}", sendReceipt.getMessageId());
        } catch (ClientException e) {
            logger.error("Failed to send message", e);
        }
        // producer.close();
    }
}
```

5. 在已创建的 Java 工程中，创建订阅普通消息程序并运行。Apache RocketMQ 支持[SimpleConsumer](https://rocketmq.apache.org/zh/docs/featureBehavior/06consumertype)和[PushConsumer](https://rocketmq.apache.org/zh/docs/featureBehavior/06consumertype)两种消费者类型，您可以选择以下任意一种方式订阅消息。
```java
import java.io.IOException;
import java.util.Collections;
import org.apache.rocketmq.client.apis.ClientConfiguration;
import org.apache.rocketmq.client.apis.ClientException;
import org.apache.rocketmq.client.apis.ClientServiceProvider;
import org.apache.rocketmq.client.apis.consumer.ConsumeResult;
import org.apache.rocketmq.client.apis.consumer.FilterExpression;
import org.apache.rocketmq.client.apis.consumer.FilterExpressionType;
import org.apache.rocketmq.client.apis.consumer.PushConsumer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class PushConsumerExample {
    private static final Logger logger = LoggerFactory.getLogger(PushConsumerExample.class);

    private PushConsumerExample() {
    }

    public static void main(String[] args) throws ClientException, IOException, InterruptedException {
        final ClientServiceProvider provider = ClientServiceProvider.loadService();
        // 接入点地址，需要设置成Proxy的地址和端口列表，一般是xxx:8080;xxx:8081
        String endpoints = "localhost:8081";
        ClientConfiguration clientConfiguration = ClientConfiguration.newBuilder()
            .setEndpoints(endpoints)
            .build();
        // 订阅消息的过滤规则，表示订阅所有Tag的消息。
        String tag = "*";
        FilterExpression filterExpression = new FilterExpression(tag, FilterExpressionType.TAG);
        // 为消费者指定所属的消费者分组，Group需要提前创建。
        String consumerGroup = "YourConsumerGroup";
        // 指定需要订阅哪个目标Topic，Topic需要提前创建。
        String topic = "TestTopic";
        // 初始化PushConsumer，需要绑定消费者分组ConsumerGroup、通信参数以及订阅关系。
        PushConsumer pushConsumer = provider.newPushConsumerBuilder()
            .setClientConfiguration(clientConfiguration)
            // 设置消费者分组。
            .setConsumerGroup(consumerGroup)
            // 设置预绑定的订阅关系。
            .setSubscriptionExpressions(Collections.singletonMap(topic, filterExpression))
            // 设置消费监听器。
            .setMessageListener(messageView -> {
                // 处理消息并返回消费结果。
                logger.info("Consume message successfully, messageId={}", messageView.getMessageId());
                return ConsumeResult.SUCCESS;
            })
            .build();
        Thread.sleep(Long.MAX_VALUE);
        // 如果不需要再使用 PushConsumer，可关闭该实例。
        // pushConsumer.close();
    }
}
```
<a name="e3a5fa50"></a>
### 4. 停止所有服务
```shell
docker-compose down
```
<a name="gmwLz"></a>
## 迅速部署具备自动切换能力的 RocketMQ 集群
![](https://img.alicdn.com/imgextra/i1/O1CN018RI1F21tGLTHOgPdV_!!6000000005874-0-tps-1397-753.jpg)

该文档主要介绍如何快速构建自动主从切换的 RocketMQ 集群，其架构如上图所示，主要增加支持自动主从切换的 Controller 组件，其可以独立部署也可以内嵌在 NameServer 中。
<a name="d1tJR"></a>
### 1. 前置准备工作
你需要准备好运行 RocketMQ 的环境，以及可运行文件。具体可以参考[前置准备工作指南](/course/deploy/rocketmq_learning-gvr7dx_awbbpb_owt0w53vl6wazv91/)。
<a name="DYqT8"></a>
### 2. 快速部署
在你的 RocketMQ 目录下，运行如下脚本：
```shell
$ sh bin/controller/fast-try.sh start
```
如果上面的步骤执行成功，可以通过运维命令查看 Controller 状态。
```shell
$ sh bin/mqadmin getControllerMetaData -a localhost:9878
```
-a 代表集群中任意一个 Controller 的地址

至此，启动成功，现在可以向集群收发消息，并进行切换测试了。

如果需要关闭快速集群，可以执行：
```shell
$ sh bin/controller/fast-try.sh stop
```
对于快速部署，默认配置在 conf/controller/quick-start 里面，默认的存储路径在 /tmp/rmqstore，且会开启一个 Controller (嵌入在 Namesrv) 和两个 Broker。
<a name="QgPQ2"></a>
#### 2.1 查看 SyncStateSet
可以通过运维工具查看 SyncStateSet：
```shell
$ sh bin/mqadmin getSyncStateSet -a localhost:9878 -b broker-a
```
-a 代表的是任意一个 Controller 的地址

如果顺利的话，可以看到以下内容：

![](https://img.alicdn.com/imgextra/i3/O1CN015m2KfB21nnKB8rD47_!!6000000007030-0-tps-511-151.jpg)
<a name="R7WAj"></a>
#### 2.2 查看 BrokerEpoch
可以通过运维工具查看 BrokerEpochEntry：
```shell
$ sh bin/mqadmin getBrokerEpoch -n localhost:9876 -b broker-a
```
-n 代表的是任意一个 Namesrv 的地址

如果顺利的话，可以看到以下内容：

![](https://img.alicdn.com/imgextra/i3/O1CN0160FeBh227wkZcAXyk_!!6000000007074-0-tps-435-185.jpg)
<a name="xOho0"></a>
### 3. 切换
部署成功后，现在尝试进行 Master 切换。

首先，kill 掉原 Master 的进程，在上文的例子中，就是使用端口 30911 的进程：
```shell
#查找端口:
$ ps -ef|grep java|grep BrokerStartup|grep ./conf/controller/quick-start/broker-n0.conf|grep -v grep|awk '{print $2}'
#杀掉 master:
$ kill -9 PID
```
接着，用 SyncStateSet admin 脚本查看：
```shell
$ sh bin/mqadmin getSyncStateSet -a localhost:9878 -b broker-a
```
可以发现 Master 已经发生了切换。

![](https://img.alicdn.com/imgextra/i2/O1CN01ik1YRy1GMVn0g71Dp_!!6000000000608-0-tps-575-150.jpg)
<a name="MyUYP"></a>
### 4. Controller 内嵌 Namesvr 集群部署
Controller 以插件方式内嵌 Namesvr 集群(3 个 Node 组成)部署，快速启动：
```shell
$ sh bin/controller/fast-try-namesrv-plugin.sh start
```
或者通过命令单独启动：
```shell
$ nohup bin/mqnamesrv -c ./conf/controller/cluster-3n-namesrv-plugin/namesrv-n0.conf &
$ nohup bin/mqnamesrv -c ./conf/controller/cluster-3n-namesrv-plugin/namesrv-n1.conf &
$ nohup bin/mqnamesrv -c ./conf/controller/cluster-3n-namesrv-plugin/namesrv-n2.conf &
```
如果上面的步骤执行成功，可以通过运维命令查看 Controller 集群状态。
```shell
$ sh bin/mqadmin getControllerMetaData -a localhost:9878
```
-a代表的是任意一个 Controller 的地址

如果 controller 启动成功可以看到以下内容：
```
#ControllerGroup        group1
#ControllerLeaderId     n0
#ControllerLeaderAddress        127.0.0.1:9878
#Peer:  n0:127.0.0.1:9878
#Peer:  n1:127.0.0.1:9868
#Peer:  n2:127.0.0.1:9858
```
启动成功后 Broker Controller 模式部署就能使用 Controller 集群。

如果需要快速停止集群：
```shell
$ sh bin/controller/fast-try-namesrv-plugin.sh stop
```
使用 fast-try-namesrv-plugin.sh 脚本快速部署，默认配置在 conf/controller/cluster-3n-namesrv-plugin 里面并且会启动 3 个 Namesvr 和 3 个 Controller (内嵌 Namesrv)。
<a name="MKY3F"></a>
### 5. Controller 独立集群部署
Controller 独立集群(3 个 Node 组成)部署，快速启动：
```shell
$ sh bin/controller/fast-try-independent-deployment.sh start
```
或者通过命令单独启动：
```shell
$ nohup bin/mqcontroller -c ./conf/controller/cluster-3n-independent/controller-n0.conf &
$ nohup bin/mqcontroller -c ./conf/controller/cluster-3n-independent/controller-n1.conf &
$ nohup bin/mqcontroller -c ./conf/controller/cluster-3n-independent/controller-n2.conf &
```
如果上面的步骤执行成功，可以通过运维命令查看 Controller 集群状态。
```shell
$ sh bin/mqadmin getControllerMetaData -a localhost:9878
```
-a 代表的是任意一个 Controller 的地址

如果 Controller 启动成功可以看到以下内容：
```
#ControllerGroup        group1
#ControllerLeaderId     n1
#ControllerLeaderAddress        127.0.0.1:9868
#Peer:  n0:127.0.0.1:9878
#Peer:  n1:127.0.0.1:9868
#Peer:  n2:127.0.0.1:9858
```
启动成功后 Broker Controller 模式部署就能使用 Controller 集群。

如果需要快速停止集群：
```shell
$ sh bin/controller/fast-try-independent-deployment.sh stop
```
使用 fast-try-independent-deployment.sh 脚本快速部署，默认配置在 conf/controller/cluster-3n-independent 里面并且会启动 3 个 Controller (独立部署)组成一个集群。

<a name="qiZ7M"></a>
### 6. 注意说明

- 若需要保证 Controller 具备容错能力，Controller 部署需要三副本及以上（遵循 Raft 的多数派协议）
- Controller 部署配置文件中配置参数`controllerDLegerPeers` 中的 IP 地址配置成其他节点能够访问的 IP，在多机器部署的时候尤为重要。例子仅供参考需要根据实际情况进行修改调整。
