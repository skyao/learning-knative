---
date: 2018-10-24T14:25:00+08:00
title: 自动伸缩
weight: 331
menu:
  main:
    parent: "serving-scaling"
description : "Autoscaling"
---

> 备注：内容来自 https://github.com/knative/serving/blob/master/docs/scaling/DEVELOPMENT.md

Knative Serving Revisions会根据传入流量自动调整大小。

## 定义

* Knative Serving **Revision** -- 自定义资源，是用户代码（在Container中）和配置的一个运行时快照。
* Knative Serving **Route** -- 自定义资源，通过Istio ingress规则向客户端暴露Revisions。
* Kubernetes **Deployment** -- 一个k8s资源，用于管理运行Container的各个Pod的生命周期。 其中之一在每个Revision中运行用户代码。
* Knative Serving **Autoscaler** -- 另一个k8s Deployment，运行单个Pod，用于监视运行用户代码的Pod上的请求负载。它增加和减少运行用户代码的Deployment的大小，以便应对更高或更低的流量负载。
* Knative Serving **Activator** -- 一个k8s Deployment，运行单个多租户Pod（每个集群一个，可用于所有Revision），发给没有Pod的Revision的请求会被Activator捕获。 它启动运行用户代码的Pods（通过Revision controller）并转发捕获的请求。
* **Concurrency** -- 当前在特定时刻服务的请求数。 更多QPS或更高的延迟意味着更多并发请求。

## 行为

Revision 有三种 autoscaling 状态：

1. **Active/活跃** 当他们活跃地服务请求时，
2. **Reserve/保留** 当他们缩小到0个 Pod但仍然在服务，还有
3. **Retired/退役** 当他们将不再接收流量。

当Revision活跃并为请求提供服务时，它将增加和减少Pod的数量，以维持每个Pod所需的平均并发请求。当不再提供请求时，修订版将处于保留状态。 当第一个请求到达时，Revision将处于Active状态，请求将排队，直到Revision准备就绪。

在Active状态下，每个Revision都有一个Deployment，用以维护所需数量的Pod。 它还有一个Autoscaler（单租户时每个Revision一个;多租户时所有Revision用一个），Autoscaler监视流量指标并调整部署所需的pod数量。每个Pod每秒钟向Autoscaler报告其并发请求数。

在Reserve状态下，Revision没有预定的Pod并且不使用CPU。Revision的Istio路由规则指向单个多租户Activator，Activator将捕获所有到Reserve Revision的流量。当Activator捕获一个Reserve Revision的请求时，它会将Revision修改为Active状态，然后在准备就绪时将请求转发给Revision。

在Retired状态下，Revision已配置资源（备注：怀疑文档有无，应该是不配置资源吧？）。 Revision不为任何请求提供服务。

注意: Retired 状态当前没有在任何地方设置. 见 [issue 1203](https://github.com/knative/serving/issues/1203).

> 备注：看到最新情况是ServingStatus被完整的从Revision中删除，不仅仅是第三个Retired状态。具体见 [Remove ServingState from Revision](https://github.com/knative/serving/pull/2301)

## Context

下图说明了autoscaler的机制：

```diagram
   +---------------------+
   | ROUTE               |
   |                     |
   |   +-------------+   |
   |   | Istio Route |---------------+
   |   +-------------+   |           |
   |         |           |           |
   +---------|-----------+           |
             |                       |
             |                       |
             | inactive              | active
             |  route                | route
             |                       |
             |                       |
             |                +------|---------------------------------+
             V         watch  |      V                                 |
       +-----------+   first  |   +- ----+  create   +------------+    |
       | Activator |------------->| Pods |<----------| Deployment |<--------------+
       +-----------+          |   +------+           +------------+    |          |
             |                |       |                                |          | resize
             |   activate     |       |                                |          |
             +--------------->|       |                                |          |
                              |       |               metrics          |   +------------+
                              |       +----------------------------------->| Autoscaler |
                              |                                        |   +------------+
                              |                                        |
                              | REVISION                               |
                              +----------------------------------------+

```

## 设计目标

1. **快速**.  Revision应该能够在30秒或更短的时间内从0扩展到1000个并发请求。
2. **轻量**.  只要有可能，系统就应该能够正确处理，不需要用户干预或配置。
3. **让一切变得更好**.  创建自定义组件是一种短期策略，可以立即工作。长期策略是使底层组件更好，以便可以用配置替换自定义代码。例如. Autoscaler 应该替换为 K8s [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) 和 [Custom Metrics](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-custom-metrics).

### Slow Brain / Fast Brain

Knative Serving Autoscaler分为两部分：

1. **Fast Brain** 维持每个Pod所需的并发请求级别  (满足 [设计目标 #1](#设计目标))，而
2. **Slow Brain** 根据CPU，内存和延迟统计信息提出所需的级别 (满足 [设计目标 #2](#设计目标)).

## Fast Brain 实现

随着Knative Serving实施的变化，这可能会发生变化。

### Autoscaler

Knative Serving Pods中有一个代理 (`queue-proxy`)，负责执行请求队列参数 (单线程或者多线程)，并向Autoscaler报告并发客户端指标。如果我们可以去掉它而只用 [Envoy](https://www.envoyproxy.io/docs/envoy/latest/)，那太好了(见 [设计目标 #3](#设计目标)).   Knative Serving Controller将Revision的标识注入queue-proxy环境变量。当queue-proxy唤醒时，它将找到Revision的Autoscaler并建立websocket连接。每隔1秒，queue-proxy就推送一个gob序列化结构，其中包含当时观察到的并发请求数。

Autoscaler 运行 controller 来监控 ["KPA"](https://github.com/knative/serving/blob/master/pkg/apis/autoscaling/v1alpha1/kpa_types.go) 资源并通过`/scale` 子资源监控和伸缩内嵌对象引用。

Autoscaler提供支持websocket的统计服务器。queue-proxy将其度量发送到Autoscaler的统计服务器，Autoscaler维护一个60秒的数据点滑动窗口。

Autoscaler实现了两种操作模式的缩放算法：Stable/稳定模式和Panic/恐慌模式。

#### Stabl/稳定模式

在稳定模式下，Autoscaler会调整Deployment的大小，以实现每Pod的所需平均并发 (当前 [硬编码](https://github.com/knative/serving/blob/c4a543ecce61f5cac96b0e334e57db305ff4bcb3/cmd/autoscaler/main.go#L36), 稍后通过Slow Brain提供).  它通过平均60秒窗口内的所有数据点来计算观察到的每个pod的并发性。当它调整Deployment的大小时，所需Pod的计数是基于度量流中观察到的Pod数量，而不是Deployment规范中Pod的数量。 这对于防止Autoscaler失效很重要（在Pod计数增加和新Pod上线以服务请求并提供指标流之间存在延迟）

#### Panic/恐慌模式

The Autoscaler evaluates its metrics every 2 seconds.  In addition to the 60-second window, it also keeps a 6-second window (the panic window).  If the 6-second average concurrency reaches 2 times the desired average, then the Autoscaler transitions into Panic Mode.  In Panic Mode the Autoscaler bases all its decisions on the 6-second window, which makes it much more responsive to sudden increases in traffic.  Every 2 seconds it adjusts the size of the Deployment to achieve the stable, desired average (or a maximum of 10 times the current observed Pod count, whichever is smaller).  To prevent rapid fluctuations in the Pod count, the Autoscaler will only increase Deployment size during Panic Mode, never decrease.  60 seconds after the last Panic Mode increase to the Deployment size, the Autoscaler transistions back to Stable Mode and begins evaluating the 60-second windows again.

Autoscaler每2秒评估一次其指标。除了60秒的窗口，它还保持一个6秒的窗口（恐慌窗口）。如果6秒平均并发性达到所需平均值的2倍，则Autoscaler将转换为恐慌模式。在恐慌模式下，Autoscaler将其所有决策都基于6秒窗口，这使得它对流量的突然增加更加敏感。每隔2秒，它会调整部署的大小，以达到稳定的所需平均值（或最大值为当前观察到的Pod数量的10倍，以较小者为准）。为了防止Pod计数的快速波动，Autoscaler只会在恐慌模式下增加部署大小，永不减少。在最后一次Panic Mode增加到部署大小后60秒，Autoscaler转换回稳定模式并再次开始评估60秒窗口。

#### Deactivation

When the Autoscaler has observed an average concurrency per pod of 0.0 for some time ([#305](https://github.com/knative/serving/issues/305)), it will transistion the Revision into the Reserve state.  This scales the Deployment to 0, stops any single tenant Autoscaler associated with the Revision, and routes all traffic for the Revision to the Activator.

### Activator

The Activator is a single multi-tenant component that catches traffic for all Reserve Revisions.  It is responsible for activating the Revisions and then proxying the caught requests to the appropriate Pods.  It woud be preferable to have a hook in Istio to do this so we can get rid of the Activator (see [Design Goal #3](#design-goals)).  When the Activator gets a request for a Reserve Revision, it calls the Knative Serving control plane to transistion the Revision to an Active state.  It will take a few seconds for all the resources to be provisioned, so more requests might arrive at the Activator in the meantime.  The Activator establishes a watch for Pods belonging to the target Revision.  Once the first Pod comes up, all enqueued requests are proxied to that Pod.  Concurrently, the Knative Serving control plane will update the Istio route rules to take the Activator back out of the serving path.

## Slow Brain Implementation

*Currently the Slow Brain is not implemented and the desired concurrency level is hardcoded at 1.0 ([code](https://github.com/knative/serving/blob/7f1385cb88ca660378f8afcc78ad4bfcddd83c47/cmd/autoscaler/main.go#L36)).*

