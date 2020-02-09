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