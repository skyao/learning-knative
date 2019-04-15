---
date: 2018-10-24T14:35:00+08:00
title: Serving概述
weight: 310
description : "Knative Serving概述"
---

Serving概述

## 介绍

定义为：Kubernetes-based, scale-to-zero, request-driven compute 

Knative Serving以Kubernetes和Istio为基础，支持serverless应用程序和功能的部署与服务。 Serving很容易上手和扩展以支持高级方案。

Knative Serving项目提供了中间件原语，可以：

- serverless容器的快速部署
- 自动扩容和缩容到零
- Istio组件的路由和网络编程
- 已部署代码和配置的时间点快照

## Serving资源

Knative Serving将一组对象定义为Kubernetes自定义资源定义（CRD）。这些对象用于定义和控制severless工作负载在群集上的行为方式：

- Service/服务：`service.serving.knative.dev` 资源自动管理工作负载的整个生命周期。它控制其他对象的创建，以确保您的应用程序具有route/路由，configuration/配置和每次服务更新的新revision/修订版本。可以将服务定义为始终将流量路由到最新版本或固定版本。
- Route/路由：`route.serving.knative.dev` 资源将网络端点映射到一个或多个revision/修订版本。可以通过多种方式管理流量，包括部分流量和命名路由。
- Configuration/配置：`configuration.serving.knative.dev` 资源维护部署所需的状态。它提供了代码和配置之间的清晰分离，并遵循Twelve-Factor 应用方法论。修改配置会创建新revision/修订版本。
- Revision/修订版本：`revision.serving.knative.dev` 资源是每次对工作负载进行代码和配置修改的时间点快照。Revision/修订版本是不可变对象，只要有用就可以一直保留。

![](images/object_model.png)

> 备注：以上内容来自 https://github.com/knative/docs/tree/master/serving