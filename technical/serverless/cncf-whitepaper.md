# CNCF Serverless Whitepaper v1.0

备注：内容翻译自 https://github.com/cncf/wg-serverless/tree/master/whitepaper

本文描述了新型云原生计算模型，该模型由新兴的“serverless”架构及其支持平台实现。它定义了serverless计算的内容，重点介绍了serverless计算的用例和成功示例，并展示了serverless计算如何与其他云应用程序开发模型（如Infrastructure-as-a-Service (IaaS), Platform-as-a-Service (PaaS)，容器编排或Container-as-a-Service（CaaS）。

本文包含对通用serverless平台机制的逻辑描述，还有和这些平台相关的编程模型和消息格式，但它没有规定标准。它介绍了几种行业serverless平台及其功能，但不推荐特定实现。

最后，还有关于CNCF如何推进云原生计算的serverless计算的一系列建议。

**工作组主席/TOC发起人:** Ken Owens (Mastercard)

**作者** (按姓氏字母顺序排列):

Sarah Allen (Google), Chris Aniszczyk (CNCF), Chad Arimura (Oracle), Ben Browning (Red Hat), Lee Calcote (SolarWinds), Amir Chaudhry (Docker), Doug Davis (IBM), Louis Fourie (Huawei), Antonio Gulli (Google), Yaron Haviv (iguazio), Daniel Krook (IBM), Orit Nissan-Messing (iguazio), Chris Munns (AWS), Ken Owens (Mastercard), Mark Peek (VMWare), Cathy Zhang (Huawei)

**其他贡献者** (按姓氏字母顺序排列):

Kareem Amin (Clay Labs), Amir Chaudhry (Docker), Sarah Conway (Linux Foundation), Zach Corleissen (Linux Foundation), Alex Ellis (VMware), Brian Grant (Google), Lawrence Hecht (The New Stack), Lophy Liu, Diane Mueller (Red Hat), Bálint Pató, Peter Sbarski (A Cloud Guru), Peng Zhao (Hyper)

# Serverless 计算

## 什么是serverless计算?

Serverless计算是指构建和运行不需要服务器管理的应用程序的概念。它描述了一种更细粒度的部署模型，应用程序捆绑一个或多个function，上载到平台，然后执行，缩放和计费，以响应当前所需的确切需求。

Serverless计算并不意味着我们不再使用服务器来托管和运行代码;这也不意味着不再需要运维工程师。相反，它指的是serverless计算的消费者不再需要花费时间和资源来进行服务器配置，维护，更新，扩展和容量规划。相反，所有这些任务和功能都由serverless平台处理，并完全从开发人员和IT/运维团队中抽象出来。因此，开发人员专注于编写应用程序的业务逻辑。运维工程师能够将重点更多的放到关键业务任务上。

Serverless有两个主要角色：

- **Developer/开发人员**：为serverless平台编写代码并从中获益，serverless平台提供了这样的视角：没有服务器，而代码始终在运行。
- **Provider/供应商**：为外部或内部客户部署serverless平台。

运行serverless平台仍然需要服务器。供应商需要管理服务器（或虚拟机和容器）。即使在空闲时，供应商也会有一些运行平台的成本。自托管系统仍然可以被视为serverless：通常一个团队充当**供应商**，另一个团队充当**开发人员**。

Serverless计算平台可以提供以下中的一个或两个：

1. Functions-as-a-Service (FaaS)，通常提供事件驱动计算。开发人员使用由事件或HTTP请求触发的function来运行和管理应用程序代码。 开发人员将代码的小型单元部署到FaaS，这些代码根据需要作为离散动作执行，无需管理服务器或任何其他底层基础设施即可进行扩展。
2. Backend-as-a-Service (BaaS)，它是基于API的第三方服务，可替代应用程序中的核心功能子集。因为这些API是作为可以自动扩展和透明操作的服务而提供的，所以对于开发人员表现为是serverless。

Serverless产品或平台为开发人员带来以下好处：

1. **零服务器运维**：serverless通过消除维护服务器资源所涉及的开销，显着改变了运行软件应用程序的成本模型。

   - **无需配置，更新和管理服务器基础设施**。管理服务器，虚拟机和容器对于公司而言是一项重大费用，其中包括人员，工具，培训和时间。Serverless大大减少了这种费用。

   - **灵活的可扩展性**：serverless的FaaS或BaaS产品可以即时而精确地扩展，以处理每个单独的传入请求。对于开发人员来说，serverless平台没有“预先计划容量”的概念，也不需要配置“自动缩放”的触发器或规则。缩放可以在没有开发人员干预的情况下自动进行。完成请求处理后，serverless FaaS会自动收缩计算资源，因此不会有空闲容量。
2. **闲置时无计算成本**：从消费者的角度来看，serverless产品的最大好处之一是空闲容量不会产生任何成本。例如，serverless计算服务不对空闲虚拟机或容器收费; 换句话说，当代码没有运行或者没有进行有意义的工作时，不收费。 对于数据库，数据库引擎容量空闲等待请求时无需收费。当然，这不包括其他成本，例如有状态存储成本或添加的功能/功能/特性集。

## Serverless技术简史

虽然按需或“花多少用多少”模式的概念可追溯到2006年和一个名为Zimki的平台，但serverless一词的第一次使用是2012年来自Iron.io的IronWorker产品 ，一个基于容器的分布式按需工作平台。

从那以后，公共云和私有云都出现了更多serverless实现。首先是BaaS产品，例如2011年的Parse和2012年的Firebase（分别由Facebook和谷歌收购）。2014年11月，[AWS Lambda](https://aws.amazon.com/lambda/)推出，2016年初在Bluemix上宣布了[IBM OpenWhisk on Bluemix](https://www.ibm.com/cloud-computing/bluemix/openwhisk)（现在是IBM Cloud Functions，其核心开源项目成为[Apache OpenWhisk](http://openwhisk.incubator.apache.org/)），[Google Cloud Functions](https://cloud.google.com/functions/)和[Microsoft Azure Functions](https://azure.microsoft.com/services/functions/)。[华为Function Stage](http://www.huaweicloud.com/product/functionstage.html)于2017年推出。还有许多开源serverless框架。每个框架（公共和私有）都具有独特的语言运行时和服务集，用于处理事件和数据。

这只是几个例子; 有关更完整和最新的列表，请参阅 [Serverless Landscape](https://docs.google.com/spreadsheets/d/10rSQ8rMhYDgf_ib3n6kfzwEuoE88qr0amUPRxKbwVCk/edit#gid=0) 文档。[Detail View: Serverless Processing Model](https://github.com/cncf/wg-serverless/tree/master/whitepaper#heading=h.vli9umq7mfhe) 部分包含有关整个FaaS模型的更多详细信息。

## Serverless用例

虽然serverless计算广泛使用，但它仍然相对较新。通常，当工作负载为以下情况时，应将serverless方法视为首选：

- 异步，并发，易于并行化为独立的工作单元
- 不经常或有零星的需求，在扩展要求方面存在巨大的，不可预测的差异
- 无状态，短暂的，对瞬间冷启动时间没有重大需求
- 在业务需求变更方面具有高度动态性，需要加快开发人员的速度

例如，对于常见的基于HTTP的应用程序，在自动扩展和更细粒度的成本模型方面有明显的优势。也就是说，使用serverless平台可能会有一些权衡。 例如，如果function的实例数下降到零，则在一段不活动时间后function启动可能会导致性能下降。因此，选择是否采用serverless架构需要仔细查看计算模型的功能性和非功能性方面。

不适合IaaS，PaaS或CaaS解决方案的非HTTP中心和非弹性规模工作负载现在可以利用serverless架构的按需性质和高效成本模型。其中一些工作负载包括：

- 执行逻辑以响应数据库更改（insert, update, trigger, delete）
- 对IoT传感器输入消息（例如MQTT消息）执行分析
- 流处理（分析或修改动态数据）
- 管理单次提取，转换和加载的作业，这些作业需要在短时间内进行大量处理（ETL）
- 通过聊天机器人界面提供认知计算（异步，但有关联）
- 调度执行时间很短的任务，例如cron或批处理样式调用
- 服务于机器学习和AI模型（检索一个或多个数据元素，如表格，NLP或图像，并与预先学习的数据模型匹配，以识别文本，面孔，异常等）
- 持续集成pipeline，按需为构建作业提供资源，而不是保留构建从属主机池等待作业分派

本节介绍serverless架构优秀的现有和新兴工作负载和用例。它还包括从早期成功案例中提取的早期结果，模式和最佳实践的详细信息。

这些场景中的每一个都显示了serverless架构如何解决技术问题，即Iaas，PaaS或CaaS效率低下或无法实现。这些例子是：

- 在没有按需模型的情况下，有效解决了一个全新的问题
- 更有效地解决了传统的云问题（性能，成本）
- 显示“大”的维度，无论是处理的数据大小还是处理的请求
- 通过低错误率的自动缩放（向上和向下）来显示弹性
- 以前所未有的速度（从天到小时）为市场带来了解决方案

### 多媒体处理

一个常见的用例，也是最早具体化的用例之一，是响应新文件上传执行一些转换过程的函数。 例如，如果将图像上载到诸如Amazon S3的对象存储服务，则该事件触发用于创建图像的缩略图版本并将其存储回另一个对象存储桶或数据库即服务的功能。 这是一个相当原子化，可并行化的计算任务的示例，该计算任务不经常运行并根据需求进行扩展。