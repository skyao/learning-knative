---
date: 2020-07-02T15:29:00+08:00
title: Broker 和 Trigger
description : "介绍 Knative 中的 Broker 和 Trigger"
weight: 440
---

Broker 和 Trigger

https://knative.dev/docs/eventing/broker/

## Broker

Broker代表了 "Event Mesh"。事件被发送到Broker的入口（Ingress），然后被发送到对该事件感兴趣的任何订阅者。一旦进入Broker，除CloudEvent之外的所有元数据都会被剥离（例如，除非设置为CloudEvent属性，否则没有该事件如何进入Broker的概念）。

可以有不同类别的Broker，围绕事件的耐久性、性能等提供不同类型的语义。这些例子中使用了Knative Eventing repo中的Broker，它使用Knative Channels来投递事件。

简单的例子，显示broker的配置是在ConfigMap config-br-default-channel中指定的，它使用InMemoryChannel。

例子：

```yaml
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  annotations:
    eventing.knative.dev/broker.class: MTChannelBasedBroker
  name: default
spec:
  # Configuration specific to this broker.
  config:
    apiVersion: v1
    kind: ConfigMap
    name: config-br-default-channel
    namespace: knative-eventing
```

更复杂的例子，显示了与上面相同的Broker，但失败的事件被传送到名为dlq-service的Knative服务。

```yaml
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  annotations:
    eventing.knative.dev/broker.class: MTChannelBasedBroker
  name: default
spec:
  # Configuration specific to this broker.
  config:
    apiVersion: v1
    kind: ConfigMap
    name: config-br-default-channel
    namespace: knative-eventing
  # Where to deliver Events that failed to be processed.
  delivery:
    deadLetterSink:
      ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: dlq-service
```

### 使用默认值创建broker

Knative Eventing提供了一个ConfigMap，默认情况下，这个ConfigMap位于knative-eventing命名空间，被称为default-br-config。开箱即用，它被配置为创建基于MT channel的broker。如果您使用的是不同的Broker实现，您应该相应地修改ConfigMap。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-br-defaults
  namespace: knative-eventing
  labels:
    eventing.knative.dev/release: v0.16.0
data:
  # Configuration for defaulting channels that do not specify CRD implementations.
  default-br-config: |
    clusterDefault:
      brokerClass: MTChannelBasedBroker
      apiVersion: v1
      kind: ConfigMap
      name: config-br-default-channel
      namespace: knative-eventing
```

有了这个ConfigMap，任何创建的Broker都将被配置为使用MTChannelBasedBroker，并且Broker.Spec.Config将按照clusterDefault配置中指定的配置。所以，下面的例子将在默认命名空间中创建一个名为default的Broker，并使用MTChannelBasedBroker作为实现。

```shell
kubectl create -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  name: default
  namespace: default
EOF
```

在defaults-br-config中指定的默认值将导致以下broker被创建。

```yaml
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
```

## Trigger

Trigger 代表了从特定的Broker订阅事件的愿望。

一个简单的例子，它将从给定的（default）broker那里接收所有的事件，并将它们传递给Knative服务my-service。

```shell
kubectl create -f - <<EOF
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: my-service-trigger
spec:
  broker: default
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: my-service
EOF
```











