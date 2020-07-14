---
date: 2018-10-24T14:20:00+08:00
title: MT Channel Based Broker
weight: 442
menu:
  main:
    parent: "eventing-broker"
description : "knative的MT Channel Based Broker"
---

https://knative.dev/docs/eventing/broker/mt-channel-based-broker/

Knative提供了一个多租户(Multi Tenant / MT)Broker的实现，使用channel进行事件路由。

### 开始准备

在使用 MT Broker 之前，您需要安装一个channel 提供商，例如，InMemoryChannel（用于开发目的），Kafka 或 Nats。

在您安装了将被MT Broker使用的channel提供商后，您必须创建一个ConfigMap，它指定如何配置Broker为路由事件创建的channel。

注意：本指南假设Knative Eventing安装在knative-eventing命名空间。如果您将Knative Eventing安装在不同的命名空间，请用该命名空间的名称替换knative-eventing。

### 配置Channel的ConfigMap

您可以通过修改每个channel类型的ConfigMap来定义每个channel类型的创建方式。

#### 示例 InMemoryChannel ConfigMap

当你安装eventing版本时，会自动创建以下YAML文件：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: knative-eventing
  name: config-br-default-channel
data:
  channelTemplateSpec: |
    apiVersion: messaging.knative.dev/v1
    kind: InMemoryChannel
```

要创建一个使用InMemoryChannel的Broker，你可以像这样创建一个Broker：

```shell
kubectl create -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  annotations:
    eventing.knative.dev/broker.class: MTChannelBasedBroker
  name: default
  namespace: default
spec:
  config:
    apiVersion: v1
    kind: ConfigMap
    name: config-br-default-channel
    namespace: knative-eventing
EOF
```

而Broker将使用InMemoryChannel来处理路由事件。

#### Kafka channel configmap 示例

要使用Kafka channel，你必须创建一个YAML文件来指定如何创建这些通道。注意：你必须安装了Kafka通道。

您可以将以下示例代码复制到您的Kafka通道ConfigMap中：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kafka-channel
  namespace: knative-eventing
data:
  channelTemplateSpec: |
    apiVersion: messaging.knative.dev/v1alpha1
    kind: KafkaChannel
    spec:
      numPartitions: 3
      replicationFactor: 1
```

注意：这个例子指定了两个Kafka Channels特有的额外参数：numPartitions和 replicationFactor。

要创建一个在底下使用Kafka的Broker，你会像这样做。

```shell
kubectl create -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  annotations:
    eventing.knative.dev/broker.class: MTChannelBasedBroker
  name: kafka-backed-broker
  namespace: default
spec:
  config:
    apiVersion: v1
    kind: ConfigMap
    name: kafka-channel
    namespace: knative-eventing
EOF
```