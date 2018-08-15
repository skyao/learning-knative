# Knative Build 资源

> 备注： 内容来自 https://github.com/knative/docs/blob/master/build/builds.md

使用Build资源对象创建并运行集群上的流程直至完成。

要在Knative中创建build，必须定义一个配置文件，在该文件中指定已经实现，并可以用来执行和完成任务的一个或多个容器镜像。

Build将一直运行，直到所有步骤完成或发生故障。

### 语法

要为Build资源定义配置文件，可以指定以下字段：

* 必须:
  * `apiVersion` - 指定API 版本, 例如 `build.knative.dev/v1alpha1`.
  * `kind` - 指定 `Build` 资源对象
  * `metadata` - 指定用于唯一标识Build资源对象的数据，例如name。
  * `spec`- 指定Build资源对象的配置信息。必须通过以下任一字段定义Build步骤：
    * `steps`- 指定要在build中运行的一个或多个容器镜像。
    * `template`- 指定包含一个或多个步骤的可重用build模板。
* 可选:
  * `source` - 指定为build提供信息的容器镜像。
  * `serviceAccountName` - 指定ServiceAccount资源对象，该对象使build能够使用定义的身份验证信息运行。
  * `volumes` - 指定一个或多个卷，让build可以使用。

以下示例是一个非工作示例，其中使用了大多数可能的配置字段：

```yaml
apiVersion: build.knative.dev/v1alpha1
kind: Build
metadata:
  name: example-build-name
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
  steps:
  - name: dockerfile-pushexample
    image: gcr.io/example-builders/push-example
    args: ["push", "${IMAGE}"]
    volumeMounts:
    - name: docker-socket-example
      mountPath: /var/run/docker.sock
  volumes:
  - name: example-volume
    emptyDir: {}
```

#### Steps

如果未定义template字段，则steps字段是必须的。可以定义一个或多个steps字段以定义build的主体。

构建中的每个`steps`都必须指定`Builder`或符合Knative build契约的容器镜像类型。 对于定义的每个`steps`字段或容器镜像：

* 从配置文件的顶部开始，Builder类型的容器镜像按顺序运行和评估，。
* 每个容器镜像运行直到完成或检测到第一个故障。

有关如何确保实现让每个step符合“builder契约”的详细信息，请参阅[`Builder`](./builder-contract.md)参考文档。


#### Template

如果未定义steps，则template字段是必需的。指定BuildTemplate资源对象，包括可重复或可共享的构建steps。

有关构建模板的示例和更多信息，请参阅[`BuildTemplate`](./build-templates.md)参考文档。


#### Source

可选。指定容器镜像。使用source字段为build提供构建所需的数据或上下文。数据将放置在已装入卷中的`/ workspace`目录中，并可用于build的所有步骤。

目前支持的source类型包括：

 * `git` - 基于Git的存储库。指定url字段以定义容器镜像的位置。指定`revision`以定义分支名称，tag名称，commit SHA或任何引用。
 * `gcs` - 位于Google云端存储中的存档。
 * `custom` - 任意容器镜像。


#### Service Account

可选。指定ServiceAccount资源对象的名称。使用serviceAccountName字段以指定服务帐户的权限运行build。如果未指定serviceAccountName字段，则build将使用Build资源对象的命名空间中的默认服务帐户运行。


#### Volumes

可选。指定一个或多个卷，为build所用，包括所有build steps。添加卷以补充在构建步骤中隐式创建的卷。

举例，使用卷来完成以下常见任务之一：

 * [装载 Kubernetes secret](./auth.md)
 * 创建一个`emptyDir`卷以充当缓存，以便在多个build steps中使用。考虑使用持久卷进行build间缓存。
 * 装载主机的 Docker socket 来为容器镜像build使用 `Dockerfile`


### 示例

使用这些代码段可以帮助您了解如何定义Knative build。

#### Using `git`

将git指定为source：

```yaml
spec:
  source:
    git:
      url: https://github.com/knative/build.git
      revision: master
  steps:
  - image: ubuntu
    args: ["cat", "README.md"]
```

#### Using `gcs`

将gcs指定为source：

```yaml
spec:
  source:
    gcs:
      type: Archive
      location: gs://build-crd-tests/rules_docker-master.zip
  steps:
  - name: list-files
    image: ubuntu:latest
    args: ["ls"]
```

#### Using `custom`

将custom指定为source：

```yaml
spec:
  source:
    custom:
      image: gcr.io/cloud-builders/gsutil
      args: ["rsync", "gs://some-bucket", "."]
  steps:
  - image: ubuntu
    args: ["cat", "README.md"]
```

#### Using an extra volume

装载多个卷：

```yaml
spec:
  steps:
  - image: ubuntu
    entrypoint: ["bash"]
    args: ["-c", "curl https://foo.com > /var/my-volume"]
    volumeMounts:
    - name: my-volume
      mountPath: /var/my-volume

  - image: ubuntu
    args: ["cat", "/etc/my-volume"]
    volumeMounts:
    - name: my-volume
      mountPath: /etc/my-volume

  volumes:
  - name: my-volume
    emptyDir: {}
```

#### Using `steps` to push images

定义steps来将容器镜像推送到仓库。

```yaml
spec:
  parameters:
  - name: IMAGE
    description: The name of the image to push
  - name: DOCKERFILE
    description: Path to the Dockerfile to build.
    default: /workspace/Dockerfile
  steps:
  - name: build-and-push
    image: gcr.io/kaniko-project/executor
    args:
    - --dockerfile=${DOCKERFILE}
    - --destination=${IMAGE}
```

#### 使用 `ServiceAccount`

指定ServiceAccount以访问私有git仓库：

```yaml
apiVersion: build.knative.dev/v1alpha1
kind: Build
metadata:
  name: test-build-with-serviceaccount-git-ssh
  labels:
    expect: succeeded
spec:
  serviceAccountName: test-build-robot-git-ssh
  source:
    git:
      url: git@github.com:knative/build.git
      revision: master

  steps:
  - name: config
    image: ubuntu
    command: ["/bin/bash"]
    args: ["-c", "cat README.md"]
```

其中`serviceAccountName：test-build-robot-git-ssh`引用以下ServiceAccount：

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: test-build-robot-git-ssh
secrets:
- name: test-git-ssh
```

而`name: test-git-ssh`，引用以下`Secret`：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-git-ssh
  annotations:
    build.knative.dev/git-0: github.com
type: kubernetes.io/ssh-auth
data:
  # Generated by:
  # cat id_rsa | base64 -w 1000000
  ssh-privatekey: LS0tLS1CRUdJTiBSU0EgUFJJVk.....[example]
  # Generated by:
  # ssh-keyscan github.com | base64 -w 100000
  known_hosts: Z2l0aHViLmNvbSBzc2g.....[example]
```

