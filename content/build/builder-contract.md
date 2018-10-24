---
date: 2018-10-24T14:21:00+08:00
title: Builders契约
weight: 220
menu:
  main:
    parent: "build"
description : "介绍Knative Builders契约"
---

> 备注：文章内容来自 https://github.com/knative/docs/blob/master/build/builder-contract.md

本文档定义了Builder镜像以及它们应遵循的约定。

## 什么是Builder?

Builder镜像是镜像的特殊分类，这些镜像可以作为Build CRD步骤的一部分运行：

例如, 在下面的build中，镜像 `gcr.io/cloud-builders/gcloud` 和 `gcr.io/cloud-builders/docker` 是 "Builders".:

```yaml
spec:
  steps:
  - image: gcr.io/cloud-builders/gcloud
    ...
  - image: gcr.io/cloud-builders/docker
    ...
```

### 典型Builders

Builder通常是一个专门构建的容器，其入口点是一个执行某些操作的工具，并在成功时以零状态退出。这些入口点通常是命令行工具，例如git，docker，mvn等。

典型的builder设置他们的`command:`:(又名`ENTRYPOINT`）作为他们包装的命令并期望使用 `args:`指导他们的行为。

builder的例子可以参考：

- [GoogleCloudPlatform/cloud-builders](https://github.com/googlecloudplatform/cloud-builders)
- [GoogleCloudPlatform/cloud-builders-community](https://github.com/googlecloudplatform/cloud-builders-community)

例如 cloud-builders/curl/ 的Dockerfile：

```dockerfile
FROM launcher.gcr.io/google/ubuntu16_04

RUN apt-get update && \
  apt-get -y install curl ca-certificates

ENTRYPOINT ["curl"]
```

### 非典型Builders

尽管通过覆盖`command:`和`args:`来实现Builder约定不太常见，但也是可以的，例如：

```yaml
steps:
- image: ubuntu
  command: ['/bin/bash']
  args: ['-c', 'echo hello $FOO']
  env:
  - name: 'FOO'
    value: 'world'
```

### 专用Builders

高级用户也可以创建专用的builder。


## 什么是Builder约定?

Builders期望Build遵循以下约定：

 * `/workspace`: 默认工作目录将是`/workspace`，这是一个由`source:` step填充并在build `steps:`之间共享的卷。
 * `/builder/home`: 这个卷通过`$HOME`暴露给`steps`。
 * 附加到Build的服务帐户的凭据可能会公开为Git或Docker凭据。

