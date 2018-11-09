---
date: 2018-10-24T14:20:00+08:00
title: 动机和目标
weight: 471
menu:
  main:
    parent: "eventing-api"
description : "Event API的动机和目标"
---

> 备注：内容来自 https://github.com/knative/eventing/blob/master/docs/spec/motivation.md

Knative Eventing的目标是定义通用的可组合原语，以启用后绑定的事件源和事件消费者。

Knative Eventing 有以下原则:

1. 服务在开发期间松散耦合并且在不同平台（kubernetes，虚拟机，SaaS 或者 PaaS）上独立部署

1. 生产者可以在消费者收听之前生成事件，并且消费者可以表达对尚未生成的事件或事件类别的兴趣。

1. 可以连接服务以创建新的应用程序



    * 无需修改生产者或消费者，而且
    * 能够从特定的生产者中选择特定的事件子集。

这些原语可以以解耦的方式生产和消费符合 CloudEvents 规范的事件。

Kubernetes没有与事件处理相关的原语，但这是 serverless 工作负载中必不可少的组件。 Eventing 引入了用于事件生成和交付的高级原语，初始聚焦在通过HTTP推送。如果您的应用程序需要新的事件源或类型，那么将它们引入现有事件框架并将与 CloudEvents 中间件和消息消费者集成所需的工作量将是最小的。

Knative eventing 实现事件传递生态系统的公共组件：事件源的枚举和发现，事件传输的配置和管理，以及事件的声明性绑定（由存储服务或早期计算生成），以便进行进一步的事件处理和持久性。

Knative Eventing API旨在独立运维，并与 Serving API 和 Build API 良好互操作。