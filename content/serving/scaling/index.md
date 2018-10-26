---
date: 2018-10-24T14:35:00+08:00
title: Serving Scaling
weight: 330
description : "Knative Serving Scaling介绍"
---

Knative Serving Scaling又称为Knative Serving Autoscaler。

## 伸缩界限

> 内容来自：https://github.com/knative/serving/blob/master/docs/scaling/README.md

有些情况下，运维人员需要设置服务于其应用的pod数量的下限和上限（例如，避免冷启动，控制计算成本等）。

可以在`configuration.revisionTemplate` 或`revision` （传播到`kpa`对象）上使用以下注释来完成以下操作：

```yaml
    # 可选
    # 如果不指定，revision可以收缩到0
    autoscaling.knative.dev/minScale: "2"
    # 可选
    # 如果不指定, 则没有上限
    autoscaling.knative.dev/maxScale: "10"
```

可以在`kpa`对象上直接使用这些注解。