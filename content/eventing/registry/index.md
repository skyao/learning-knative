---
date: 2020-07-02T15:29:00+08:00
title: Event Registry
description : "介绍 Knative 中的 Event Registry"
weight: 430
---

Eventing Registry

https://knative.dev/docs/eventing/event-registry/

### 概述

Event Registry 维护着一个事件类型目录，这些事件可以从不同Broker中消费。Event Registry 引入了一个新的 EventType CRD，以便将事件类型的信息持久化在集群的数据存储中。

### 利用Registry发现事件

使用Registry，可以发现能够从Broker的事件网格中消费的不同类型的事件。该Registry是为与Broker/Trigger模型一起使用而设计的，旨在帮助创建Triggers。

要查看可订阅的事件类型，请输入以下命令：

```bash
kubectl get eventtypes -n <namespace>
```

下面，我们展示了一个在测试集群中使用 default 命名空间执行上述命令的输出示例。

```
NAME                                         TYPE                                    SOURCE                                                               SCHEMA        BROKER     DESCRIPTION     READY     REASON
dev.knative.source.github.push-34cnb         dev.knative.source.github.push          https://github.com/knative/eventing                                                default                    True
dev.knative.source.github.push-44svn         dev.knative.source.github.push          https://github.com/knative/serving                                                 default                    True
dev.knative.source.github.pullrequest-86jhv  dev.knative.source.github.pull_request  https://github.com/knative/eventing                                                default                    True
dev.knative.source.github.pullrequest-97shf  dev.knative.source.github.pull_request  https://github.com/knative/serving                                                 default                    True
dev.knative.kafka.event-cjvcr                dev.knative.kafka.event                 /apis/v1/namespaces/default/kafkasources/kafka-sample#news                         default                    True
dev.knative.kafka.event-tdt48                dev.knative.kafka.event                 /apis/v1/namespaces/default/kafkasources/kafka-sample#knative-demo                 default                    True
google.pubsub.topic.publish-hrxhh            google.pubsub.topic.publish             //pubsub.googleapis.com/knative/topics/testing                                     dev                        False     BrokerIsNotReady
```

我们可以看到，在 default 命名空间的注册表中，有七个不同的EventTypes。我们选取第一个，看看EventType的yaml是什么样子:

```bash
kubectl get eventtype dev.knative.source.github.push-34cnb -o yaml
```

省略不相关的字段：

```yaml
apiVersion: eventing.knative.dev/v1
kind: EventType
metadata:
  name: dev.knative.source.github.push-34cnb
  namespace: default
  generateName: dev.knative.source.github.push-
spec:
  type: dev.knative.source.github.push
  source: https://github.com/knative/eventing
  schema:
  description:
  broker: default
status:
  conditions:
    - status: "True"
      type: BrokerExists
    - status: "True"
      type: BrokerReady
    - status: "True"
      type: Ready
```

从消费者的角度来看，最重要的字段是 spec 还有 status 字段。

名字/name是咨询性的（即非权威性的），我们通常会生成它（generateName）以避免命名冲突（例如，两个EventTypes在两个不同的Github仓库上监听拉动请求）。由于消费者创建Triggers时既不需要name，也不需要generateName，所以我们把它们的讨论推迟到以后。

关于状态/status，它的主要目的是告诉消费者（或集群 operator）该EventType是否准备好被消费。这个准备状态是基于Broker已经准备好的。我们可以从示例输出中看到，PubSub EventType还没有准备好，因为它的dev Broker还没有准备好。

我们来详细谈谈 spec 字段：

- type：是权威的。这是指 CloudEvent 类型，因为它进入了事件网格。它是强制性的。事件消费者可以（并且在大多数情况下会）根据此属性创建过滤的 Triggers。

- source：指的是进入事件网格的 CloudEvent source。该属性是强制性的。事件消费者可以（大多数情况下会）在此属性上创建触发器来进行过滤。

- schema：是具有 EventType schema的有效 URI。它可以是JSON schema、protobuf schema等。可选的。

- description：是描述EventType的字符串。它是可选的。

- broker指的是可以提供EventType的Broker。它是强制性的。

### 订阅事件

现在您知道了哪些事件可以从 Brokers 的事件网格中消费，您可以创建 Triggers 来订阅特定事件。

以下是几个 Triggers 示例，根据上述注册表输出，使用 type 和/或 source 的精确匹配来订阅事件。

1. 订阅任意source的GitHub推送。

   ```yaml
   apiVersion: eventing.knative.dev/v1
   kind: Trigger
   metadata:
     name: push-trigger
     namespace: default
   spec:
     broker: default
     filter:
       attributes:
         type: dev.knative.source.github.push
     subscriber:
       ref:
         apiVersion: serving.knative.dev/v1
         kind: Service
         name: push-service
   ```
   
   根据上面的注册表输出，该特定类型的事件只存在两个source（knative的*eventing*和*serving*仓库）。如果以后有新的源注册为GitHub推送，这个触发器将能够消费它们。
   
2. 从knative的eventing仓库中订阅GitHub pull请求

	```yaml
	apiVersion: eventing.knative.dev/v1
	kind: Trigger
	metadata:
	  name: gh-knative-eventing-pull-trigger
	  namespace: default
	spec:
	  broker: default
	  filter:
	    attributes:
	      type: dev.knative.source.github.pull_request
	      source: https://github.com/knative/eventing
	  subscriber:
	    ref:
	      apiVersion: serving.knative.dev/v1
	      kind: Service
	      name: gh-knative-eventing-pull-service
	```

3. 订阅发送到knative-demo topic的Kafka消息

	```yaml
	apiVersion: eventing.knative.dev/v1
	kind: Trigger
	metadata:
	  name: kafka-knative-demo-trigger
	  namespace: default
	spec:
	  broker: default
	  filter:
	    attributes:
	      type: dev.knative.kafka.event
	      source: /apis/v1/namespaces/default/kafkasources/kafka-sample#knative-demo
	  subscriber:
	    ref:
	      apiVersion: serving.knative.dev/v1
	      kind: Service
	      name: kafka-knative-demo-service
	```

4. 订阅从GCP的knative项目发送到测试主题的PubSub消息。

	```yaml
	apiVersion: eventing.knative.dev/v1
	kind: Trigger
	metadata:
	  name: gcp-pubsub-knative-testing-trigger
	  namespace: default
	spec:
	  broker: dev
	  filter:
	    attributes:
	      source: //pubsub.googleapis.com/knative/topics/testing
	  subscriber:
	    ref:
	      apiVersion: serving.knative.dev/v1
	      kind: Service
	      name: gcp-pubsub-knative-testing-service
	```

	需要注意的是，在Broker准备好之前，该Trigger的订阅者无法消费事件。

### 填充Registry

现在我们知道了如何使用注册表发现事件，以及如何利用这些信息来订阅感兴趣的事件，让我们进入下一个主题。首先我们如何实际填充注册表？

- 手工注册

	为了填充注册表，集群配置人员可以手动注册EventTypes。这意味着配置人员可以简单地应用EventTypes yaml文件，就像应用任何其他Kubernetes资源一样。

	```
	kubectl apply -f <event_type.yaml>
	```

- 自动注册

	由于手工注册可能比较繁琐且容易出错，我们也支持自动注册EventTypes。EventTypes的创建是在实例化Event Source时完成的。目前，我们支持以下事件源的EventTypes自动注册:

   - CronJobSource
   - ApiServerSource
   - Github来源
   - GcpPubSubSource
   - KafkaSource
   - AwsSqsSource

让我们来看一个例子，特别是我们用来填充测试集群中的注册表的KafkaSource样本。下面是yaml的样子:

```yaml
apiVersion: sources.knative.dev/v1beta1
kind: KafkaSource
metadata:
  name: kafka-sample
  namespace: default
spec:
  bootstrapServers:
   - my-cluster-kafka-bootstrap.kafka:9092
  topics:
   - knative-demo
    - news
  sink:
    apiVersion: eventing.knative.dev/v1
    kind: Broker
    name: default
```

在这个讨论中，上面的yaml中的相关信息是sink和topic。我们观察到，sink是Broker的一种。我们目前只支持自动创建指向Broker的Sources实例的EventTypes。关于topic，我们就是用这个来生成EventTypes source字段，它等于CloudEvent source属性。

当你 `kubectl apply` 这个yaml时，KafkaSource `kafka-source-sample` 将被实例化，两个EventTypes将被添加到注册表中（因为有两个topic）。

