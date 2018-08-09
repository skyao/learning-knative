Knative介绍
================

Knative的定义：基于Kubernetes的平台，用于构建，部署和管理现代serverless工作负载。

>  Kubernetes-based platform to build, deploy, and manage modern serverless workloads.

Knative扩展了Kubernetes，提供了一组中间件组件，这些组件对于构建可在任何地方运行的现代，以源为中心和基于容器的应用程序至关重要：在本地，在云中，甚至在 第三方数据中心。

Knative项目下的每个组件都尝试识别常见模式并编纂最佳实践，这些最佳实践被真实世界中基于Kubernetes的成功框架和应用程序共享。 Knative组件专注于解决许多平凡但困难的任务，例如：

- 部署容器
- 在Kubernetes上编排source-to-URL的工作流程
- 使用蓝绿部署路由和管理流量
- 根据需求自动收缩和调整工作负载大小
- 将运行服务绑定到事件生态系统

Knative的开发人员可以使用熟悉的习语，语言和框架来部署任何工作负载：函数，应用程序或容器。

## 组件

目前有以下Knative组件：

- Build - 源到容器的构建编排
- Eventing - 管理和交付事件
- Serving - 请求驱动的计算，可以扩展到零

## 听众

Knative的设计考虑了不同的角色：

![](images/knative-audience.svg)

- 开发人员

  Knative组件为开发人员提供Kubernetes原生API，用于将serverless风格的函数，应用程序和容器部署到自动伸缩运行时。

- 运维

  Knative组件旨在集成到更加优雅的产品中，云服务提供商或大型企业的内部团队可以随后运维。

  任何企业或云提供商都可以将Knative组件应用到他们自己的系统中，并将这些收益传递给他们的客户。

- 贡献者

  凭借明确的项目范围，轻量级治理模型以及可插拔组件之间简洁的分离线，Knative项目建立了高效的贡献者工作流程。

  Knative是一个多元化，开放和包容的社区。

## knative特性

> 来自：https://cloud.google.com/knative/

- 所有人必备的基本原语

  Knative提供了一组中间件组件，这些组件对于构建可在任何地方运行的现代，以源为中心和基于容器的应用程序至关重要：在本地，在云中，甚至在第三方数据中心。 Knative组件基于Kubernetes构建，并编写最佳实践，这些最佳实践被真实世界中基于Kubernetes的成功框架和应用程序共享。它使开发人员能够专注于编写有趣的代码，而无需担心构建，部署和管理应用程序的“枯燥但困难”的部分。

- 开发人员友好的软件

​	Knative提供了一组可重用的组件，专注于解决许多平凡但困难的任务，例如编排源到容器工作流，在部署期间路由和管理流量，自动伸缩工作负载或将运行中的服务绑定到事件生态系统。 开发人员甚至可以使用熟悉的习语，语言和框架来部署任何工作负载：函数，应用程序或容器。

- 支持流行的开发模式

​	Knative专注于惯用的开发人员体验。 它支持常见的开发模式，如GitOps，DockerOps，ManualOps，以及Django，Ruby on Rails，Spring等工具和框架。

- 两全其美：灵活性和控制力

  Knative旨在轻松插入现有的构建和CI / CD工具链。 通过专注于开源优先技术，这些技术可以在任何地方，任何云上，在Kubernetes支持的任何基础架构上运行，企业可以随时随地移动工作负载。 这为客户根据自己独特的要求调整系统提供了所需的灵活性和控制。

- 运维友好

​	Knative旨在由所有主要云提供商作为服务运行。 Google目前与Pivotal，SAP，Red Hat，IBM等行业领导者合作，共同打造最适合开发人员需求的构建模块。 Knative为实际工作负载提供动力，并且与Kubernetes和Istio等其他尖端技术兼容。

- 在Kubernetes Engine上运行severless工作负载

​	您现在可以通过启用serverless加载项来在Google Kubernetes Engine（GKE）上运行serverless工作负载。serverless的附加组件由Knative提供支持，可帮助开发人员通过单击即可协调构建，服务和事件，从而通过GKE的灵活性和控制来实现惯用的开发人员体验。

