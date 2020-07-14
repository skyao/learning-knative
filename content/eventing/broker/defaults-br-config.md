---
date: 2018-10-24T14:20:00+08:00
title: defaults-br-config
weight: 441
menu:
  main:
    parent: "eventing-broker"
description : "knative的defaults-br-config"
---

https://knative.dev/docs/eventing/broker/defaults-br-config/

## 默认br-config

Knative提供了一个ConfigMap，通过在创建Broker时提供默认值，使得创建Broker更加容易。你可以通过修改这个文件来控制使用哪些Broker实现以及如何配置它们。您可以灵活地控制集群级别（集群中创建的每个Broker）以及命名空间级别（覆盖某些命名空间的行为）的默认值。

### 文件的格式

让我们来看看安装发行版（本例中为v0.16.0）时开箱即用的ConfigMap。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-br-defaults
  namespace: knative-eventing
  labels:
    eventing.knative.dev/release: devel
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

这意味着任何没有特定的BrokerClass注解或spec.config的Broker的创建都将使用MTChannelBasedBroker实现，并且该Broker的spec.config将像这样。

```
spec:
  config:
    apiVersion: v1
    kind: ConfigMap
    name: config-br-default-channel
    namespace: knative-eventing
```

### 更改默认的BrokerClass

#### 更改集群的默认BrokerClass

如果您安装了一个或多个不同的broker，您可以在集群级别和命名空间更改默认的broker。例如，如果您安装了MT Channel Based Broker 如 `YourBroker`，并且希望在默认情况下创建的任何broker都使用YourBroker，您可以修改 configmap ，使其看起来像这样。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-br-defaults
  namespace: knative-eventing
  labels:
    eventing.knative.dev/release: devel
data:
  # Configuration for defaulting channels that do not specify CRD implementations.
  default-br-config: |
    clusterDefault:
      brokerClass: YourBroker
```

现在在集群中创建的每一个没有BrokerClass注解的Broker都将使用YourBroker作为Broker的实现。请注意，您可以在创建Broker时，通过明确指定BrokerClass注解来使用不同的实现。

#### 更改命名空间的默认BrokerClass

如前所述，您还可以控制某些命名空间集的默认行为。因此，举例来说，如果你想为所有其他创建的Broker使用YourBroker，但想为以下命名空间使用MTChannelBasedBroker：namespace1和namespace2。你可以像这样修改configmap：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-br-defaults
  namespace: knative-eventing
  labels:
    eventing.knative.dev/release: devel
data:
  # Configuration for defaulting channels that do not specify CRD implementations.
  default-br-config: |
    clusterDefault:
      brokerClass: YourBroker
    namespaceDefaults:
      namespace1:
        brokerClass: MTChannelBasedBroker
      namespace2:
        brokerClass: MTChannelBasedBroker
```

### 更改Broker的默认配置

#### 更改集群的默认配置

你也可以通过指定默认值到broker.spec.config中，如果创建时留空，来控制Broker配置的默认行为。

如果你已经安装了不同的通道实现(例如Kafka)，并且默认情况下你想为任何创建的Broker使用该通道，你可以将ConfigMap改成这样。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-br-defaults
  namespace: knative-eventing
  labels:
    eventing.knative.dev/release: devel
data:
  # Configuration for defaulting channels that do not specify CRD implementations.
  default-br-config: |
    clusterDefault:
      brokerClass: MTChannelBasedBroker
      apiVersion: v1
      kind: ConfigMap
      name: config-kafka-channel
      namespace: knative-eventing
```

现在，在集群中创建的每一个没有spec.config的Broker将被配置为使用config-kafka-channel ConfigMap。请注意，你仍然可以通过在spec.config中明确地为任何给定的Broker指定不同的配置。

#### 更改命名空间的默认配置

如前所述，你还可以控制一些命名空间集的默认行为。因此，例如，如果你想为所有其他创建的Broker使用config-kafka-channel，但想使用config-br-default-channel配置以下命名空间：namespace3和namespace4。你会像这样修改config map：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-br-defaults
  namespace: knative-eventing
  labels:
    eventing.knative.dev/release: devel
data:
  # Configuration for defaulting channels that do not specify CRD implementations.
  default-br-config: |
    clusterDefault:
      brokerClass: MTChannelBasedBroker
      apiVersion: v1
      kind: ConfigMap
      name: config-kafka-channel
      namespace: knative-eventing
    namespaceDefaults:
      namespace3:
        apiVersion: v1
        kind: ConfigMap
        name: config-br-default-channel
        namespace: knative-eventing
      namespace4:
        apiVersion: v1
        kind: ConfigMap
        name: config-br-default-channel
        namespace: knative-eventing
```

请注意，对于这些命名空间，我们不会覆盖brokerClass。brokerClass和config是可以独立配置的。