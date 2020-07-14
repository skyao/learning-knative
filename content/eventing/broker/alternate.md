---
date: 2018-10-24T14:20:00+08:00
title: Broker替代品
weight: 443
menu:
  main:
    parent: "eventing-broker"
description : "knative的Broker替代品"
---

https://knative.dev/docs/eventing/broker/alternate/



在Knative Eventing生态系统中，只要尊重Broker Conformance Spec，我们欢迎替代的Broker实现。

以下是除了Knative Eventing提供的默认Broker实现之外，由社区或供应商提供的经纪人列表。

#### GCP Broker

GCP Broker 是为在 GCP 中运行而优化的。



## Broker Conformance Spec

https://github.com/knative/eventing/blob/master/docs/spec/broker.md

### 背景

Broker指定一个入口（Ingress），生产者可以向其生产事件。Trigger指定了一个订阅者（subscriber），该订阅者将接收发送到其分配的broker并符合指定过滤器的事件。

### 控制平面

#### Broker

Broker对象应该在其状态中包含一个Ready条件。

当它的入口可以接收事件时，Broker 应该指示Ready=True。

当一个Broker处于Ready状态时，它应该是一个有效的Addressable可寻址对象，并且它的status.address.url字段应该指示它的入口地址。

#### Trigger

Trigger应该在其状态中包含一个Ready条件。

当事件可以传递给它的订阅者时，触发器应该指示Ready=True。

当触发器处于Ready状态时，它应该通过 status.subscriberUri 字段来指示其订阅者的URI。

触发器必须只精确分配给一个Broker。如果用户没有指定Broker，触发器在创建时应该被分配一个默认的Broker。

触发器可以在其被分配的Broker存在之前被创建。当被分配的Broker存在并且是Ready时，触发器应该进入Ready状态。

Trigger必须支持指定键值对列表的属性过滤器。通过属性筛选器的事件必须包含与所有键值对完全匹配的上下文或扩展属性。

### 数据平面

#### Ingress

Broker应该暴露一个HTTP或HTTPS端点作为入口。它可以同时暴露两个端点。

入口端点必须至少符合以下一个版本的规范：

- CloudEvents 0.3 规范

- CloudEvents 1.0规范

其他版本可能会被拒绝。建议使用CloudEvents 1.0版本。

Broker不应该对所产生的事件的CloudEvents版本进行升级。它应该支持 CloudEvents 的 HTTP 协议绑定的二进制内容模式和结构化内容模式。

HTTP(S)端点可以是任何端口，而不仅仅是标准的80和443。除 HTTP 外，Channel还可自行决定暴露其他非 HTTP 端点（例如暴露 gRPC 端点以接受事件）。

经纪人必须拒绝所有使用 POST 以外的方法的 HTTP 生产请求，并以 HTTP 状态码 405 Method Not Supported 响应。非事件队列请求（如健康检查）不受限制。

如果produce请求被接受，Broker必须用200 HTTP状态码响应。

如果 Broker 收到 produce 请求，但无法解析有效的 CloudEvent，那么它必须使用 HTTP 状态码 400 Bad Request 拒绝该请求。

#### 投递

投递的事件必须符合 CloudEvents 规范。由生产者设置的所有 CloudEvent 属性（包括数据和规格版本属性）必须以与Broker接收的方式相同的方式从用户处接收。

Broker应支持通过 CloudEvents 的 HTTP 协议绑定的二进制内容模式或结构化内容模式投递事件。

Broker接受的事件至少应向所有触发器的所有订阅者传递一次（at least once），这些触发器： 

1. 收到生产的请求时已准备就绪
2. 指定与该事件相匹配的过滤器，以及
3. 存在时，事件可以被交付。

事件可以额外投递给在事件被接受后才Ready的触发器。

在事件从被生产者接受到投递给订阅者之间，事件可以被排队或延迟。

Broker可以选择不投递一个事件，由于持续的用户不可用或诸如存储容量的限制。在这种情况下，Broker应该尝试通知运营商。Broker可以将这些事件转发到一个替代的终端或存储机制，如死信队列。

如果没有准备好的Trigger与接受的事件相匹配，Broker可以在不通知生产者的情况下放弃该事件。从生产者的角度来看，该事件成功地交付给了零个订阅者。

如果多个Triggers引用同一个订阅者，订阅者可能会被期望多次确认事件的成功投递。

投递响应中包含的事件应该被发布到Broker入口，并被处理，就像该事件已经被生产到Broker的addressable可寻址端点一样。

投递响应中包含的事件如果是畸形的，应该将被当作事件投递失败来处理。推理是，如果事件被转换不成功（例如编程错误），它应该被视为失败。

订阅者可能会收到一个确认，即Reply事件被Broker接受。如果Reply事件未被接受，初始事件应该被重新投递给订阅者。

### 可观察性

Broker应该公开各种指标，包括但不限于：

1. 畸形生产请求的数量(400系列响应)
2. 被接受的生产请求数量（200系列答复）
3. 投递的事件数量

默认情况下应启用指标，如果需要，可通过包含的配置参数禁用指标。

在收到具有 CloudEvents 分布式跟踪扩展中定义的上下文属性的事件后，除非回复事件使用不同的跟踪属性集进行发送，否则 Broker 应在向订阅者和回复事件中保留该跟踪头。转发的跟踪头应该与broker发出的任何中间跨度一起更新。

Broker发出的跨距应尽可能遵循消息系统的OpenTelemetry语义约定。特别是，由broker发出的span应该设置以下属性。

- messaging.system: "knative"
- messaging.destination：broker:name.namespace或trigger:name.namespace，包含事件被路由到的Broker或Trigger。
- messaging.protocol：底层传输协议的名称。
- messaging.message_id：事件ID。

### 例外

这些方面的规范在本大纲中没有测试。

- 未能发布的回复会导致初始消息被重新发送。需要特定的实现设置来诱发失败。
- 度量支持。目前没有共享格式可以用来测试对度量的支持。



