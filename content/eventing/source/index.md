---
date: 2018-11-02T15:29:00+08:00
title: Eventing Source
description : "介绍 Knative 中的 Eventing Source"
---

Eventing Source

https://knative.dev/docs/eventing/sources/

事件源是 Kubernetes 自定义资源，它提供了一种机制用于注册特定软件系统对某一类事件的兴趣。由于不同的事件源可能由不同的自定义资源来描述，本页面提供了可用的源资源类型的索引以及安装说明的链接。

这是为Knative提供的非详尽的事件来源列表。

包含在这个列表中并不意味着认可，也不意味着任何程度的支持。

## Sources

这些都是作为CRD安装的来源。

| Name                                                         | Status           | Support | Description                                                  |
| ------------------------------------------------------------ | ---------------- | ------- | ------------------------------------------------------------ |
| [AWS SQS](https://github.com/knative/eventing-contrib/blob/master/awssqs/pkg/apis/sources/v1alpha1/aws_sqs_types.go) | 概念验证         | None    | Brings [AWS Simple Queue Service](https://aws.amazon.com/sqs/) messages into Knative. |
| [Apache Camel](https://github.com/knative/eventing-contrib/blob/master/camel/source/pkg/apis/sources/v1alpha1/camelsource_types.go) | 概念验证         | None    | Allows to use [Apache Camel](https://github.com/apache/camel) components for pushing events into Knative. |
| [Apache CouchDB](https://github.com/knative/eventing-contrib/tree/{{< branch >}}/couchdb) | 积极开发中       | None    | Brings [Apache CouchDB](https://couchdb.apache.org/) messages into Knative. |
| [Apache Kafka](https://github.com/knative/eventing-contrib/blob/master/kafka/source/pkg/apis/sources/v1alpha1/kafka_types.go) | 概念验证         | None    | Brings [Apache Kafka](https://kafka.apache.org/) messages into Knative. |
| [BitBucket](https://github.com/nachocano/bitbucket-source)   | 概念验证         | None    | Registers for events of the specified types on the specified BitBucket organization/repository. Brings those events into Knative. |
| [CloudAuditLogsSource](https://github.com/google/knative-gcp/blob/master/docs/examples/cloudauditlogssource/README.md) | 积极开发中       | None    | Registers for events of the specified types on the specified [Google Cloud Audit Logs](https://cloud.google.com/logging/docs/audit/). Brings those events into Knative. |
| [CloudPubSubSource](https://github.com/google/knative-gcp/blob/master/docs/examples/cloudpubsubsource/README.md) | 积极开发中       | None    | Brings [Cloud Pub/Sub](https://cloud.google.com/pubsub/) messages into Knative. |
| [CloudSchedulerSource](https://github.com/google/knative-gcp/blob/master/docs/examples/cloudschedulersource/README.md) | 积极开发中       | None    | Create, update, and delete [Google Cloud Scheduler](https://cloud.google.com/scheduler/) Jobs. When those jobs are triggered, receive the event inside Knative. |
| [CloudStorageSource](https://github.com/google/knative-gcp/blob/master/docs/examples/cloudstoragesource/README.md) | 积极开发中       | None    | Registers for events of the specified types on the specified [Google Cloud Storage](https://cloud.google.com/storage/) bucket and optional object prefix. Brings those events into Knative. |
| [~~Cron Job~~](https://knative.dev/docs/eventing/migration/ping) | 被PingSource替代 | None    | Deprecated, replace with [PingSource](https://knative.dev/docs/eventing/migration/ping) or a [CronJob using SinkBinding](https://knative.dev/docs/eventing/samples/sinkbinding/) |
| [GitHub](https://github.com/knative/eventing-contrib/blob/master/github/pkg/apis/sources/v1alpha1/githubsource_types.go) | 概念验证         | None    | Registers for events of the specified types on the specified GitHub organization/repository. Brings those events into Knative. |
| [GitLab](https://github.com/knative/eventing-contrib/blob/master/gitlab/pkg/apis/sources/v1alpha1/gitlabsource_types.go) | 概念验证         | None    | Registers for events of the specified types on the specified GitLab repository. Brings those events into Knative. |
| [Kubernetes](https://github.com/knative/eventing/blob/master/pkg/apis/sources/v1alpha1/apiserver_types.go) | 积极开发中       | Knative | 将Kubernetes API服务器事件引入Knative。                      |
| [Ping](https://github.com/knative/eventing/blob/master/pkg/apis/sources/v1alpha2/ping_types.go) | 发展中           | None    | 使用内存中的定时器，按照指定的cron计划产生具有固定有效载荷的事件。 |
| [VMware](https://github.com/vmware-tanzu/sources-for-knative/tree/{{< branch >}}/pkg/apis/source/v1alpha1/vspheresource_types.go) | 积极开发中       | None    | Brings [vSphere](https://www.vmware.com/products/vsphere.html) events into Knative. |

## Meta Sources

这些虽然不能直接使用，但可以让写Source变得更容易。

| Name                                                         | Status     | Support | Description                                                  |
| ------------------------------------------------------------ | ---------- | ------- | ------------------------------------------------------------ |
| [Auto Container Source](https://github.com/Harwayne/auto-container-source) | 概念验证   | None    | AutoContainerSource是一个控制器，它允许Source CRD不需要控制器。它注意到具有特定标签的CRD并开始控制该类型的资源。它利用Container Source作为底层基础设施。 |
| [Container Source](https://knative.dev/docs/eventing/migration/sinkbinding) | 积极开发中 | Knative | 给定一个至少指定了容器镜像的 spec.template，ContainerSource 将使用指定的镜像运行 Pod 。K_SINK(目标地址)和KE_CE_OVERRIDES(JSON CloudEvents属性)环境变量被注入到运行的镜像中。它被其他多个Sources用作底层基础设施。 |
| [Sample Source](https://github.com/knative/sample-source)    | 积极开发中 | Knative | Used as reference implementation supporting [Writing an Event Source from Scratch tutorial](https://knative.dev/docs/eventing/samples/writing-receive-adapter-source). |
| [SinkBinding](https://knative.dev/docs/eventing/samples/sinkbinding/) | 积极开发中 | Knative | SinkBinding提供了一个框架，用于将K_SINK（目标地址）和K_CE_OVERRIDES（JSON cloudevents属性）环境变量注入到任何Kubernetes资源中，这些资源的 spec.template 看起来像Pod（又名PodSpecable）。 |

### ContainerSource Containers

### Source Containers

These are containers intended to be used with `ContainerSource`. See the docs [here](https://knative.dev/docs/eventing/samples/container-source/index.html).

| Name                                                         | Status             | Support     | Description                                                  |
| ------------------------------------------------------------ | ------------------ | ----------- | ------------------------------------------------------------ |
| [AWS CodeCommit](https://github.com/triggermesh/aws-event-sources/blob/master/cmd/awscodecommitsource/README.md) | Supported          | TriggerMesh | Registers for events of the specified types on the specified AWS CodeCommit repository. Brings those events into Knative. |
| [AWS Cognito](https://github.com/triggermesh/aws-event-sources/blob/master/cmd/awscognitosource/README.md) | Supported          | TriggerMesh | Registers for AWS Cognito events. Brings those events into Knative. |
| [AWS DynamoDB](https://github.com/triggermesh/aws-event-sources/blob/master/cmd/awsdynamodbsource/README.md) | Supported          | TriggerMesh | Registers for events of on the specified AWS DynamoDB table. Brings those events into Knative. |
| [AWS Kinesis](https://github.com/triggermesh/aws-event-sources/tree/master/cmd/awskinesissource/README.md) | Supported          | TriggerMesh | Registers for events on the specified AWS Kinesis stream. Brings those events into Knative. |
| [AWS SNS](https://github.com/triggermesh/aws-event-sources/tree/master/cmd/awssnssource) | Supported          | TriggerMesh | Registers for events of the specified AWS SNS endpoint. Brings those events into Knative. |
| [AWS SQS](https://github.com/triggermesh/aws-event-sources/tree/master/cmd/awssqssource/README.md) | Supported          | TriggerMesh | Registers for events of the specified AWS SQS queue. Brings those events into Knative. |
| [FTP / SFTP](https://github.com/vaikas-google/ftp)           | Proof of concept   | None        | Watches for files being uploaded into a FTP/SFTP and generates events for those. |
| [Heartbeat](https://github.com/Harwayne/auto-container-source/tree/master/heartbeat-source) | Proof of Concept   | None        | Uses an in-memory timer to produce events as the specified interval. Uses AutoContainerSource for underlying infrastructure. |
| [Heartbeats](https://github.com/knative/eventing-contrib/tree/{{< branch >}}/cmd/heartbeats) | Proof of Concept   | None        | Uses an in-memory timer to produce events at the specified interval. |
| [K8s](https://github.com/Harwayne/auto-container-source/tree/master/k8s-event-source) | Proof of Concept   | None        | Brings Kubernetes cluster events into Knative. Uses AutoContainerSource for underlying infrastructure. |
| [WebSocket](https://github.com/knative/eventing-contrib/tree/{{< branch >}}/cmd/websocketsource) | Active Development | None        | Opens a WebSocket to the specified source and packages each received message as a Knative event. |

### SinkBindings

These are containers intended to be used with `SinkBinding`. See the docs [here](https://knative.dev/docs/eventing/samples/sinkbinding/index.html).

| Name                                       | Status             | Support | Description                                                  |
| ------------------------------------------ | ------------------ | ------- | ------------------------------------------------------------ |
| [Konnek](https://konnek.github.io/docs/#/) | Active Development | None    | Retrieves events from cloud platforms (like AWS and GCP) and transforms them into CloudEvents for consumption in Knative. |