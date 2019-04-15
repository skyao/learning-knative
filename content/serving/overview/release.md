---
date: 2018-11-13T10:35:00+08:00
title: Serving版本发布
weight: 319
menu:
  main:
    parent: "serving-overview"
description : "Serving版本发布"
---

用于跟踪knative serving的release情况。

原始信息来自 https://github.com/knative/serving/releases

## 最新版本

### v0.2.1

同v0.2。

### v0.2

发布于2018-10-31日。

#### Meta

可插入性**：

我们已经在“under the hood”取得了重大进展，封装了knative / serve的主要子系统，以支持可插拔性（例如替换Istio）。我们的核心资源现在通过新的内部API配置这些子系统：Networking，Autoscaling和Caching。

**松耦合**：

我们花了相当大的精力确保所有的组件可以单独使用，并将我们的发布工件拆分为最小的单元。 例如，您现在可以安装和使用knative/serve而无需knative/build，甚至可以插入替代的Build CRD！

我们的发布现在包括：

- `serving.yaml`：只有knative/serving组件。
- `build.yaml`：只有knative/build的0.2.0版本
- `monitoring*.yaml`：监控堆栈的多种不同配置。
- `istio* .yaml`：Istio堆栈的两种配置，一种带有sidecar注入，一种没有。
- `release* .yaml`：与上次发布的类似打包

#### Autoscaling

**新的共享Autoscaler** 

我们已使用单个共享autoscaler替换了以前的每Revision的autoscaler。 此autoscaler基于与先前autoscaler相同的逻辑，但已演变为纯度量驱动（包括0-> 1-> 0），从而消除了不必要的 `Revision` `servingState` 字段。

### 引入 ContainerConcurrency 字段

我们用一个整型字段 `ContainerConcurrency` 替换了 `ConcurrencyModel`（`Single`或`Multi`）。 对于某些用例（例如，有限制的线程池），这允许将并发限制为除1之外的值。

- 1 是新的Single值。
- 0 是新的Multi（无限制）。

示例：

```yaml
spec:
  containerConcurrency: 1
  container:
    image: docker.io/{username}/helloworld-go
```

`ContainerConcurrency` 现在用于确定 autoscaler 的目标并发度。

#### Core API

**解藕Build**

除非您打算使用它，否则运行`Serving`不再需要`Build`。在安装`Build`时，仍支持表达内联构建的旧样式，但不赞成使用以下内容：

```yaml
spec:
  build:
    apiVersion: build.knative.dev/v1alpha1
    kind: Build
    spec:
      # What was previously directly under "build:"
```

此外，可以插入替代的`Build`实现，唯一的要求是那些`Build`资源通过以下方式指示完成：

```yaml
status:
  conditions:
  - type: Succeeded
    status: True | False | Unknown
```

**Revision GC**

`Configuration`现在将基于以下几个标准回收不可路由的 `Revision`：

- 创建以来的时间
- 从上次观察到可路由的时间（通过`serve.knative.dev/lastPinned` annotation heartbeat）
- 年龄（例如保留最后N个）
- 它是`Configuration`的LatestReadyRevision吗？
- 可以在`config/config-gc.yaml`中配置

**其他功能**

- 服务现在支持发布和手动模式。
- Knative资源的简称
- kubectl输出中的自定义列（仅限K8s 1.11+）
- 更长的“resync”时段，对（某些）configmap更改进行全局resync
- 我们现在创建caching.internal.knative.dev/Image 资源来发信号通知对缓存很重要的镜像。 运维必须安装扩展才能利用这些提示。

#### Networking

**ClusterIngress**

`Route`不再直接依赖于`VirtualService`，而是依赖一个中间资源`ClusterIngress`，它可以针对不同的网络平台进行不同的协调。

**Istio更新**

- 迁移到 Istio 1.0.2。
- 由于0.8.0的缺陷而需要的hack和变通被删除。
- 对Route的集群本地访问（通过`route.ns.svc.cluster.local`名称）不再需要istio sidecar。

**Monitoring**

`monitoring`命名空间已更改为`knative-monitoring`。

## 历史版本

### v0.1.1

发布于2018-08-14。

修正：

- 小写header name以修复使用Istio 1.0的验证。
- 更新共享网关TLS设置以使用Istio 1.0验证。
- 更新到Build v0.1.1，以便使用containerd。

### v0.1

发布于2018-07-19。

First Knative Serving Alpha Release!

没有其他release信息。

