# CNCF × Alibaba 云原生技术公开课

### 第一堂“云原生”课

* 云原生的定义

  云原生是一条最佳路径或者最佳实践。更详细的说，云原生为用户指定了一条低心智负担的、敏捷的、能够以可扩展、可复制的方式最大化地利用云的能力、发挥云的价值的最佳路径。

  因此，云原生其实是一套指导进行软件架构设计的思想。按照这样的思想而设计出来的软件：首先，天然就“生在云上，长在云上”；其次，能够最大化地发挥云的能力，使得我们开发的软件和“云”能够天然地集成在一起，发挥出“云”的最大价值。

  所以，云原生的最大价值和愿景，就是认为未来的软件，会从诞生起就生长在云上，并且遵循一种新的软件开发、发布和运维模式，从而使得软件能够最大化地发挥云的能力。

* 为什么容器技术具有革命性

  容器技术和集装箱技术的革命性非常类似，即：容器技术使得应用具有了一种“自包含”的定义方式。所以，这样的应用才能以敏捷的、以可扩展、可复制的方式发布在云上，发挥出云的能力。这也就是容器技术对云发挥出的革命性影响所在，所以说，容器技术正是云原生技术的核心底盘。

* 云原生的技术范畴

  * 云应用定义与开发流程

    这包括应用定义与镜像制作、配置 CI/CD、消息和 Streaming 以及数据库等。

  * 云应用的编排与管理流程

    包括了应用编排与调度、服务发现与治理、远程调用、API 网关以及 Service Mesh。

  * 监控与可观测性

    这部分所强调的是云上应用如何进行监控、日志收集、Tracing 以及在云上如何实现破坏性测试，也就是混沌工程的概念。

  * 云原生的底层技术

    比如容器运行时、云原生存储技术、云原生网络技术等。

  * 云原生工具集

    在前面的这些核心技术点之上，还有很多配套的生态或者周边的工具需要使用，比如流程自动化与配置管理、容器镜像仓库、云原生安全技术以及云端密码管理等。

  * Serverless

    Serverless 是一种 PaaS 的特殊形态，它定义了一种更为“极端抽象”的应用编写方式，包含了 FaaS 和 BaaS 这样的概念。而无论是 FaaS 还是 BaaS，其最为典型的特点就是按实际使用计费（Pay as you go），因此 Serverless 计费也是重要的知识和概念。

* 云原生思想的两个理论基础

  * 不可变基础设施

    这一点目前是通过容器镜像来实现的，其含义就是应用的基础设施应该是不可变的，是一个自包含、自描述可以完全在不同环境中迁移的东西。

  * 云应用编排理论

    当前的实现方式就是 Google 所提出来的“容器设计模式”。

* 基础设施向云演进的过程

  一旦应用部署完成之后，那么这套应用基础设施就不会再修改了。如果需要更新，那么需要更改公共镜像来构建新服务直接替换旧服务。而我们之所以能够实现直接替换，就是因为容器提供了自包含的环境（包含应用运行所需的所有依赖）。

* 基础设施向云演进的意义

  * 基础设施一致性和可靠性

    容器镜像、自包含、可漂移

  * 简单可预测的部署和运维

    自描述，自运维、流程自动化、容易水平扩展、可快速复制的管控系统与支撑组件

* 云原生关键技术点

  * 如何构建自包含、可定制的应用镜像
  * 能不能实现应用快速部署与隔离能力
  * 应用基础设施创建和销毁的自动化管理
  * 可复制的管控系统和支撑组件

* 总结

  “未来的软件一定是生长于云上的”这是云原生理念的最核心假设。而所谓“云原生”，实际上就是在定义一条能够让应用最大程度利用云的能力、发挥云的价值的最佳路径。在这条路径上，脱离了“应用”这个载体，“云原生”就无从谈起；容器技术，则是将这个理念落地、将软件交付的革命持续进行下去的重要手段之一。

### 容器基本概念

* 容器与镜像

  * 操作系统管理的进程

    * 进程可见、可相互通信

      高级权限的进程可以攻击其他进程。

    * 共享同一份文件系统

      这些进程可以对于已有的数据进行增删改查，具有高级权限的进程可能会将其他进程的数据删除掉，破坏掉其他进程的正常运行。此外，进程与进程之间的依赖可能会存在冲突。

    * 使用相同的系统资源

      因为这些进程使用的是同一个宿主机的资源，应用之间可能会存在资源抢占的问题，当一个应用需要消耗大量 CPU 和内存资源的时候，可能会破坏其他应用的运行，导致其他应用无法正常地提供服务。

  * 如何为进程提供一个独立的运行环境

    * 资源视图隔离 namespace
    * 独立的文件系统 chroot
    * 控制资源使用率 cgroup

  * 什么是容器

    容器是一个视图隔离、资源可限制、独立文件系统的进程集合。

  * 什么是镜像

    运行容器所需要的所有文件集合。

    通常情况下，我们会采用 Dockerfile 来构建镜像，这是因为 Dockerfile 提供了非常便利的语法糖，能够帮助我们很好地描述构建的每个步骤。当然，每个构建步骤都会对已有的文件系统进行操作，这样就会带来文件系统内容的变化，我们将这些变化称之为 changeset。当我们把构建步骤所产生的变化依次作用到一个空文件夹上，就能够得到一个完整的镜像。

    changeset 具有分层及复用的特点。

  * 如何构建镜像

    Dockerfile - FROM / WORKDIR / COPY / RUN / CMD

    docker build / docker push（docker registry）

  * 如何运行容器

    docker pull / docker images / docker run

* 容器生命周期

  * 单进程模型

    init 进程生命周期 = 容器生命周期

    运行期间可运行 exec 执行运维操作

  * 数据持久化

    独立于容器的生命周期

    数据卷 - docker volume vs bind

* 容器项目的架构

  * moby 容器引擎架构

    * moby daemon

      会对上提供有关于容器、镜像、网络以及 volume 的管理。

    * containerd

      容器运行时管理引擎，独立于 moby daemon ，可以对上提供容器、镜像的相关管理。

    * shim

      其类似于一个守护进程。

      containerd 需要管理容器生命周期，而容器可能是由不同的容器运行时所创建出来的（runC / kata / gVisor），因此需要提供一个灵活的插件化管理。而 shim 就是针对于不同的容器运行时所开发的，这样就能够从 containerd 中脱离出来，通过插件的形式进行管理。

      因为 shim 插件化的实现，使其能够被 containerd 动态接管。如果不具备这样的能力，当 moby daemon 或者 containerd daemon 意外退出的时候，容器就没人管理了，那么它也会随之消失、退出，这样就会影响到应用的运行。

      因为随时可能会对 moby 或者 containerd 进行升级，如果不提供 shim 机制，那么就无法做到原地升级，也无法做到不影响业务的升级，因此 shim 非常重要，它实现了动态接管的能力。

* 容器 vs VM

  VM 利用 Hypervisor 虚拟化技术来模拟 CPU、内存等硬件资源，这样就可以在宿主机上建立一个 Guest OS，这是常说的安装一个虚拟机。

  每一个 Guest OS 都有一个独立的内核，比如 Ubuntu、CentOS 甚至是 Windows 等，在这样的 Guest OS 之下，每个应用都是相互独立的，VM 可以提供一个更好的隔离效果。但这样的隔离效果需要付出一定的代价，因为需要把一部分的计算资源交给虚拟化，这样就很难充分利用现有的计算资源，并且每个 Guest OS 都需要占用大量的磁盘空间，比如 Windows 操作系统的安装需要 10~30G 的磁盘空间，Ubuntu 也需要 5~6G，同时这样的方式启动很慢。

  容器是针对于进程而言的，因此无需 Guest OS，只需要一个独立的文件系统提供其所需要文件集合即可。所有的文件隔离都是进程级别的，因此启动时间快于 VM，并且所需的磁盘空间也小于 VM。当然了，进程级别的隔离并没有想象中的那么好，隔离效果相比 VM 要差很多。

### Kubernetes 核心概念

* 什么是 Kubernetes

  Kubernetes 是一个自动化的容器编排平台，它负责应用的部署、应用的弹性以及应用的管理，这些都是基于容器的。

  核心功能：服务发现与负载均衡、容器自动装箱（scheduling）、存储编排、自动容器恢复、自动发布与回滚、配置与密文管理、批量执行、水平伸缩。

* Kubernetes 的架构

  * Master

    所有 UI、CLI 这些 user 侧的组件，只会和 Master 进行连接，把希望的状态或者想执行的命令下发给 Master，Master 会把这些命令或者状态下发给相应的节点，进行最终的执行。

    * API Server（可以水平扩展）

      Kubernetes 中所有的组件都会和 API Server 进行连接，组件与组件之间一般不进行独立的连接，都依赖于 API Server 进行消息的传送。

    * Controller（可以进行热备）

      它用来完成对集群状态的一些管理（如自动容器恢复、水平伸缩）。

    * Scheduler（可以进行热备）

      把一个用户提交的 container，依据它对 CPU、对 memory 请求大小，找一台合适的节点，进行放置。

    * etcd

      是一个分布式的存储系统，API Server 中所需要的这些原信息都被放置在 etcd 中，etcd 本身是一个高可用系统，通过 etcd 保证整个 Kubernetes 的 Master 组件的高可用性。

  * Node

    Kubernetes 的 Node 是真正运行业务负载的，每个业务负载会以 Pod 的形式运行。

    * Pod

      一个 Pod 中运行一个或者多个容器，真正去运行这些 Pod 的组件的是叫做 Kubelet。

    * Kubelet

      它通过 API Server 接收到所需要 Pod 运行的状态，然后提交到 Container Runtime 组件中。

    * Container Runtime

      在 OS 上创建容器所需要运行的环境，最终把容器或者 Pod 运行起来。

    * Storage Plugin

    * Network Plugin

    * Kube-proxy

      它是为了提供 Service network 来进行搭网组网的。

* Kubernetes 的核心概念与 API

  * 核心概念

    * Pod

      最小的调度以及资源单元

      由一个或者多个容器组成

      定义容器运行的方式（Command、环境变量）

      提供给容器共享的运行环境（网络、进程空间）

    * Volume

      声明在 Pod 中的容器可以访问的文件目录

      可以被挂载在 Pod 中一个或者多个容器的指定路径下

      支持多种后端存储的抽象（本地存储、分布式存储、云存储）

    * Deployment

      Deployment 是 Pod 更上层的一个抽象，一般用 Deployment 抽象做应用的真正管理，而 Pod 是组成 Deployment 最小的单元

      定义一组 Pod 的副本数目、版本等

      通过 Controller 维持 Pod 的数目（自动恢复失败的 Pod）

      通过控制器以指定的策略控制版本（滚动升级、重新生成、回滚）

    * Service

      Service 提供了一个或者多个 Pod 实例的稳定访问地址（ClusterIP、NodePort、LoadBalancer）

    * Namespace

      一个集群内部的逻辑隔离机制（鉴权、资源额度）

      每个资源都属于一个 Namespace，同一个 Namespace 中的资源命名唯一，不同 Namespace 中的资源可重名

  * API（HTTP + JSON）

    apiVersion / kind / metadata / spec

    labels - 资源集合的默认表达形式

* minikube

  minikube status / kubectl get nodes / kubectl get deployments

  kubectl get --watch deployments

  kubectl apply -f deployment.yaml / kubectl apply -f deployment-update.yaml / kubectl apply -f deployment-scale.yaml

  kubectl describe deployment nginx-deployment

  kubectl delete deployment nginx-deployment

### 理解 Pod 和容器设计模式

* 为什么需要 Pod

  容器里 PID=1 的进程就是应用本身（“单进程”模型）-> 管理容器 = 直接管理应用本身 -> 不可变基础设施

  类比：镜像 -> 软件安装包，容器 -> 进程，Pod -> 进程组，Kubernetes -> 操作系统

  Pod 是一个逻辑单位，多个容器的组合（共享某些资源），Pod 也是 Kubernetes 的原子调度单位（Task co-scheduling）

  亲密关系，两个应用需要运行在同一台宿主机上，调度

  超亲密关系，发生文件交换、通过 localhost 或者本地 Socket 进行通信、非常频繁的 RPC 调用、共享某些 Linux Namespace（如 Network Namespace），Pod

* Pod 的实现机制

  * Pod 要解决的问题

    如何让一个 Pod 里的多个容器之间最高效的共享某些资源和数据，因为容器之间原本是被 Linux Namespace 和 cgroups 隔开的。

  * 共享网络

    通过 Infra Container（非常小的镜像，汇编语言编写的，永远处于“暂停”状态）来共享同一个 Network Namespace，Pod 中其它容器通过 Join Namespace 的方式加入到 Infra Container 的 Network Namespace 中，它们看到的网络视图是完全一样的

    使用 localhost 进行通信

    一个 Pod 只有一个 IP 地址，就是这个 Pod 的 Network Namespace 对应的 IP 地址，也是这个 Infra Container 的 IP 地址

    整个 Pod 的生命周期与 Infra Container 的生命周期一致，而与容器 A 和 B 无关。这也是为什么在 Kubernetes 里面，允许单独更新 Pod 里的某一个镜像，即：做这个操作，整个 Pod 不会重建，也不会重启。

  * 共享存储

    ![Pod 共享存储](https://github.com/songor/cloud-native-learned/blob/master/images/Pod%20%E5%85%B1%E4%BA%AB%E5%AD%98%E5%82%A8.PNG)

    Pod level volume -> shared-data 对应在宿主机上的目录会被同时绑定挂载进容器中

* 容器设计模式（Sidecar）

  通过在 Pod 里定义专门容器（initContainers），来执行主业务容器需要的辅助工作

  将辅助功能同主业务容器解耦，实现独立发布和能力重用

  应用与日志收集、代理容器、适配器容器

### 应用编排与管理：核心原理

* 资源元信息

  * Kubernetes 资源对象

    Spec - 期望的状态

    Status - 观测到的状态

    Metadata - Labels / Annotations / OwnerReference

    * Labels

      标识型的 Key: Value 元数据

      用于筛选资源；唯一的组合资源的方法

      可以使用 selector 来查询，类似于 SQL 'select * where ...'（相等型 Selector =、集合型 Selector in / notin，!release）

      environment: production / release: stable

    * Annotations

      Key: Value

      存储资源的非标识性信息；扩展资源的 spec / status

      一般比 label 更大；可以包含特殊字符；可以结构化也可以非结构化

    * OwnerReference

      Owner 即集合类资源，如 Pod 的集合 replicaset / statefulset

      集合类资源的控制器创建了归属资源（Replicaset 控制器创建 Pod）

      方便反向查找创建资源的对象；方便进行级联删除

  * 实践

    kubectl get pods --show-labels / kubectl get pods nginx -o yaml | less

    kubectl label pods nginx env=test --overwrite / kubectl label pods nginx tie-

    kubectl get pods --show-labels -l env=dev,tie=front / kubectl get pods --show-labels -l 'env in (test, dev)'

    kubectl annotate pods nginx my-annotate='my annotate, contains special characters'

    kubectl get replicasets nginx-replicasets -o yaml | less / kubectl get pods nginx-replicasets-rhd68 -o yaml | less

* 控制器模式

  * 控制循环

    Controller（控制器）、System（被控制的系统）、Sensor（传感器）三个逻辑组件独立运行

    spec / status / diff、controller operation、system output -> 不断使系统（status）向终态（spec）趋近

  * 命令式 API

    命令没有响应 - 反复重试；需要记录当前的操作（复杂）

    多次重试 - 巡检做修正（额外工作、危险）

    多并发访问 - 需要加锁（复杂、低效）

  * 声明式 API

    天然地记录了状态

    幂等操作、可在任意时刻反复操作

    正常操作即巡检

    可合并多个变更

  * 控制器模式总结

    声明式 API + 控制器

    由声明式的 API 驱动 - k8s 资源对象

    由控制器异步地控制系统向终态驱动

    使系统的自动化和无人值守化成为可能

    便于扩展 - 自定义资源和控制器

### 应用编排与管理：Deployment

* Deployment 能帮助我们做什么事情（管理部署发布的控制器）

  定义一组 Pod 的期望数量，Controller 会维持 Pod 数量与期望数量一致

  配置 Pod 发布方式，Controller 会按照给定策略更新 Pod，保证更新过程中不可用的 Pod 数量在限定范围内

  如果发布有问题，支持“一键”回滚

* Deployment 用例

  * Deployment 语法

    ![Deployment 语法](https://github.com/songor/cloud-native-learned/blob/master/images/Deployment%20%E8%AF%AD%E6%B3%95.PNG)

  * 查看 Deployment 状态

    kubectl create -f nginx-deployment.yaml

    kubectl get deployment nginx-deployment

    DESIRED - 期望 pod 数量 replicas / CURRENT - 当前实际 pod 数量 / UP-TO-DATE - 到达期望版本 pod 数量 / AVAILABLE / AGE

    kubectl edit deployment nginx-deployment

  * 查看 Pod

    kubectl get pod -> ${deployment-name}-${template-hash}-${ramdom-suffix}

    kubectl get pod nginx-deployment-${template-hash}-${ramdom-suffix} -o yaml -> ownerReferences 是 ReplicaSet 而非 Deployment

    kubectl edit pod nginx-deployment-${template-hash}-${ramdom-suffix}

    kubectl get replicaset

  * 更新镜像

    kubectl set image deployment.v1.apps nginx-deployment nginx=nginx:1.9.1

    set image - 设置镜像

    deployment.v1.apps - 资源类型，也可以写为 deployment 或 deployment.apps

    nginx-deployment - 要更新的 Deployment 名字

    nginx= - 要更新的容器名字

    nginx:1.9.1 - 新的镜像

  * 快速回滚

    kubectl rollout undo deployment nginx-deployment - 回滚到 Deployment 上一个版本

    kubectl rollout undo deployment.v1.apps nginx-deployment --to-revision=2 - 回滚 Deployment 到某一个版本

    kubectl rollout history deployment.v1.apps nginx-deployment - 版本列表

  * DeploymentStatus

    ![DeploymentStatus](https://github.com/songor/cloud-native-learned/blob/master/images/DeploymentStatus.PNG)

* 架构设计

  * 管理模式

    Deployment 只负责管理不同版本的 ReplicaSet，由 ReplicaSet 管理 Pod 副本数

    每个 ReplicaSet 对应了 Deployment template 的一个版本

    一个 ReplicaSet 下的 Pod 是相同的版本

  * Deployment 控制器

  * ReplicaSet 控制器

  * spec 字段解析

    * MinReadySeconds

      Deployment 判断 Pod 是否为 Available 的最小 Ready 时间

    * RevisionHistoryLimit

      保留历史 ReplicaSet 的数量，默认为 10

    * Paused

      标识 Deployment 只做数量维持，不做新的发布

    * ProgressDeadlineSeconds

      Deployment Status Processing -> Failed 最大时间 

  * 升级策略字段解析

    * MaxUnavailable

      滚动过程中最多有多少个 Pod 不可用

    * MaxSurge

      滚动过程中最多存在多少个 Pod 超过期望 replicas 数量

### 应用编排与管理：Job 和 DaemonSet

* Job 能帮助我们做什么事情（管理任务的控制器）

  创建一个或多个 Pod 确保指定数量的 Pod 可以成功地运行或终止

  跟踪 Pod 状态，根据配置及时重试失败的 Pod

  确定依赖关系，保证上一个任务运行完毕后再运行下一个任务

  控制任务并行度，并根据配置确保 Pod 队列大小

* Job 用例

  * Job 语法

    ![Job 语法](https://github.com/songor/cloud-native-learned/blob/master/images/Job%20%E8%AF%AD%E6%B3%95.PNG)

    restartPolicy - 重启策略

    backoffLimit - 重试次数限制

  * 查看 Job 状态

    kubectl create -f job.yaml

    kubectl get jobs

    COMPLETIONS - 完成 Pod 数量 / DURATION - Job 实际业务运行时长 / AGE

  * 查看 Pod

    kubectl get pod -> ${job-name}-${random-suffix}

    kubectl get pods pi-${random-suffix} -o yaml

    kubectl logs pods pi-${random-suffix}

  * 并行运行 Pod

    completions / parallelism

    watch "kubectl get pods"

  * CronJob 语法

    schedule - crontab 时间格式

    startingDeadlineSeconds - Job 最长启动时间

    concurrencyPolicy - 是否允许并行运行（是否等待上一个 Job 执行完成）

    successfulJobsHistoryLimit - 允许留存历史 Job 个数

    kubectl get cronjobs

* Job 架构设计

  * 管理模式

    Job Controller 负责根据配置创建 Pod

    Job Controller 跟踪 Job 状态，根据配置及时重试 Pod 或继续创建

    Job Controller 会自动添加 label 来跟踪对应的 Pod，并根据配置并行或串行创建 Pod

  * Job 控制器

* DaemonSet 能帮助我们做什么事情（守护进程控制器）

  保证集群内每一个（或者一些）节点都运行一组相同的 Pod

  跟踪集群节点状态，保证新加入的节点自动创建对应的 Pod

  跟踪集群节点状态，保证移除的节点删除对应的 Pod

  跟踪 Pod 状态，保证每个节点 Pod 处于运行状态

* DaemonSet 适用场景

  集群存储进程 - glusterd、ceph

  日志收集进程 - fluentd、logstash

  需要在每个节点运行的监控收集器

* DaemonSet 用例

  * DaemonSet 语法

    kind: DaemonSet

    kubectl create -f daemonset.yaml

  * 查看 DaemonSet 状态

    kubectl get ds

    NODE SELECTOR - 节点选择标签

    kubectl get pods

  * 更新 DaemonSet

    * 更新策略

      RollingUpdate - DaemonSet 默认更新策略，当更新 DaemonSet 模板后，老的 Pod 会被先删除，然后再去创建新的 Pod，可以配合健康检查做滚动更新

      OnDelete - 当 DaemonSet 模板更新后，只有手动地删除某一个对应的 Pod，此节点 Pod 才会被更新

    * 实践

      kubectl set image ds/fluentd-elasticsearch fluentd-elasticsearch=fluent/fluntd:v1.4

      kubectl rollout status ds/fluentd-elasticsearch

* DaemonSet 架构设计

  * 管理模式

    DaemonSet Controller 负责根据配置创建 Pod

    DaemonSet Controller 跟踪 Pod 状态，根据配置及时重试 Pod 或者继续创建

    DaemonSet Controller 会自动添加 affinity & label 来跟踪对应的 Pod，并根据配置在每个节点或者适合的部分节点创建 Pod

  * DaemonSet 控制器

### 应用配置管理

* Pod 配置管理

  不可变基础设施（容器）的可变配置 - ConfigMap

  敏感信息的存储和使用 - Secret

  集群中 Pod 自我的身份认证 - ServiceAccount

  容器运行资源的配置管理 - Sepc.Containers[].Resources.limits/requests

  容器的运行安全管控 - Spec.Containers[].SecurityContext

  容器启动前置条件校验 - Spec.InitContainers

* ConfigMap

  * 介绍

    主要管理容器运行所需的配置文件，环境变量，命令行参数等可变配置。用于解耦容器镜像和可变配置，从而保障工作负载（Pod）的可移植性

    ![ConfigMap](https://github.com/songor/cloud-native-learned/blob/master/images/ConfigMap.PNG)

  * 创建

    kubectl create configmap [NAME] [DATA]

    [DATA] - 指定文件或者目录；指定键值对

    kubectl create configmap kube-flannel-cfg --from-file=configure-pod-container/configmap/cni-conf.json -n kube-system -> 文件名为 Key，文件内容为 Value

    kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm -> 键值对

  * 使用

    环境变量 -> Pod.spec.containers[].env[].valueFrom.configMapKeyRef.name / key

    命令行参数 -> Pod.spec.containers[].command ${}

    挂载配置文件 -> Pod.spec.containers[].volumeMounts[].name / mountPath

  * 注意

    ConfigMap 文件大小限制 1MB（etcd 要求）

    Pod 只能引用相同 Namespace 中的 ConfigMap

    Pod 引用的 ConfigMap 不存在时，Pod 无法创建成功，即 Pod 创建前需要先创建好 ConfigMap

    ConfigMap 配置环境变量时，如果 ConfigMap 中的某些 key 被认为无效，该环境变量将不会注入容器，但是 Pod 可以正常创建

    只有通过 k8s api 创建的 Pod 才能使用 ConfigMap，其他方式创建的 Pod（如 manifest 创建的 static pod）不能使用 ConfigMap

* Secret

  * 介绍

    集群中用于存储密码，token 等敏感信息的资源对象，其中的敏感数据采用 Base64 编码保存

    ![Secret](https://github.com/songor/cloud-native-learned/blob/master/images/Secret.PNG)

  * 创建

    可以用户自己创建，也有系统自动创建（k8s 为每个 Namespace 的默认用户 default ServiceAccount 创建 Secret）

    kubectl create secret generic [NAME] [DATA] [TYPE]

    [DATA] - 指定文件、键值对

    [TYPE] - 默认为 Opaque

    kubectl create secret generic myregistrykey --from-file=.dockerconfigjson=/root/.docker/config.json --type=kubernetes.io/dockerconfigjson

    kubectl create secret generic prod-db-secret --from-literal=username=produser --from-literal=password=Y4nys7f11

  * 使用

    一般通过 volume 挂载到指定容器目录，供容器中业务使用；另外在需要访问私有镜像仓库时，也可以引用 Secret 来实现

    Pod.spec.volumes[].secret.secretName

    Pod.spec.containers[].volumeMounts[].name / mountPath

  * 使用私有镜像仓库

    Pod.spec.imagePullSecrets

    ServiceAccount.imagePullSecrets

  * 注意

    Secret 文件大小限制 1MB

    机密信息采用 Secret 存储仍需要慎重考虑；可以考虑 Kubernetes + Vault 来解决敏感信息的加密和权限管理

    不建议采取 list / watch 获取 Secret 信息，而推荐使用 get 获取需要的 Secret，从而减少更多 Secret 暴露的可能性

* ServiceAccount

  * 介绍

    主要用于解决 Pod 在集群中的身份认证问题；其中认证使用的授权信息，利用 Secret 进行管理

    ![ServiceAccount](https://github.com/songor/cloud-native-learned/blob/master/images/ServiceAccount.PNG)

  * Pod 里的应用访问它所属的 k8s 集群

    Pod 创建时 Admission Controller 会根据指定的 ServiceAccount（默认为 default）把对应的 Secret 挂载到容器中固定的目录下（/var/run/secrets/kubernetes.io/serviceaccount），k8s 自动实现

    当 Pod 访问集群时，可以利用 Secret 中的 token 文件来认证 Pod 身份（ca.crt 用于校验服务端）

    默认的 token 认证信息：

    Group - system:serviceaccounts:[namespace-name]

    User - system:serviceaccount:[namespace-name]:[pod-name]

    Pod 身份被认证合法后，其权限需要通过 RBAC 功能来配置

* Resource

  容器资源配置管理

  ![Resource](https://github.com/songor/cloud-native-learned/blob/master/images/Resource.PNG)

  * 支持资源类型

    CPU（单位 millicore，1 Core = 1000 millicore）、Memory、ephemeral storage（临时存储）、自定义资源

  * 配置方法

    资源配置分为 request / limit 两种类型

    * CPU

      spec.containers[].resources.limits.cpu

      spec.containers[].resources.requests.cpu

    * Memory

      spec.containers[].resources.limits.memory

      spec.containers[].resources.requests.memory

    * ephemeral storage

      spec.containers[].resources.limits.ephemeral-storage

      spec.containers[].resources.requests.ephemeral-storage

  * Pod 服务质量（QoS）配置

    当节点上资源不足时，将依据 BestEffort，Burstable，Guaranteed 的优先顺序驱逐 Pod

    * Guaranteed

      Pod 里的每个容器都必须有 CPU / 内存限制和请求，而且必须是一样的

    * Burstable

      非 Guaranteed

      Pod 里至少有一个容器有 CPU / 内存请求

    * BestEffort

      非 Guaranteed

      非 Burstable

* SecurityContext

  主要用于限制容器的行为，从而保障系统和其他容器的安全

  ![SecurityContext](https://github.com/songor/cloud-native-learned/blob/master/images/SecurityContext.PNG)

  * 分类

    容器级别的 SecurityContext，仅对指定容器有效

    Pod 级别的 SecurityContext，对指定 Pod 中的所有容器生效

    Pod Security Policies（PSP），对集群内所有 Pod 生效

  * 权限和访问控制设置项

    Discretionary Access Control / SELinux / privileged / Linux Capabilities / AppArmor / Seccomp / AllowPrivilegeEscalation

* InitContainer

  InitContainer 会先于普通 Container 启动执行，直到所有 InitContainer 执行成功后，普通 Container 才会被启动

  Pod 中多个 InitContainer 之间是按次序依次启动执行，而 Pod 中多个普通 Container 是并行启动

  InitContainer 执行成功后就结束退出了，而普通 Container 可能会一直执行或者重启

  一般 InitContainer 用于普通 Container 启动前的初始化（如配置文件准备）或前置条件检测（如网络连通检测）

### 应用存储和持久化数据卷：核心知识

* Kubernetes Volume 类型

  * 本地存储

    emptyDir / hostPath

  * 网络存储

    in-tree - awsElasticBlockStore / gcePersistentDisk / nfs

    out-of-tree - flexvolume / iscsi

  * Projected Volume

    Secret / ConfigMap / DownwardAPI / ServiceAccountToken

  * PVC 与 PV 体系

* Pod Volumes

  * 使用场景

    一个 Pod 中某一个容器异常退出，被 kubelet 拉起如何保证之前产生的重要数据不丢

    同一个 Pod 多个容器如何共享数据

  * 不足

    Pod 中声明的 Volume 的生命周期与 Pod 相同，无法准确表达 Volume 复用、共享语义，新功能扩展很难实现，无法满足场景：Pod 销毁重建 / 宿主机故障迁移 / 多 Pod 共享同一个 Volume / Volume Snapshot、Resize 等功能的扩展实现

  * 使用

    Pod.spec.volumes[] 声明 Pod 的 Volumes 信息

    Pod.spec.containers[].volumeMounts[] 声明 Container 如何使用 Pod 的 Volumes

    多个 Container 共享同一个 Volume 时，可以通过 Pod.spec.containers[].volumeMounts[].subPath 隔离不同容器在同一个 Volume 上数据存储的路径

    emptyDir - Pod 删除之后该目录也会被清除

    hostPath - 宿主机上路径，Pod 删除之后该目录仍然存在

* Persistent Volumes（PV）

  优化 Pod Volumes：将存储与计算分离，使用不同的组件管理存储与计算资源，解耦 Pod 与 Volume 的生命周期关联

  * Static Volume Provisioning

    Cluster Admin 需要提前规划或预测存储需求，而用户的需求是多样化的，很容易导致用户提交的 PVC 找不到合适的 PV

    * 使用

      系统管理员预先创建 PV

      PersisentVolume.spec.capacity.storage - 该 volume 的总容量大小

      PersisentVolume.spec.accessModes - PV 访问策略控制列表，必须同 PVC 的访问策略控制列表匹配才能绑定；ReadWriteOnce（只允许单个 Node 访问），ReadOnlyMany（允许多个 Node 只读访问），ReadWriteMany（允许多个 Node 读写访问）

      PersisentVolume.spec.persistentVolumeReclaimPolicy - PV 被 release 之后（与之绑定的 PVC 被删除）回收再利用策略；Delete（volume 被 release 之后直接 delete），Retain（默认策略，由系统管理员来手动管理该 volume）

      PersisentVolume.spec.csi.driver - 指定由什么 volume plugin 来挂载该 volume（需要提前在 node 上部署）

      用户创建 PVC

      PersistentVolumeClaim.spec.accessModes

      PersistentVolumeClaim.spec.resources.requests.storage

      用户创建 Pod

      Pod.spec.volumes[].persistentVolumeClaim.claimName

  * Dynamic Volume Provisioning

    Cluster Admin 只创建不同类型存储的模板（StorageClass），用户在 PVC 中指定使用哪种存储模板以及自己需要的大小、访问方式等参数，然后 k8s 自动生成相应的 PV 对象（k8s 结合 PVC 和 SC 两者的信息动态创建 PV 对象）
    
    * 使用
    
      系统管理员创建 StorageClass
    
      StorageClass.provisioner - 指定使用什么 volume plugin 来 create / delete / attach / detach / mount / unmount PV
    
      StorageClass.parameters
    
      StorageClass.reclaimPolicy
    
      用户创建 PVC
    
      PersistentVolumeClaim.spec.accessModes
    
      PersistentVolumeClaim.spec.resources.requests.storage
    
      PersistentVolumeClaim.spec.storageClassName
    
      用户创建 Pod
    
      Pod.spec.volumes[].persistentVolumeClaim.claimName
    
  * PV 状态流转

    create PV -> pending -> available -> bound -> released -> deleted / failed

    到达 released 状态的 PV 无法根据 Reclaim Policy 回到 available 状态而再次绑定新的 PVC，此时，如果想复用原来 PV 对应的存储中的数据，复用旧的 PV 中记录的存储信息新建 PV 对象 / 复用 PVC 对象，即不解绑 PVC 和 PV

* PersistentVolumeClaim（PVC）

  职责分离，PVC 中只用申明自己需要的存储 size，access mode 等业务真正关心的存储需求（不用关心存储实现细节），PV 和其对应的后端存储信息则交给 cluster admin 统一运维和管控，安全访问策略更容易控制

  PVC 简化了用户对存储的需求，PV 才是存储的实际信息的承载体，通过 kube-controller-manager 中的 PersistentVolumeController 将 PVC 与合适的 PV 绑定到一起，从而满足用户对存储的需求

  PVC 像是面向对象编程中抽象出来的接口，PV 是接口对应的实现

* 实践

  Static：

  kubectl create -f nas_pv.yaml / kubectl create -f nas_pvc.yaml / kubectl create -f dp_nas_demo.yaml

  kubectl get pv

  kubectl delete -f dp_nas_demo.yaml / kubectl delete -f nas_pvc.yaml / kubectl delete -f nas_pv.yaml

  Dynamic：

  kubectl create -f storageclass_disk.yaml / kubectl create -f disk_pvc.yaml

* k8s 对 PVC / PV 体系的完整处理流程

  create -> attach -> mount

### 应用存储和持久化数据卷：存储快照与拓扑调度

* 存储快照

  * 产生背景

    保证重要数据在误操作之后可以快速恢复，以提高操作容错率

    快速进行复制、迁移重要数据

  * snapshot

    PVC -> StorageClass -> PV

    VolumeSnapshot -> VolumeSnapshotClass -> VolumeSnapshotContent

    * 示例

      创建 VolumeSnapshotClass 对象：

      VolumeSnapshotClass.snapshotter: diskplugin.csi.alibabacloud.com - 指定 Volume Plugin

      创建 VolumeSnapshot 对象：

      VolumeSnapshot.spec.snapshotClassName

      VolumeSnapshot.spec.source.name - Snapshot 的数据源

      VolumeSnapshot.spec.source.kind: PersistentVolumeClaim

  * restore

    PersistentVolumeClaim 扩展字段 spec.dataSource 可以指定为 VolumeSnapshot 对象，从而根据 PVC 对象生成新的 PV 对象，对应的存储数据是从 VolumeSnapshot 关联的 VolumeSnapshotContent 恢复的

    * 示例

      PersistentVolumeClaim.spec.dataSource.name

      PersistentVolumeClaim.spec.dataSource.kind: VolumeSnapshot

* 拓扑的含义

  对 k8s 集群中 nodes 的“位置”关系一种人为划分规则，通过在 node 的 labels 中设置标识自己属于具体的拓扑域

  kubernetes.io/hostname=ip-172-20-114-199.ec2.internal - hostname

  failure-domain.beta.kubernetes.io/region=us-east-1 - region

  failure-domain.beta.kubernetes.io/zone=us-east-1c - zone

  也可以自定义一个 key: value pair 来定义一个具体的拓扑域

* 存储拓扑调度

  * 产生背景

    k8s 中通过 PVC&PC 体系将存储与计算分离，使用不同的 Controllers 管理存储资源和计算资源。但如果创建的 PV 有访问“位置”（spec.nodeAffinity）的限制，也就是只有在特定的一些 nodes 上才能访问 PV，原有的创建 Pod 的流程与创建 PV 的流程分离，就无法保证新创建的 Pod 被调度到可以访问 PV 的 node 上，最终导致 Pod 无法正常运行。如 Local PV 只能在指定的 Node 上被 Pod 使用；单 Region 多 Zone 的 k8s 集群，阿里巴巴云盘当前只能被同一个 Zone 的 Node 上的 Pod 访问

  * 本质问题

    Static / Dynamic Volume Provisioning PV 时，不知道使用它的 Pod 会被调度到哪些 Node 上，但 PV 本身的访问对 Node 的“位置”（拓扑）有限制

  * 流程改进

    Static / Dynamic Volume Provisioning PV 的操作延迟到 Pod 调度结果确定之后

  * k8s 相关组件改进

    PV Controller - 支持延迟绑定

    Dynamic PV Provisioner - 动态创建 PV 时要结合 Pod 待运行的 Node 的“位置”信息

    Scheduler - 选择 Node 时要考虑 Pod 的 PVC 绑定需求

  * 示例

    * Dynamic Provision PV 拓扑

      StorageClass.volumeBindingMode: WaitForFirstConsumer

      StorageClass.allowedTopologies

      当 PVC 对象被创建之后由于对应的 StorageClass 的 volumeBindingMode 为 WaitForFirstConsumer，并不会马上动态生成 PV 对象，而要等到使用该 PVC 对象的第一个 Pod 调度结果出来之后；并且 kube-scheduler 在调度 Pod 的时候会去选择满足 StorageClass.allowedTopologies 中指定的拓扑限制的 Nodes


### 可观测性：你的应用健康吗

* 如何保障应用健康稳定

  提高应用的可观测性（应用健康状态、应用资源使用、应用实时日志），提高应用的可恢复能力（应用出现问题需要降低影响范围，进行问题调试、诊断；可以通过自愈机制恢复）

* Liveness（存活探针） 与 Readness（就绪探针）

  * 探测方式

    HttpGet - 通过发送 HTTP Get 请求返回 200-399 状态码则表明容器健康

    Exec - 通过执行命令来检查服务是否正常，命令返回值为 0 则表示容器健康

    TcpSocket - 通过容器的 IP 和 Port 执行 TCP 检查，如果能够建立 TCP 连接则表明容器健康

  * 探测结果

    Success、Failure、Unknown

  * 重启策略

    Always、OnFailure、Never

  * Pod Probe Spec

    Pod.spec.containers.livenessProbe / readinessProbe.httpGet / exec / tcpSocket

    initialDelaySeconds - Pod 启动后延迟多久进行检查

    periodSeconds - 检查的间隔时间

    timeoutSeconds - 探测的超时时间

    successThreshold - 探测失败后再次判断成功的阈值

    failureThreshold - 探测失败的重试次数

  * Liveness

    用于判断容器是否存活，即 Pod 状态是否为 Running，如果探测结果不成功，则会触发 kubelet 杀掉容器，并根据配置的策略判断是否重启容器，如果默认不配置 Liveness 探针，则认为返回值默认为成功，适用于支持重新拉起的应用

  * Readness

    用于判断容器是否启动完成，即 Pod 状态是否为 Ready，如果探测结果不成功，则会将 Pod 从 Endpoint 中移除（切断上层流量到 Pod），直至下次判断成功，再将 Pod 挂回到 Endpoint 上，适用于启动后无法立即对外服务的应用

  * 合适的探测方式

    调大判断的超时阈值，防止在容器压力较高的情况下出现偶发超时

    调整判断的次数阈值，3 次的默认值在短周期下不一定是最佳实践

    Exec 执行的 Shell 脚本判断容器中可能调用时间会非常长

    TcpSocket 遇到 TLS 场景

* 状态机制 & 常见应用异常

  Pod 停留在 Pending - 调度器没有介入，可以通过 kubectl describe pod 查看事件进行排查，通常和资源使用相关

  Pod 停留在 Waiting - 一般表示 Pod 的镜像没有正常拉取

  Pod 不断被拉起且可以看到 Crashing - 一般表示 Pod 已经完成调度并启动，但是启动失败，通常由于配置、权限造成，需查看 Pod 日志

  Pod 处在 Running 但是没有正常工作 - 通常由于部分字段拼写错误造成的，可以通过校验部署来排查，例如 kubectl apply --validate -f pod.yaml

  Service 无法正常工作 - 在排除网络插件自身的问题外，最可能的是 label 配置有问题，可以通过查看 endpoint 的方式进行检查

*  应用远程调试

  * Pod 远程调试

    kubectl exec -it pod-name /bin/bash

    kubectl exec -it pod-name -c container-name /bin/bash

  * Service 远程调试

    使用 Telepresence 将本地的应用代理到集群中的一个 Service 上

    本地开发的应用需要调用集群中的服务 - kubectl port-forward svc/app -n app-namespace

  * kubectl-debug

### 可观测性：监控与日志

* 背景

  监控和日志是大型分布式系统的重要基础设施，监控可以帮助开发者查看系统的运行状态，而日志可以协助问题的排查和诊断。

  在 Kubernetes 中，监控和日志属于生态的一部分，并不是核心组件，因此大部分的能力依赖上层的云厂商的适配。Kubernetes 定义了接入的标准和规范，任何符合接口标准的组件都可以快速集成。

* 监控

  * 监控的类型

    资源监控 - CPU、内存、网络等资源类的指标

    性能监控 - 应用的内部监控，通常是通过 Hook 机制在虚拟机层、字节码层隐式回调，或者在应用层显式注入，获取更深层次的监控指标，常用来应用诊断与调优

    安全监控 - 越权管理、安全漏洞扫描等

    事件监控 - 紧密贴合 Kubernetes 的设计理念

  * Kubernetes 的监控接口标准

    注册了三种不同的 metrics 接口，将监控的消费能力进行标准化和解耦，从而实现了与社区的融合

    Resource Metrics - metrics.k8s.io

    Custom Metrics - custom.metrics.k8s.io

    External Metrics - external.metrics.k8s.io

  * Prometheus - 开源社区的监控“标准”

  * kube-eventer - Kubernetes 事件离线工具

* 日志

  * 场景

    主机内核的日志 - 协助开发者诊断如网络栈异常，驱动异常，文件系统异常，影响节点（内核）稳定的异常

    Runtime 的日志 - Docker 日志

    核心组件的日志 - APIServer 日志可以用来审计，Scheduler 日志可以诊断调度，etcd 日志可以查看存储状态，Ingress 日志可以分析接入层流量

    部署应用的日志 - 通过应用日志分析查看业务层的状态

  * 采集位置

    宿主机文件，容器内文件，容器标准、错误输出

  * Fluentd

### Kubernetes 网络概念及策略控制

* 基本法：约法三章 + 四大目标

  * Kubernetes 对于 Pod 间的网络没有任何限制，只需要满足如下“三个基本条件”

    所有 Pod 可以与其他 Pod 直接通信，无需显式使用 NAT

    所有 Node 可以与所有 Pod 直接通信，无需显式使用 NAT

    Pod 可见的 IP 地址为其他 Pod 与其通信时所用，无需显式转换

  * 基于以上准入条件，我们在审视一个网络方案的时候，需要考虑如下“四大目标”

    容器与容器间的通信

    Pod 与 Pod 之间的通信

    Pod 与 Service 间的通信

    外部世界与 Service 间的通信

*  Per Node Per IP

* Network Namespace

  Network Namespace 是实现网络虚拟化的内核基础，创建了隔离的网络空间：

  拥有独立的附属网络设备

  独立的协议栈，IP 地址和路由表

  iptables 规则

  ipvs 等

* Pod 与 Network Namespace 的关系

  每个 Pod 拥有独立的 Network Namespace 空间，Pod 内的 Container 共享该空间，可通过 Loopback 接口实现通信，或通过共享的 Pod IP 对外提供服务

* 典型容器网络实现方案

* Network Policy

  Network Policy 提供了基于策略的网络控制，用于隔离应用并减少攻击面。它使用标签选择器模拟传统的分段网络，并通过策略控制它们之间的流量以及来自外部的流量

  通过使用标签选择器（namespaceSelector，podSelector）来控制 Pod 之间的流量

### Kubernetes Service

* Kubernetes 中的服务发现与负载均衡

  * Kubernetes 应用间如何相互调用

    Pod 生命周期短暂，IP 地址随时变化

    Deployment 的 Pod 组需要统一访问入口和做负载均衡

    在不同环境部署时保持同样的部署拓扑和访问方式

  * 应用服务如何暴露到外部访问和负载均衡

* Service 语法

  Service.spec.selector.app - Pod 选择器

  Service.spec.ports[].protocol / port / targetPort

* 创建和查看 Service

  kubectl create -f server.yaml / kubectl get pod -o wide -l run=nginx

  kubectl create -f service.yaml / kubectl describe svc -> IP / Endpoints

* 集群内访问 Service

  * Service 的 ClusterIP

  * 访问服务名，依靠 DNS 解析

    {servicename}.{namespace}

  * 环境变量

    env

    $NGINX_SERVICE_HOST

* Headless Service

  Service.spec.clusterIP: None - 不再通过虚拟 IP 负载均衡

  通过 servicename 方式直接解析到后端所有的 Pod IP，客户端自主选择需要访问的 Pod

* 向集群外暴露 Service

  Service.spec.type: LoadBalancer

  LoadBalancer -> NodePort -> ClusterIP

  kubectl get svc -o wide -> EXTERNAL-IP