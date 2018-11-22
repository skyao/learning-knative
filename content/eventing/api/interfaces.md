---
date: 2018-11-05T18:50:00+08:00
title: 接口契约
weight: 473
menu:
  main:
    parent: "eventing-api"
description : "Event API的接口契约"
---

> 备注：内容来自 https://github.com/knative/eventing/blob/master/docs/spec/interfaces.md

## 可寻址

**可寻址**资源通过网络传输接收事件（目前仅支持HTTP）。 Addressable在成功处理事件（例如，通过将其提交到稳定存储）后返回成功。 当用作Addressable时，仅使用确认或返回代码来确定事件是否已成功处理。 可寻址的一个例子是_Channel_。

### 控制平面

An **Addressable** resource MUST expose a `status.address.hostname` field.
The _hostname_ value is a cluster-resolvable DNS name which is capable of
receiving event deliveries. _Addressable_ resources may be referenced in the
`reply` section of a _Subscription_, and also by other custom resources acting
as an event Source.

**可寻址**资源必须暴露 `status.address.hostname` 字段。 hostname值是一个群集可解析的DNS名称，它能够接收事件传递。 可订阅资源可以在订阅的回复部分中引用，也可以由充当事件源的其他自定义资源引用。

### Data Plane

An **Addressable** resource will only respond to requests with success or
failure.  Any payload (including a valid CloudEvent) returned to the sender
will be ignored. An _Addressable_ may receive the same event multiple times
even if it previously indicated success.

---

## Callable

A **Callable** resource represents an _Addressable_ endpoint which receives
events and optionally returns events to forward downstream. One example of a
_Callable_ is a function. Note that all _Callable_ resources are _Addressable_
(they accept an event and return a status code when completed), but not all
_Addressable_ resources are _Callable_.

### Control Plane

A **Callable** resource MUST expose a `status.address.hostname` field (like
_Addressable_). The _hostname_ value is a cluster-resolvable DNS name which is
capable of receiving event deliveries and returning a resulting event in the
reply.. _Callable_ resources may be referenced in the `subscriber` section of
a _Subscription_.

<!-- TODO(evankanderson):

What other properties separate a callable from an Addressable. We have talked
about using an annotation like `eventing.knative.dev/returnType = any` to
represent the return type of the _Callable_.

--->

### Data Plane

The **Callable** resource receives one event and returns zero or more events
in response. The returned events are not required to be related to the received
event. The _Callable_ should return a successful response if the event was
processed successfully.

The _Callable_ is not responsible for ensuring successful delivery of any
received or returned event. It may receive the same event multiple times even
if it previously indicated success.