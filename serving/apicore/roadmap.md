# 2018 API Core 路线图

> 备注：
>
> - 内容来自 https://github.com/knative/serving/blob/master/pkg/apis/roadmap-2018.md
> - 看样子这个项目的确是在很早期的阶段，很多基本的东西都没有完成

API Core组的目的是为Knative Serving项目实现控制平面API。这包括API治理流程以及实施和支持文档。

这个路线图是我们希望在2018年完成的。

## References

- [Resource Overview](https://github.com/knative/serving/blob/master/docs/spec/overview.md)
- [Conformance Tests](https://github.com/knative/serving/blob/master/test/conformance/README.md)

在2018年，我们将主要关注策划和实施Knative Serving资源规范。

## 感兴趣的领域和需求

1. **Process**. 必须让贡献者清楚如何推动对Knative Serving API的更改。
2. **Schema**. [The Knative Serving API schema](https://github.com/knative/serving/blob/master/docs/spec/spec.md) 匹配 [我们的实现](https://github.com/knative/serving/blob/master/pkg/apis/serving).
3. **Semantics**. The  of Knative Serving API 交互的[语义](https://github.com/knative/serving/blob/master/pkg/controller) 符合 [我们的规范](https://github.com/knative/serving/blob/master/docs/spec/normative_examples.md), 并且 [一致性测试](https://github.com/knative/serving/blob/master/test/conformance/README.md)很好地涵盖了它们。

### Process

1. **定义流程** 用于提议，批准和实现API更改
2. **定义我们的约定**  在API更改时应该遵循，以便与Knative Serving API保持一致。

### 规范

1. 完成初始API规范的实现。
2. （根据我们的流程）跟踪我们API规范随时间的变化，包括API资源的版本控制。
3. 我们的实现从API规范中分类漂移(Triage drift)。

### 语义

1. 实现我们在 ["normative examples"](https://github.com/knative/serving/blob/master/docs/spec/normative_examples.md) 中概述的所需语义
2. 优雅而清晰地失败，如我们的["错误条件和报告"](https://github.com/knative/serving/blob/master/docs/spec/errors.md)文档中所述。

1. 通过确保我们的一致性测试完全涵盖语义，确保我们的实现与API规范的持续一致性。

1. 有关运维人员如何/应该如何定制Knative Serving安装（例如运行时契约）的指南，应该提供文档。