---
date: 2019-05-07T18:00:00+08:00
title: Runtime Contract文档
weight: 351
menu:
  main:
    parent: "runtime-contract"
description : "Knative Runtime Contract文档"
---

> 备注：内容摘要自 https://github.com/knative/serving/blob/master/docs/runtime-contract.md

## 抽象

Knative serverless 计算基础设施扩展了 [Open Container Initiative运行时规范](https://github.com/opencontainers/runtime-spec/blob/master/spec.md) ，以描述serverless执行工作负载的功能和要求。与通用容器相比，无状态的、请求触发（如按需）、可自动调整的容器具有以下属性：

- 很少或没有长期的运行时状态（特别是在没有请求流量的情况下，代码可以收缩到零的情况）。
- 记录和监视聚合（遥测）对于理解和调试系统很重要，因为可以随时创建或删除容器以响应自动伸缩。
- 多租户迫切要求允许在相对稳定的底层硬件资源上共享突发应用的成本。

除了通过 [引用Knative Kubernetes资源](https://github.com/knative/serving/blob/master/docs/spec/spec.md) 之外，此契约不会在运行时环境之上定义控制面。同样，除了提供日志数据集合的合约之外，此合同不定义度量或日志聚合的实现。预计平台运营商将提供对聚合遥测的访问。

## 背景

[OCI规范](https://github.com/opencontainers/runtime-spec/blob/master/spec.md) （[V1.0.1](https://github.com/opencontainers/runtime-spec/blob/v1.0.1/spec.md)）是本文件的基础。当本文档与OCI规范发生冲突时，则假定本文档覆盖了通用的OCI建议。在本文档未指定行为的情况下，开发人员应假设符合OCI的底层实现。此外，核心Knative定义假定 [Linux容器配置](https://github.com/opencontainers/runtime-spec/blob/master/config-linux.md)。

特别是，默认的 Knative 实现依赖于 Kubernetes 的行为来实现容器操作。在某些情况下，当前2018年的Kubernetes行为不如本文档中推荐的那样高。Knative作者的目标是尽可能多地将所需功能推入 Kubernetes 和/或 Istio，而不是实现reach-around（？）层。

本文档考虑了给定 Knative 环境的两种用户，并特别关注在环境中运行代码的开发人员（还有语言和工具开发人员）的期望。

- **开发人员** 编写代码，打包到容器中，在Knative集群上运行
	- **语言和工具开发人员** 通常编写开发人员用来将代码打包到容器中的工具。因此，他们关注包装开发人员代码、符合此运行时合约的工具。
- **运营商**（也称为**平台提供商**）提供计算资源并管理Knative和底层抽象的软件配置（例如，Linux，Kubernetes，Istio等）。

## 运行时和生命周期

Knative旨在最大限度地减少运行服务所需的调优和生产配置。其中一些易于生产的功能包括：

1. 请求规模或事件规模粒度的无状态计算。
2. 在0和许多实例之间自动缩放（流程横向扩展模型）。
3. 在可能的情况下，根据观察到的行为自动调整资源需求。

为了实现这些属性，作为 serverless 平台的一部分运行的容器应该遵守以下属性：

- 快速启动时间（不超过1秒就内可以处理请求或事件，假设容器镜像层有缓存）
- 最小化本地状态（以支持自动伸缩并收缩到零）
- 仅在有活动请求时的使用CPU

### 状态

一般的 OCI 状态可能无法在容器内进行内省，并且可能仅对系统运维或平台提供者可见。在高度共享的环境中，容器可能会遇到以下情况：

- `status` 为 `stopped` 的容器可以由系统立即回收。
- 容器进程可以以pid 0启动，通过使用PID命名空间或其他进程。

### 生命周期

- 当容器处于非活动状态时，容器可能会被杀死。Serverless计算依赖于入站请求来确定容器的活动性。当容器通过OCI规范的 kill 命令被杀死时，容器会被发送一个 SIGTERM 信号 ，以便优雅关闭现有的资源和连接。如果容器在规定的宽限期后没有关闭，则通过 SIGKILL 信号强制杀死容器。

- 环境可能限制使用 prestart，poststart 以及 poststop 来挂钩平台运营商，而不是开发者。所有这些钩子都在运行时命名空间的上下文中定义，而不是在容器命名空间中定义，并且可能暴露系统级信息（并且是非可移植的）。

- 开发人员指定进程的失败必须记录到开发人员可见的日志系统中。

此外，某些 serverless 环境可能使用 linux 中除 docker 之外的执行模型（例如，runv或 Kata Containers）。在 linux 中使用 docker 之外的执行模型的实现可能会改变生命周期契约，超出OCI规范，只要：

1. 默认符合 OCI 生命周期契约，无论提供多少扩展。

2. 扩展执行模型或生命周期的实现必须提供有关扩展模型或生命周期的文档以及有关如何选择加入扩展生命周期契约的文档。

### 错误

- 平台可以提供事后从特定执行中查看文件系统内容的机制。因为容器（特别是出现故障的容器）可能经常启动，所以运维或平台提供商应该限制这些故障所消耗的总空间。
- 容器默认应在 `/dev/termination-log` 中写入自己的终止消息。如果容器没有写入消息，则最后几行日志输出应报告为执行错误（即通过 [设置`terminationMessagePolicy`为`FallbackToLogsOnError`](https://kubernetes.io/docs/tasks/debug-application-cluster/determine-reason-pod-failure/#customizing-the-termination-message)）。

### 警告

由OCI指定。

### 操作

[OCI接口](https://github.com/opencontainers/runtime-spec/blob/v1.0.0-rc3/runtime.md#operations) 不应在容器内暴露。运维或平台提供商可以直接与OCI接口进行交互，但这超出了本规范的范围。

可以向开发人员公开调用 `kill` 操作的可选方法，以向容器提供信令。

### 钩子

运维钩子不应该由Knative开发人员配置。运营或平台提供商可以使用钩子来实现他们自己的生命周期控制。

### Linux运行时

#### 文件描述符

从容器上的 `stdin` 文件描述符读取应该总是导致 `EOF`。容器上的`stdout`和`stderr`文件描述符应当收集和保留在开发人员可以访问的日志库中。

在容器内，管道和文件描述符可用于在同一容器中运行的进程之间进行通信。

#### 开发符号链接

由OCI指定。