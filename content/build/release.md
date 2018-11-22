---
date: 2018-11-14T22:00:00+08:00
title: Build版本发布
weight: 201
menu:
  main:
    parent: "build"
description : "Build版本发布"
---

用于跟踪knative build的release情况。

原始信息来自 https://github.com/knative/build/releases

## 最新版本

### v0.2

发布于2018-10-31。

**新特性**

新资源: 集群范围的 Build 模版 ([#302](https://github.com/knative/build/issues/302))
BuildTemplate 参数可以用于构建步骤镜像名称 ([#293](https://github.com/knative/build/pull/293))

Builds 规范现在可以指定：

- build范围的 timeout ([#315](https://github.com/knative/build/pull/315))
- 节点选择器 ([#236](https://github.com/knative/build/pull/236)) 和亲和力 ([#344](https://github.com/knative/build/pull/344))
- 可以从子目录获取资源 ([#245](https://github.com/knative/build/pull/245))

现在报告的Build状态：

- per-step StepStates, updated as the build progresses through its steps ([#309](https://github.com/knative/build/pull/309))
- start time, allowing measurement of pending latency ([#322](https://github.com/knative/build/pull/322))
- step states only for requested steps, ignoring implicit steps ([#340](https://github.com/knative/build/pull/340))


## 历史版本

### v0.1.2

发布于2018-09-18。

release.yaml使用最新版本的`ko`重建，其中包含 [google/go-containerregistry#192](https://github.com/google/go-containerregistry/issues/192) 的修复程序，它似乎使用containerd打破了群集

Cherrypicked cd69f25和c8a0d97包含`creds-init`和`git-init`的较小构建器映像。

这样可以在尚未存在这些镜像的节点上实现更快的构建。

### v0.1.1

发布于2018-08-09。

release.yaml使用最新版本的`ko`重建，其中包含 [google/go-containerregistry#192](https://github.com/google/go-containerregistry/issues/192) 的修复程序，它似乎使用containerd打破了群集

### v0.1.0

发布于2018-07-21。

First Knative Build alpha release!

无Release信息。

