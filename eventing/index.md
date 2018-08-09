# eventing

eventing是Knative事件绑定和发送的开源规范和实现。

eventing是一个正在进行中的事件系统，旨在满足云原生开发的常见需求：

1. 服务在开发期间松散耦合并独立部署
2. 生产者可以在消费者收听之前生成事件，并且消费者可以表达对尚未生成的事件或事件类别的兴趣。
3. 可以连接服务以创建新的应用程序
   - 无需修改生产者或消费者，而且
   - 能够从特定的生产者中选择特定的事件子集。

上述问题与CloudEvents的设计目标是一致的，CloudEvents是由CNCF serverless工作组开发的跨服务互操作性的通用规范。

## 架构

为了细分问题，我们将服务器端组件拆分为三个抽象：

![](images/concepts.png)

### Bus/总线

总线通过NATS或Kafka等消息总线提供k8s原生抽象。在这个层面上，抽象基本上是发布-订阅; 事件发布到Channel，订阅将该Channel路由到感兴趣的各方。

- **Channel** is a network endpoint which receives (and optionally persists) events using a Bus-specific implementation.
- **Subscription** connects events received on a Channel to an interested `target`, represented as a DNS name. There may be multiple Subscriptions on a single channel.
- **Bus** defines the adaptation layers needed to implement channels and subscriptions using a specific persistence strategy (such as delivery of events to a Kafka topic).

- **Channel**是网络终端，它使用特定于总线的实现来接收（并可选地持久化）事件。
- **Subscription**将在channel上收到的事件连接到感兴趣的目标，表示为DNS名称。 单个频道上可能有多个订阅。
- 总线定义了使用特定持久性策略（例如将事件传递到Kafka主题）实现通道和订阅所需的适配层。





