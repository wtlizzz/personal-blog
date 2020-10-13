title: Kubernetes入门
author: Wtli
tags:
  - Kubernetes
categories:
  - 后端
date: 2020-09-27 09:15:00
---
本文主要对K8S的基本信息进行学习。
<!--more-->


### Kubernetes

Kubernetes 是一个可移植的、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。Kubernetes 拥有一个庞大且快速增长的生态系统。Kubernetes 的服务、支持和工具广泛可用。

### Kubernetes意义
让我们回顾一下为什么 Kubernetes 如此有用。
#### 服务器部署演变

![0FJYFK.png](https://s1.ax1x.com/2020/09/27/0FJYFK.png)


**传统部署时代：** 早期，组织在物理服务器上运行应用程序。无法为物理服务器中的应用程序定义资源边界，这会导致资源分配问题。例如，如果在物理服务器上运行多个应用程序，则可能会出现一个应用程序占用大部分资源的情况，结果可能导致其他应用程序的性能下降。一种解决方案是在不同的物理服务器上运行每个应用程序，但是由于资源利用不足而无法扩展，并且组织维护许多物理服务器的成本很高。

**虚拟化部署时代：** 作为解决方案，引入了虚拟化功能，它允许您在单个物理服务器的 CPU 上运行多个虚拟机（VM）。虚拟化功能允许应用程序在 VM 之间隔离，并提供安全级别，因为一个应用程序的信息不能被另一应用程序自由地访问。

因为虚拟化可以轻松地添加或更新应用程序、降低硬件成本等等，所以虚拟化可以更好地利用物理服务器中的资源，并可以实现更好的可伸缩性。

每个 VM 是一台完整的计算机，在虚拟化硬件之上运行所有组件，包括其自己的操作系统。

**容器部署时代：** 容器类似于 VM，但是它们具有轻量级的隔离属性，可以在应用程序之间共享操作系统（OS）。因此，容器被认为是轻量级的。容器与 VM 类似，具有自己的文件系统、CPU、内存、进程空间等。由于它们与基础架构分离，因此可以跨云和 OS 分发进行移植。

容器因具有许多优势而变得流行起来。下面列出了容器的一些好处：

- 敏捷应用程序的创建和部署：与使用 VM 镜像相比，提高了容器镜像创建的简便性和效率。
- 持续开发、集成和部署：通过快速简单的回滚(由于镜像不可变性)，提供可靠且频繁的容器镜像构建和部署。
- 关注开发与运维的分离：在构建/发布时而不是在部署时创建应用程序容器镜像，从而将应用程序与基础架构分离。
- 可观察性不仅可以显示操作系统级别的信息和指标，还可以显示应用程序的运行状况和其他指标信号。
- 跨开发、测试和生产的环境一致性：在便携式计算机上与在云中相同地运行。
- 云和操作系统分发的可移植性：可在 Ubuntu、RHEL、CoreOS、本地、Google Kubernetes Engine 和其他任何地方运行。
- 以应用程序为中心的管理：提高抽象级别，从在虚拟硬件上运行 OS 到使用逻辑资源在 OS 上运行应用程序。
- 松散耦合、分布式、弹性、解放的微服务：应用程序被分解成较小的独立部分，并且可以动态部署和管理 - 而不是在一台大型单机上整体运行。
- 资源隔离：可预测的应用程序性能。
- 资源利用：高效率和高密度。

### Kubernetes功能 

容器是打包和运行应用程序的好方式。在生产环境中，您需要管理运行应用程序的容器，并确保不会停机。例如，如果一个容器发生故障，则需要启动另一个容器。如果系统处理此行为，会不会更容易？

这就是Kubernetes的意义所在！Kubernetes为您提供了一个可弹性运行分布式系统的框架。Kubernetes 会满足您的扩展要求、故障转移、部署模式等。例如，Kubernetes 可以轻松管理系统的 Canary 部署。

Kubernetes 为您提供：

- 服务发现和负载均衡  
Kubernetes 可以使用 DNS 名称或自己的 IP 地址公开容器，如果到容器的流量很大，Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。
- 存储编排  
Kubernetes 允许您自动挂载您选择的存储系统，例如本地存储、公共云提供商等。
- 自动部署和回滚  
您可以使用 Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态更改为所需状态。例如，您可以自动化 Kubernetes 来为您的部署创建新容器，删除现有容器并将它们的所有资源用于新容器。
- 自动二进制打包  
Kubernetes 允许您指定每个容器所需 CPU 和内存（RAM）。当容器指定了资源请求时，Kubernetes 可以做出更好的决策来管理容器的资源。
- 自我修复  
Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的运行状况检查的容器，并且在准备好服务之前不将其通告给客户端。
- 密钥与配置管理  
Kubernetes 允许您存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。您可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。


### Kubernetes 不是什么

Kubernetes 不是传统的、包罗万象的 PaaS（平台即服务）系统。由于 Kubernetes 在容器级别而不是在硬件级别运行，因此它提供了 PaaS 产品共有的一些普遍适用的功能，例如部署、扩展、负载均衡、日志记录和监视。但是，Kubernetes 不是单一的，默认解决方案是可选和可插拔的。Kubernetes 提供了构建开发人员平台的基础，但是在重要的地方保留了用户的选择和灵活性。

Kubernetes：

- Kubernetes 不限制支持的应用程序类型。Kubernetes 旨在支持极其多种多样的工作负载，包括无状态、有状态和数据处理工作负载。如果应用程序可以在容器中运行，那么它应该可以在 Kubernetes 上很好地运行。
- Kubernetes 不部署源代码，也不构建您的应用程序。持续集成(CI)、交付和部署（CI/CD）工作流取决于组织的文化和偏好以及技术要求。
- Kubernetes 不提供应用程序级别的服务作为内置服务，例如中间件（例如，消息中间件）、数据处理框架（例如，Spark）、数据库（例如，mysql）、缓存、集群存储系统（例如，Ceph）。这样的组件可以在 Kubernetes 上运行，并且/或者可以由运行在 Kubernetes 上的应用程序通过可移植机制（例如，开放服务代理）来访问。
- Kubernetes 不指定日志记录、监视或警报解决方案。它提供了一些集成作为概念证明，并提供了收集和导出指标的机制。
- Kubernetes 不提供或不要求配置语言/系统（例如 jsonnet），它提供了声明性 API，该声明性 API 可以由任意形式的声明性规范所构成。
- Kubernetes 不提供也不采用任何全面的机器配置、维护、管理或自我修复系统。
- 此外，Kubernetes 不仅仅是一个编排系统，实际上它消除了编排的需要。编排的技术定义是执行已定义的工作流程：首先执行 A，然后执行 B，再执行 C。相比之下，Kubernetes 包含一组独立的、可组合的控制过程，这些过程连续地将当前状态驱动到所提供的所需状态。从 A 到 C 的方式无关紧要，也不需要集中控制，这使得系统更易于使用且功能更强大、健壮、弹性和可扩展性。

### Kubernetes-Pod
**Pods：**Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。  
Pod （就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） 容器； 这些容器共享存储、网络、以及怎样运行这些容器的声明。 Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod 所建模的是特定于应用的“逻辑主机”，其中包含一个或多个应用容器， 这些容器是相对紧密的耦合在一起的。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于 在同一逻辑主机上运行的云应用。

#### 什么是 Pod？

Pod 的共享上下文包括一组 Linux 名字空间、控制组（cgroup）和可能一些其他的隔离 方面，即用来隔离 Docker 容器的技术。 在 Pod 的上下文中，每个独立的应用可能会进一步实施隔离。

就 Docker 概念的术语而言，Pod 类似于共享名字空间和文件系统卷的一组 Docker 容器。

#### 使用 Pod

通常你不需要直接创建 Pod，甚至单实例 Pod。 相反，你会使用诸如 Deployment 或 Job 这类工作负载资源 来创建 Pod。如果 Pod 需要跟踪状态， 可以考虑 StatefulSet 资源。

Kubernetes 集群中的 Pod 主要有两种用法：

- 运行单个容器的 Pod。"每个 Pod 一个容器"模型是最常见的 Kubernetes 用例； 在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。

- 运行多个协同工作的容器的 Pod。 Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。 这些位于同一位置的容器可能形成单个内聚的服务单元 —— 一个容器将文件从共享卷提供给公众， 而另一个单独的“挂斗”（sidecar）容器则刷新或更新这些文件。 Pod 将这些容器和存储资源打包为一个可管理的实体。

每个 Pod 都旨在运行给定应用程序的单个实例。如果希望横向扩展应用程序（例如，运行多个实例 以提供更多的资源），则应该使用多个 Pod，每个实例使用一个 Pod。 在 Kubernetes 中，这通常被称为 副本（Replication）。 通常使用一种工作负载资源及其控制器 来创建和管理一组 Pod 副本。

#### Pod 怎样管理多个容器

Pod 被设计成支持形成内聚服务单元的多个协作过程（形式为容器）。 Pod 中的容器被自动安排到集群中的同一物理机或虚拟机上，并可以一起进行调度。 容器之间可以共享资源和依赖、彼此通信、协调何时以及何种方式终止自身。

例如，你可能有一个容器，为共享卷中的文件提供 Web 服务器支持，以及一个单独的 “sidecar（挂斗）”容器负责从远端更新这些文件，如下图所示：

![0FUB38.png](https://s1.ax1x.com/2020/09/27/0FUB38.png)

有些 Pod 具有 Init 容器应用容器运行前必须先运行完成的一个或多个初始化容器。 和 应用容器。 Init 容器会在启动应用容器之前运行并完成。

Pod 天生地为其成员容器提供了两种共享资源：网络和 存储。

#### 使用 Pod

你很少在 Kubernetes 中直接创建一个个的 Pod，甚至是单实例（Singleton）的 Pod。 这是因为 Pod 被设计成了相对临时性的、用后即抛的一次性实体。 当 Pod 由你或者间接地由 控制器 创建时，它被调度在集群中的节点上运行。 Pod 会保持在该节点上运行，直到 Pod 结束执行、Pod 对象被删除、Pod 因资源不足而被 驱逐 或者节点失效为止。
```
Note： 重启 Pod 中的容器不应与重启 Pod 混淆。 Pod 不是进程，而是容器运行的环境。 在被删除之前，Pod 会一直存在。
```
当你为 Pod 对象创建清单时，要确保所指定的 Pod 名称是合法的 DNS 子域名。

#### Pod 和控制器

你可以使用工作负载资源来创建和管理多个 Pod。 资源的控制器能够处理副本的管理、上线，并在 Pod 失效时提供自愈能力。 例如，如果一个节点失败，控制器注意到该节点上的 Pod 已经停止工作， 就可以创建替换性的 Pod。调度器会将替身 Pod 调度到一个健康的节点执行。

#### Pod 模版
负载资源的控制器通常使用 Pod 模板（Pod Template） 来替你创建 Pod 并管理它们。  
Pod 模板是包含在工作负载对象中的规范，用来创建 Pod。  
工作负载的控制器会使用负载对象中的 PodTemplate 来生成实际的 Pod。 PodTemplate 是你用来运行应用时指定的负载资源的目标状态的一部分。  
下面的示例是一个简单的 Job 的清单，其中的 template 指示启动一个容器。 该 Pod 中的容器会打印一条消息之后暂停。
```
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # 这里是 Pod 模版
    spec:
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # 以上为 Pod 模版
```

修改 Pod 模版或者切换到新的 Pod 模版都不会对已经存在的 Pod 起作用。 Pod 不会直接收到模版的更新。相反， 新的 Pod 会被创建出来，与更改后的 Pod 模版匹配。  
例如，Deployment 控制器针对每个 Deployment 对象确保运行中的 Pod 与当前的 Pod 模版匹配。如果模版被更新，则 Deployment 必须删除现有的 Pod，基于更新后的模版 创建新的 Pod。每个工作负载资源都实现了自己的规则，用来处理对 Pod 模版的更新。    
在节点上，kubelet并不直接监测 或管理与 Pod 模版相关的细节或模版的更新，这些细节都被抽象出来。 这种抽象和关注点分离简化了整个系统的语义，并且使得用户可以在不改变现有代码的 前提下就能扩展集群的行为。

#### Pod 联网

每个 Pod 都在每个地址族中获得一个唯一的 IP 地址。 Pod 中的每个容器共享网络名字空间，包括 IP 地址和网络端口。 Pod 内 的容器可以使用 localhost 互相通信。 当 Pod 中的容器与 Pod 之外 的实体通信时，它们必须协调如何使用共享的网络资源 （例如端口）。

在同一个 Pod 内，所有容器共享一个 IP 地址和端口空间，并且可以通过 localhost 发现对方。 他们也能通过如 SystemV 信号量或 POSIX 共享内存这类标准的进程间通信方式互相通信。 不同 Pod 中的容器的 IP 地址互不相同，没有 特殊配置 就不能使用 IPC 进行通信。 如果某容器希望与运行于其他 Pod 中的容器通信，可以通过 IP 联网的方式实现。

Pod 中的容器所看到的系统主机名与为 Pod 配置的 name 属性值相同。 网络部分提供了更多有关此内容的信息。

#### 容器的特权模式 

Pod 中的任何容器都可以使用容器规约中的 安全性上下文中的 privileged 参数启用特权模式。 这对于想要使用使用操作系统管理权能（Capabilities，如操纵网络堆栈和访问设备） 的容器很有用。 容器内的进程几乎可以获得与容器外的进程相同的特权。

<u>Note: 你的容器运行时必须支持 特权容器的概念才能使用这一配置。</u>

#### 静态 Pod

静态 Pod（Static Pod） 直接由特定节点上的 kubelet 守护进程管理， 不需要API 服务器看到它们。 尽管大多数 Pod 都是通过控制面（例如，Deployment） 来管理的，对于静态 Pod 而言，kubelet 直接监控每个 Pod，并在其失效时重启之。

静态 Pod 通常绑定到某个节点上的 kubelet。 其主要用途是运行自托管的控制面。 在自托管场景中，使用 kubelet 来管理各个独立的 控制面组件。

kubelet 自动尝试为每个静态 Pod 在 Kubernetes API 服务器上创建一个 镜像 Pod。 这意味着在节点上运行的 Pod 在 API 服务器上是可见的，但不可以通过 API 服务器来控制。


### Kubernetes常用概念

**集群：**集群由一组被称作节点的机器组成。这些节点上运行 Kubernetes 所管理的容器化应用。集群具有至少一个工作节点和至少一个主节点。  
工作节点托管作为应用程序组件的 Pod 。主节点管理集群中的工作节点和 Pod 。多个主节点用于为集群提供故障转移和高可用性。

这张图表展示了包含所有相互关联组件的 Kubernetes 集群。
![0FNFLd.png](https://s1.ax1x.com/2020/09/27/0FNFLd.png)

**控制平面组件：**控制平面的组件对集群做出全局决策(比如调度)，以及检测和响应集群事件（例如，当不满足部署的 replicas 字段时，启动新的 pod）。

**kube-apiserver：**主节点上负责提供 Kubernetes API 服务的组件；它是 Kubernetes 控制面的前端。

**etcd：**etcd 是兼具一致性和高可用性的键值数据库，可以作为保存 Kubernetes 所有集群数据的后台数据库。

**kube-scheduler：**主节点上的组件，该组件监视那些新创建的未指定运行节点的 Pod，并选择节点让 Pod 在上面运行。

**kube-controller-manager：**在主节点上运行控制器的组件。

从逻辑上讲，每个控制器都是一个单独的进程，但是为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。
这些控制器包括:
- 节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应。
- 副本控制器（Replication Controller）: 负责为系统中的每个副本控制器对象维护正确数量的 Pod。
- 端点控制器（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)。
- 服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌.

**cloud-controller-manager：**云控制器管理器是 1.8 的 alpha 特性。在未来发布的版本中，这是将 Kubernetes 与任何其他云集成的最佳方式。

cloud-controller-manager 进运行特定于云平台的控制回路。 如果你在自己的环境中运行 Kubernetes，或者在本地计算机中运行学习环境， 所部属的环境中不需要云控制器管理器。

与 kube-controller-manager 类似，cloud-controller-manager 将若干逻辑上独立的 控制回路组合到同一个可执行文件中，供你以同一进程的方式运行。 你可以对其执行水平扩容（运行不止一个副本）以提升性能或者增强容错能力。

下面的控制器都包含对云平台驱动的依赖：
- 节点控制器（Node Controller）: 用于在节点终止响应后检查云提供商以确定节点是否已被删除
- 路由控制器（Route Controller）: 用于在底层云基础架构中设置路由
- 服务控制器（Service Controller）: 用于创建、更新和删除云提供商负载均衡器

**Node 组件：**节点组件在每个节点上运行，维护运行的 Pod 并提供 Kubernetes 运行环境。


**kubelet：**一个在集群中每个节点上运行的代理。它保证容器都运行在 Pod 中。

kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。kubelet 不会管理不是由 Kubernetes 创建的容器。

**kube-proxy：**kube-proxy 是集群中每个节点上运行的网络代理,实现 Kubernetes Service 概念的一部分。

kube-proxy 维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。

如果操作系统提供了数据包过滤层并可用的话，kube-proxy会通过它来实现网络规则。否则，kube-proxy 仅转发流量本身。

**插件（Addons）：**插件使用 Kubernetes 资源（DaemonSet、 Deployment等）实现集群功能。 因为这些插件提供集群级别的功能，插件中命名空间域的资源属于 kube-system 命名空间。

**DNS：**尽管其他插件都并非严格意义上的必需组件，但几乎所有 Kubernetes 集群都应该 有集群 DNS， 因为很多示例都需要 DNS 服务。

集群 DNS 是一个 DNS 服务器，和环境中的其他 DNS 服务器一起工作，它为 Kubernetes 服务提供 DNS 记录。

Kubernetes 启动的容器自动将此 DNS 服务器包含在其 DNS 搜索列表中。

**Web 界面（仪表盘）：**Dashboard 是Kubernetes 集群的通用的、基于 Web 的用户界面。 它使用户可以管理集群中运行的应用程序以及集群本身并进行故障排除。

### Kubernetes对象

在 Kubernetes 系统中，Kubernetes 对象 是持久化的实体。 Kubernetes 使用这些实体去表示整个集群的状态。特别地，它们描述了如下信息：
- 哪些容器化应用在运行（以及在哪些节点上）
- 可以被应用使用的资源
- 关于应用运行时表现的策略，比如重启策略、升级策略，以及容错策略

Kubernetes 对象是 “目标性记录” —— 一旦创建对象，Kubernetes 系统将持续工作以确保对象存在。 通过创建对象，本质上是在告知 Kubernetes 系统，所需要的集群工作负载看起来是什么样子的， 这就是 Kubernetes 集群的 期望状态（Desired State）。

操作 Kubernetes 对象 —— 无论是创建、修改，或者删除 —— 需要使用 Kubernetes API。 比如，当使用 kubectl 命令行接口时，CLI 会执行必要的 Kubernetes API 调用， 也可以在程序中使用 客户端库直接调用 Kubernetes API。

#### 对象规约（Spec）与状态（Status）

几乎每个 Kubernetes 对象包含两个嵌套的对象字段，它们负责管理对象的配置： 对象 spec（规约） 和 对象 status（状态） 。 对于具有 spec 的对象，你必须在创建对象时设置其内容，描述你希望对象所具有的特征： 期望状态（Desired State） 。

status 描述了对象的 当前状态（Current State），它是由 Kubernetes 系统和组件 设置并更新的。在任何时刻，Kubernetes 控制平面 都一直积极地管理着对象的实际状态，以使之与期望状态相匹配。

例如，Kubernetes 中的 Deployment 对象能够表示运行在集群中的应用。 当创建 Deployment 时，可能需要设置 Deployment 的 spec，以指定该应用需要有 3 个副本运行。 Kubernetes 系统读取 Deployment 规约，并启动我们所期望的应用的 3 个实例 —— 更新状态以与规约相匹配。 如果这些实例中有的失败了（一种状态变更），Kubernetes 系统通过执行修正操作 来响应规约和状态间的不一致 —— 在这里意味着它会启动一个新的实例来替换。

#### 描述 Kubernetes 对象

创建 Kubernetes 对象时，必须提供对象的规约，用来描述该对象的期望状态， 以及关于对象的一些基本信息（例如名称）。 当使用 Kubernetes API 创建对象时（或者直接创建，或者基于kubectl）， API 请求必须在请求体中包含 JSON 格式的信息。 大多数情况下，需要在 .yaml 文件中为 kubectl 提供这些信息。 kubectl 在发起 API 请求时，将这些信息转换成 JSON 格式。

这里有一个 .yaml 示例文件，展示了 Kubernetes Deployment 的必需字段和对象规约：
```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```
使用类似于上面的 .yaml 文件来创建 Deployment的一种方式是使用 kubectl 命令行接口（CLI）中的 kubectl apply 命令， 将 .yaml 文件作为参数。下面是一个示例：
```
kubectl apply -f https://k8s.io/examples/application/deployment.yaml --record
```
输出类似如下这样：
```
deployment.apps/nginx-deployment created
```
#### 必需字段

在想要创建的 Kubernetes 对象对应的 .yaml 文件中，需要配置如下的字段：
- apiVersion - 创建该对象所使用的 Kubernetes API 的版本
- kind - 想要创建的对象的类别
- metadata - 帮助唯一性标识对象的一些数据，包括一个 name 字符串、UID 和可选的 namespace

### Kubernetes架构

#### Kubernetes节点

Kubernetes 通过将容器放入在节点（Node）上运行的 Pod 中来执行你的工作负载。 节点可以是一个虚拟机或者物理机器，取决于所在的集群配置。每个节点都包含用于运行 Pod 所需要的服务，这些服务由 控制面负责管理。

通常集群中会有若干个节点；而在一个学习用或者资源受限的环境中，你的集群中也可能 只有一个节点。

节点上的组件包括 kubelet、 container runtime以及 kube-proxy。

#### Kubernetes管理

向 API 服务器添加节点的方式主要有两种：
1. 节点上的 kubelet 向控制面执行自注册；
2. 手动添加一个 Node 对象。

在你创建了 Node 对象或者节点上的 kubelet 执行了自注册操作之后， 控制面会检查新的 Node 对象是否合法。例如，如果你使用下面的 JSON 对象来创建 Node 对象：
```
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

Kubernetes 会在内部创建一个 Node 对象作为节点的表示。Kubernetes 检查 kubelet 向 API 服务器注册节点时使用的 metadata.name 字段是否匹配。 如果节点是健康的（即所有必要的服务都在运行中），则该节点可以用来运行 Pod。 否则，直到该节点变为健康之前，所有的集群活动都会忽略该节点。

<u>Note：Kubernetes 会一直保存着非法节点对应的对象，并持续检查该节点是否已经 变得健康。必需删除该 Node 对象以停止健康检查操作。</u>

Node 对象的名称必须是合法的 DNS 子域名。

#### 节点自注册
当 kubelet 标志 --register-node 为 true（默认）时，它会尝试向 API 服务注册自己。 这是首选模式，被绝大多数发行版选用。

对于自注册模式，kubelet 使用下列参数启动：

- --kubeconfig - 用于向 API 服务器表明身份的凭据路径。
- --cloud-provider - 与某云驱动 进行通信以读取与自身相关的元数据的方式。
- --register-node - 自动向 API 服务注册。
- --register-with-taints - 使用所给的污点列表（逗号分隔的 \<key\>=\<value\>:\<effect\>）注册节点。 当 register-node 为 false 时无效。
- --node-ip - 节点 IP 地址。
- --node-labels - 在集群中注册节点时要添加的 标签。 （参见 NodeRestriction 准入控制插件所实施的标签限制）。
- --node-status-update-frequency - 指定 kubelet 向控制面发送状态的频率。
  
启用节点授权模式和 NodeRestriction 准入插件 时，仅授权 kubelet 创建或修改其自己的节点资源。

#### 手动节点管理

你可以使用 kubectl 来创建和修改 Node 对象。

如果你希望手动创建节点对象时，请设置 kubelet 标志 --register-node=false。

你可以修改 Node 对象（忽略 --register-node 设置）。 例如，修改节点上的标签或标记其为不可调度。

你可以结合使用节点上的标签和 Pod 上的选择算符来控制调度。 例如，你可以限制某 Pod 只能在符合要求的节点子集上运行。

如果标记节点为不可调度（unschedulable），将阻止新 Pod 调度到该节点之上，但不会 影响任何已经在其上的 Pod。 这是重启节点或者执行其他维护操作之前的一个有用的准备步骤。

要标记一个节点为不可调度，执行以下命令：
```
kubectl cordon $NODENAME
```

<u>Note：被 DaemonSet 控制器创建的 Pod 能够容忍节点的不可调度属性。 DaemonSet 通常提供节点本地的服务，即使节点上的负载应用已经被腾空，这些服务也仍需 运行在节点之上。
</u>

#### 节点状态 

一个节点的状态包含以下信息:
- 地址
- 状况
- 容量与可分配
- 信息

你可以使用 kubectl 来查看节点状态和其他细节信息：
```
kubectl describe node <节点名称>
```
##### 地址 
这些字段的用法取决于你的云服务商或者物理机配置。

- HostName：由节点的内核设置。可以通过 kubelet 的 --hostname-override 参数覆盖。
- ExternalIP：通常是节点的可外部路由（从集群外可访问）的 IP 地址。
- InternalIP：通常是节点的仅可在集群内部路由的 IP 地址。

##### 状况

conditions 字段描述了所有 Running 节点的状态。状况的示例包括：  
节点条件使用 JSON 对象表示。例如，下面的响应描述了一个健康的节点。
```
"conditions": [
  {
    "type": "Ready",
    "status": "True",
    "reason": "KubeletReady",
    "message": "kubelet is posting ready status",
    "lastHeartbeatTime": "2019-06-05T18:38:35Z",
    "lastTransitionTime": "2019-06-05T11:41:27Z"
  }
]
```
<u>说明： 如果使用命令行工具来打印已保护（Cordoned）节点的细节，其中的 Condition 字段可能 包括 SchedulingDisabled。SchedulingDisabled 不是 Kubernetes API 中定义的 Condition，被保护起来的节点在其规约中被标记为不可调度（Unschedulable）。</u>

如果 Ready 条件处于 Unknown 或者 False 状态的时间超过了 pod-eviction-timeout 值， （一个传递给 kube-controller-manager 的参数）， 节点上的所有 Pod 都会被节点控制器计划删除。默认的逐出超时时长为 5 分钟。 某些情况下，当节点不可达时，API 服务器不能和其上的 kubelet 通信。 删除 Pod 的决定不能传达给 kubelet，直到它重新建立和 API 服务器的连接为止。 与此同时，被计划删除的 Pod 可能会继续在游离的节点上运行。

节点控制器在确认 Pod 在集群中已经停止运行前，不会强制删除它们。 你可以看到这些可能在无法访问的节点上运行的 Pod 处于 Terminating 或者 Unknown 状态。 如果 kubernetes 不能基于下层基础设施推断出某节点是否已经永久离开了集群， 集群管理员可能需要手动删除该节点对象。 从 Kubernetes 删除节点对象将导致 API 服务器删除节点上所有运行的 Pod 对象并释放它们的名字。

节点生命周期控制器会自动创建代表状况的 污点。 当调度器将 Pod 指派给某节点时，会考虑节点上的污点。 Pod 则可以通过容忍度（Toleration）表达所能容忍的污点。

capacity 块中的字段标示节点拥有的资源总量。 allocatable 块指示节点上可供普通 Pod 消耗的资源量。

可以在学习如何在节点上预留计算资源 的时候了解有关容量和可分配资源的更多信息。

第二个是保持节点控制器内的节点列表与云服务商所提供的可用机器列表同步。 如果在云环境下运行，只要某节点不健康，节点控制器就会询问云服务是否节点的虚拟机仍可用。 如果不可用，节点控制器会将该节点从它的节点列表删除。

第三个是监控节点的健康情况。节点控制器负责在节点不可达 （即，节点控制器因为某些原因没有收到心跳，例如节点宕机）时， 将节点状态的 NodeReady 状况更新为 "Unknown"。 如果节点接下来持续处于不可达状态，节点控制器将逐出节点上的所有 Pod（使用体面终止）。 默认情况下 40 秒后开始报告 "Unknown"，在那之后 5 分钟开始逐出 Pod。 节点控制器每隔 --node-monitor-period 秒检查每个节点的状态。

kubelet 负责创建和更新 NodeStatus 和 Lease 对象。

当一个可用区域（Availability Zone）中的节点变为不健康时，节点的驱逐行为将发生改变。 节点控制器会同时检查可用区域中不健康（NodeReady 状况为 Unknown 或 False） 的节点的百分比。如果不健康节点的比例超过 --unhealthy-zone-threshold （默认为 0.55）， 驱逐速率将会降低：如果集群较小（意即小于等于 --large-cluster-size-threshold 个节点 - 默认为 50），驱逐操作将会停止，否则驱逐速率将降为每秒 --secondary-node-eviction-rate 个（默认为 0.01）。 在单个可用区域实施这些策略的原因是当一个可用区域可能从控制面脱离时其它可用区域 可能仍然保持连接。 如果你的集群没有跨越云服务商的多个可用区域，那（整个集群）就只有一个可用区域。

跨多个可用区域部署你的节点的一个关键原因是当某个可用区域整体出现故障时， 工作负载可以转移到健康的可用区域。 因此，如果一个可用区域中的所有节点都不健康时，节点控制器会以正常的速率 --node-eviction-rate 进行驱逐操作。 在所有的可用区域都不健康（也即集群中没有健康节点）的极端情况下， 节点控制器将假设控制面节点的连接出了某些问题， 它将停止所有驱逐动作直到一些连接恢复。

节点控制器还负责驱逐运行在拥有 NoExecute 污点的节点上的 Pod， 除非这些 Pod 能够容忍此污点。 节点控制器还负责根据节点故障（例如节点不可访问或没有就绪）为其添加 污点。 这意味着调度器不会将 Pod 调度到不健康的节点上。

<u>注意： kubectl cordon 会将节点标记为“不可调度（Unschedulable）”。 此操作的副作用是，服务控制器会将该节点从负载均衡器中之前的目标节点列表中移除， 从而使得来自负载均衡器的网络请求不会到达被保护起来的节点。</u>

Kubernetes 调度器保证节点上 有足够的资源供其上的所有 Pod 使用。它会检查节点上所有容器的请求的总和不会超过节点的容量。 总的请求包括由 kubelet 启动的所有容器，但不包括由容器运行时直接启动的容器， 也不包括不受 kubelet 控制的其他进程。

<u>说明： 如果要为非 Pod 进程显式保留资源。请参考 为系统守护进程预留资源。
</u>













