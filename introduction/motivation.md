# 动机

备注：内容来自 https://github.com/knative/serving/blob/master/docs/spec/motivation.md

Knative Serving项目的目标是**为Serverless工作负载提供通用工具包和API框架**。

我们将Serverless工作负载定义为以下计算工作负载：

- 无状态
- 适用于流程横向扩展模型
- 主要由应用程序级别（例如L7-HTTP）请求流量驱动

虽然Kubernetes提供了支持此模型的基本原语（如Deployment和Service），但我们的经验表明，更紧凑和更丰富的自用模型对开发人员具有实质性的好处。特别是，通过标准化更高级的原语（这些原语执行大量公共基础架构自动化）应该可以构建一致的工具包，与使用kubectl更新yaml文件相比，它提供了更丰富的体验。

Knative Serving API由Compute API， [Build API](https://github.com/knative/build) 和 [Eventing API](https://github.com/knative/eventing) 组成。

