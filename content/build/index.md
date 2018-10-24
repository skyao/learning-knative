---
date: 2018-10-24T14:25:00+08:00
title: Build模块
weight: 200
keywords:
- Build模块
description : "介绍Knative的Build模块"
---

## 介绍

Knative build是正在进行的构建系统，旨在满足云原生开发的常见需求。

Knative build扩展了Kubernetes并利用现有的Kubernetes原语为您提供从源代码开始运行群集上容器构建的能力。 例如，您可以编写一个构建，该构建使用Kubernetes原生资源从存储库中获取源代码，将其构建到容器镜像，然后运行该镜像。

虽然Knative build针对构建，测试和部署源代码进行了优化，但您仍然需要负责开发相应的组件：

- 从存储库中检索源代码。
- 针对共享文件系统运行多个连续作业，例如：

	* 安装依赖。
	* 运行单元和集成测试。

- 构建容器镜像。
- 将容器镜像推送到镜像注册，或将它们部署到集群。

Knative build的目标是提供标准的，可移植的，可重用的而且性能优化的方法，用于定义和运行集群上的容器镜像构建。 通过提供在Kubernetes上运行构建这样“无聊但困难”的任务，Knative使您不必独立开发和重现这些通用的基于Kubernetes的开发流程。

虽然迄今为止，Knative build还不提供完整的独立CI / CD解决方案，但它确实提供了一个较低级别的构建块，该构建块专门用于在大型系统中实现集成和利用。

> 备注：以上内容来自 https://github.com/knative/build 

## 实现

`Build`是Knative中的自定义资源，允许您定义可以运行并完成同时还可以提供状态的流程。例如，使用Knative Build来获取，构建和打包代码，该Knative Build会通知流程是否成功。

Knative Build在集群上运行，由Kubernetes自定义资源定义（ Custom Resource Definition/CRD）实现。

对于为执行任务或操作而创建的`Builder`或容器映像，可以通过单个配置文件定义Knative Build。

还可以考虑使用Knative Build将应用程序的源代码构建到容器镜像中，然后可以在Knative serving上运行。 

### Knative Builds的关键特性

- `Build` 可以包括多个步骤，而每个步骤指定一个 `Builder`.  
- `Builder` 是一种容器镜像，可以创建该镜像来完成任何任务，无论是流程中的单个步骤，还是整个流程本身。
- `Build`中的步骤可以推送到仓库。
- `BuildTemplate`可用于定义可重用的模板。
- 可以定义`Build`中的`source`以装载数据到Kubernetes Volume，并支持：
  - `git` 仓库
  - Google Cloud Storage
  - 任意容器镜像
- 通过ServiceAccount来使用Kubernetes Secrets进行身份**验证**。

### 配置语法示例

使用以下示例来了解Knative Build的各种组件的语法和结构。请注意，以下示例中并不包含Build配置文件的所有元素，但您可以引用Knative Build samples部分以及上面的参考文件以获取更多信息。

以下示例演示了使用身份验证的构建，并包含多个步骤和多个仓库：

```yaml
apiVersion: build.knative.dev/v1alpha1
kind: Build
metadata:
  name: example-build
spec:
  serviceAccountName: build-auth-example
  source:
    git:
      url: https://github.com/example/build-example.git
      revision: master
  steps:
  - name: ubuntu-example
    image: ubuntu
    args: ["ubuntu-build-example", "SECRETS-example.md"]
  steps:
  - image: gcr.io/example-builders/build-example
    args: ['echo', 'hello-example', 'build']
```



> 备注：以上内容来自 https://github.com/knative/docs/tree/master/build 