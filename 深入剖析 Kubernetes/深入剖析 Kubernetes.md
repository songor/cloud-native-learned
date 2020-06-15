# 深入剖析 Kubernetes

### 开篇词 | 打通“容器技术”的任督二脉

### 01 | 预习篇 · 小鲸鱼大事记（一）：初出茅庐

* PaaS

  事实上，像 Cloud Foundry 这样的 PaaS 项目，最核心的组件就是一套应用的打包和分发机制。Cloud Foundry 为每种主流编程语言都定义了一种打包格式，而“cf push”的作用，基本上等同于用户把应用的可执行文件和启动脚本打进一个压缩包内，上传到云上 Cloud Foundry 的存储中。接着，Cloud Foundry 会通过调度器选择一个可以运行这个应用的虚拟机，然后通知这个机器上的 Agent 把应用压缩包下载下来启动。

  这时候关键来了，由于需要在一个虚拟机上启动很多个来自不同用户的应用，Cloud Foundry 会调用操作系统的 Cgroups 和 Namespace 机制为每一个应用单独创建一个称作“沙盒”的隔离环境，然后在“沙盒”中启动这些应用进程。这样，就实现了把多个用户的应用互不干涉地在虚拟机里批量地、自动地运行起来的目的。

  这，正是 PaaS 项目最核心的能力。而这些 Cloud Foundry 用来运行应用的隔离环境，或者说“沙盒”，就是所谓的“容器”。

* Docker 镜像

  而 Docker 镜像解决的，恰恰就是打包这个根本性的问题。所谓 Docker 镜像，其实就是一个压缩包。但是这个压缩包里的内容，比 PaaS 的应用可执行文件 + 启停脚本的组合就要丰富多了。实际上，大多数 Docker 镜像是直接由一个完整操作系统的所有文件和目录构成的，所以这个压缩包里的内容跟你本地开发和测试环境用的操作系统是完全一样的。

  更重要的是，这个压缩包包含了完整的操作系统文件和目录，也就是包含了这个应用运行所需要的所有依赖，所以你可以先用这个压缩包在本地进行开发和测试，完成之后，再把这个压缩包上传到云端运行。

  在这个过程中，你完全不需要进行任何配置或者修改，因为这个压缩包赋予了你一种极其宝贵的能力：本地环境和云端环境的高度一致！

* PaaS vs Docker

  Docker 项目给 PaaS 世界带来的“降维打击”，其实是提供了一种非常便利的打包机制。这种机制直接打包了应用运行所需要的整个操作系统，从而保证了本地环境和云端环境的高度一致，避免了用户通过“试错”来匹配两种不同运行环境之间差异的痛苦过程。

  不过，Docker 项目固然解决了应用打包的难题，但正如前面所介绍的那样，它并不能代替 PaaS 完成大规模部署应用的职责。

### 02 | 预习篇 · 小鲸鱼大事记（二）：崭露头角

### 03 | 预习篇 · 小鲸鱼大事记（三）：群雄并起

### 04 | 预习篇 · 小鲸鱼大事记（四）：尘埃落定

### 05 | 白话容器基础（一）：从进程说开去

* "边界"

  容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个“边界”。

  对于 Docker 等大多数 Linux 容器来说，Cgroups 技术是用来制造约束的主要手段，而 Namespace 技术则是用来修改进程视图的主要方法。

  ```sh
  docker run -it busybox /bin/sh
  
  / # ps
  PID   USER     TIME  COMMAND
      1 root      0:00 /bin/sh
      6 root      0:00 ps
  ```

  可以看到，我们在 Docker 里最开始执行的 /bin/sh，就是这个容器内部的第 1 号进程（PID=1），而这个容器里一共只有两个进程在运行。这就意味着，前面执行的 /bin/sh，以及我们刚刚执行的 ps，已经被 Docker 隔离在了一个跟宿主机完全不同的世界当中。

  这种技术，就是 Linux 里面的 Namespace 机制。而 Namespace 的使用方式也非常有意思：它其实只是 Linux 创建新进程的一个可选参数，比如：`int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); `。

  而除了我们刚刚用到的 PID Namespace，Linux 操作系统还提供了 Mount、UTS、IPC、Network 和 User 这些 Namespace，用来对各种不同的进程上下文进行“障眼法”操作。

  比如，Mount Namespace，用于让被隔离进程只看到当前 Namespace 里的挂载点信息；Network Namespace，用于让被隔离进程看到当前 Namespace 里的网络设备和配置。

  这，就是 Linux 容器最基本的实现原理了。

  所以，Docker 容器这个听起来玄而又玄的概念，实际上是在创建容器进程时，指定了这个进程所需要启用的一组 Namespace 参数。这样，容器就只能“看”到当前 Namespace 所限定的资源、文件、设备、状态，或者配置。而对于宿主机以及其他不相关的程序，它就完全看不到了。

  所以说，容器，其实是一种特殊的进程而已。

* 虚拟机 vs Docker

  ![虚拟机 vs Docker](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/%E8%99%9A%E6%8B%9F%E6%9C%BA%20vs%20Docker.jpg)

  这幅图的左边，画出了虚拟机的工作原理。其中，名为 Hypervisor 的软件是虚拟机最主要的部分。它通过硬件虚拟化功能，模拟出了运行一个操作系统需要的各种硬件，比如 CPU、内存、I/O 设备等等。然后，它在这些虚拟的硬件上安装了一个新的操作系统，即 Guest OS。

  这样，用户的应用进程就可以运行在这个虚拟的机器中，它能看到的自然也只有 Guest OS 的文件和目录，以及这个机器里的虚拟设备。这就是为什么虚拟机也能起到将不同的应用进程相互隔离的作用。

  在理解了 Namespace 的工作方式之后，你就会明白，跟真实存在的虚拟机不同，在使用 Docker 的时候，并没有一个真正的“Docker 容器”运行在宿主机里面。Docker 项目帮助用户启动的，还是原来的应用进程，只不过在创建这些进程时，Docker 为它们加上了各种各样的 Namespace 参数。

  这时，这些进程就会觉得自己是各自 PID Namespace 里的第 1 号进程，只能看到各自 Mount Namespace 里挂载的目录和文件，只能访问到各自 Network Namespace 里的网络设备，就仿佛运行在一个个“容器”里面，与世隔绝。

  不过，相信你此刻已经会心一笑：这些不过都是“障眼法”罢了。

### 06 | 白话容器基础（二）：隔离与限制

* Docker 项目比虚拟机更受欢迎的原因

  这是因为，使用虚拟化技术作为应用沙盒，就必须要由 Hypervisor 来负责创建虚拟机，这个虚拟机是真实存在的，并且它里面必须运行一个完整的 Guest OS 才能执行用户的应用进程。这就不可避免地带来了额外的资源消耗和占用。

  根据实验，一个运行着 CentOS 的 KVM 虚拟机启动后，在不做优化的情况下，虚拟机自己就需要占用 100~200 MB 内存。此外，用户应用运行在虚拟机里面，它对宿主机操作系统的调用就不可避免地要经过虚拟化软件的拦截和处理，这本身又是一层性能损耗，尤其对计算资源、网络和磁盘 I/O 的损耗非常大。

  而相比之下，容器化后的用户应用，却依然还是一个宿主机上的普通进程，这就意味着这些因为虚拟化而带来的性能损耗都是不存在的；而另一方面，使用 Namespace 作为隔离手段的容器并不需要单独的 Guest OS，这就使得容器额外的资源占用几乎可以忽略不计。

  所以说，“敏捷”和“高性能”是容器相较于虚拟机最大的优势，也是它能够在 PaaS 这种更细粒度的资源管理平台上大行其道的重要原因。

* 隔离得不彻底

  首先，既然容器只是运行在宿主机上的一种特殊的进程，那么多个容器之间使用的就还是同一个宿主机的操作系统内核。

  尽管你可以在容器里通过 Mount Namespace 单独挂载其他不同版本的操作系统文件，比如 CentOS 或者 Ubuntu，但这并不能改变共享宿主机内核的事实。这意味着，如果你要在 Windows 宿主机上运行 Linux 容器，或者在低版本的 Linux 宿主机上运行高版本的 Linux 容器，都是行不通的。

  其次，在 Linux 内核中，有很多资源和对象是不能被 Namespace 化的，最典型的例子就是：时间。

  这就意味着，如果你的容器中的程序使用 settimeofday(2) 系统调用修改了时间，整个宿主机的时间都会被随之修改，这显然不符合用户的预期。

* “限制”

  虽然容器内的第 1 号进程在“障眼法”的干扰下只能看到容器里的情况，但是宿主机上，它作为第 100 号进程与其他所有进程之间依然是平等的竞争关系。这就意味着，虽然第 100 号进程表面上被隔离了起来，但是它所能够使用到的资源（比如 CPU、内存），却是可以随时被宿主机上的其他进程（或者其他容器）占用的。当然，这个 100 号进程自己也可能把所有资源吃光。这些情况，显然都不是一个“沙盒”应该表现出来的合理行为。

  而 Linux Cgroups 就是 Linux 内核中用来为进程设置资源限制的一个重要功能。

  Linux Cgroups 的全称是 Linux Control Group。它最主要的作用，就是限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等。

  在 Linux 中，Cgroups 给用户暴露出来的操作接口是文件系统，即它以文件和目录的方式组织在操作系统的 /sys/fs/cgroup 路径下。

  `mount -t cgroup`

  可以看到，在 /sys/fs/cgroup 下面有很多诸如 cpuset、cpu、memory 这样的子目录，也叫子系统。这些都是我这台机器当前可以被 Cgroups 进行限制的资源种类。而在子系统对应的资源种类下，你就可以看到该类资源具体可以被限制的方法。

  `ls /sys/fs/cgroup/cpu`

  如果熟悉 Linux CPU 管理的话，你就会在它的输出里注意到 cfs_period 和 cfs_quota 这样的关键词。这两个参数需要组合使用，可以用来限制进程在长度为 cfs_period 的一段时间内，只能被分配到总量为 cfs_quota 的 CPU 时间。

  ```sh
  cd /sys/fs/cgroup/cpu
  mkdir container
  
  while : ; do : ; done &
  [1] 226
  
  cat /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us
  cat /sys/fs/cgroup/cpu/container/cpu.cfs_period_us
  echo 20000 > /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us
  echo 226 > /sys/fs/cgroup/cpu/container/tasks
  ```

  Linux Cgroups 的设计还是比较易用的，简单粗暴地理解呢，它就是一个子系统目录加上一组资源限制文件的组合。而对于 Docker 等 Linux 容器项目来说，它们只需要在每个子系统下面，为每个容器创建一个控制组（即创建一个新目录），然后在启动容器进程之后，把这个进程的 PID 填写到对应控制组的 tasks 文件中就可以了。

  ```sh
  docker run -it --cpu-period=100000 --cpu-quota=20000 ubuntu /bin/bash
  cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_period_us
  cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_quota_us
  ```

* 容器是一个“单进程”模型

  通过以上讲述，你现在应该能够理解，一个正在运行的 Docker 容器，其实就是一个启用了多个 Linux Namespace 的应用进程，而这个进程能够使用的资源量，则受 Cgroups 配置的限制。

  这也是容器技术中一个非常重要的概念，即：容器是一个“单进程”模型。

  由于一个容器的本质就是一个进程，用户的应用进程实际上就是容器里 PID=1 的进程，也是其他后续创建的所有进程的父进程。这就意味着，在一个容器中，你没办法同时运行两个不同的应用，除非你能事先找到一个公共的 PID=1 的程序来充当两个不同应用的父进程，这也是为什么很多人都会用 systemd 或者 supervisord 这样的软件来代替应用本身作为容器的启动进程。

  但是，在后面分享容器设计模式时，我还会推荐其他更好的解决办法。这是因为容器本身的设计，就是希望容器和应用能够同生命周期，这个概念对后续的容器编排非常重要。否则，一旦出现类似于“容器是正常运行的，但是里面的应用早已经挂了”的情况，编排系统处理起来就非常麻烦了。

### 07 | 白话容器基础（三）：深入理解容器镜像

* 容器里的进程看到的文件系统又是什么样子的呢？

  即使开启了 Mount Namespace，容器进程看到的文件系统也跟宿主机完全一样。

  仔细思考一下，你会发现这其实并不难理解：Mount Namespace 修改的，是容器进程对文件系统“挂载点”的认知。但是，这也就意味着，只有在“挂载”这个操作发生之后，进程的视图才会被改变。而在此之前，新创建的容器会直接继承宿主机的各个挂载点。

  这时，你可能已经想到了一个解决办法：创建新进程时，除了声明要启用 Mount Namespace 之外，我们还可以告诉容器进程，有哪些目录需要重新挂载。

  `mount("none", "/tmp", "tmpfs", 0, "");`

  更重要的是，因为我们创建的新进程启用了 Mount Namespace，所以这次重新挂载的操作，只在容器进程的 Mount Namespace 中有效。如果在宿主机上用 mount -l 来检查一下这个挂载，你会发现它是不存在的。

  这就是 Mount Namespace 跟其他 Namespace 的使用略有不同的地方：它对容器进程视图的改变，一定是伴随着挂载操作（mount）才能生效。

* 每当创建一个新容器时，我希望容器进程看到的文件系统就是一个独立的隔离环境

  不难想到，我们可以在容器进程启动之前重新挂载它的整个根目录“/”。而由于 Mount Namespace 的存在，这个挂载对宿主机不可见，所以容器进程就可以在里面随便折腾了。

  在 Linux 操作系统里，有一个名为 chroot 的命令可以帮助你在 shell 中方便地完成这个工作。顾名思义，它的作用就是帮你“change root file system”，即改变进程的根目录到你指定的位置。

  `chroot $HOME /bin/bash`

  实际上，Mount Namespace 正是基于对 chroot 的不断改良才被发明出来的，它也是 Linux 操作系统里的第一个 Namespace。

  当然，为了能够让容器的这个根目录看起来更“真实”，我们一般会在这个容器的根目录下挂载一个完整操作系统的文件系统，比如 Ubuntu16.04 的 ISO。这样，在容器启动之后，我们在容器里通过执行 "ls /" 查看根目录下的内容，就是 Ubuntu 16.04 的所有目录和文件。而这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“容器镜像”。它还有一个更为专业的名字，叫作：rootfs（根文件系统）。

  现在，你应该可以理解，对 Docker 项目来说，它最核心的原理实际上就是为待创建的用户进程：启用 Linux Namespace 配置；设置指定的 Cgroups 参数；切换进程的根目录（Change Root）。这样，一个完整的容器就诞生了。

  另外，需要明确的是，rootfs 只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像。

  实际上，同一台机器上的所有容器，都共享宿主机操作系统的内核。

  这就意味着，如果你的应用程序需要配置内核参数、加载额外的内核模块，以及跟内核进行直接的交互，你就需要注意了：这些操作和依赖的对象，都是宿主机操作系统的内核，它对于该机器上的所有容器来说是一个“全局变量”，牵一发而动全身。

  这也是容器相比于虚拟机的主要缺陷之一：毕竟后者不仅有模拟出来的硬件机器充当沙盒，而且每个沙盒里还运行着一个完整的 Guest OS 给应用随便折腾。

* 什么是容器的“一致性”呢？

  由于 rootfs 里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着，应用以及它运行所需要的所有依赖，都被封装在了一起。

  事实上，对于大多数开发者而言，他们对应用依赖的理解，一直局限在编程语言层面。比如 Golang 的 Godeps.json。但实际上，一个一直以来很容易被忽视的事实是，对一个应用来说，操作系统本身才是它运行所需要的最完整的“依赖库”。

  有了容器镜像“打包操作系统”的能力，这个最基础的依赖环境也终于变成了应用沙盒的一部分。这就赋予了容器所谓的一致性：无论在本地、云端，还是在一台任何地方的机器上，用户只需要解压打包好的容器镜像，那么这个应用运行所需要的完整的执行环境就被重现出来了。

  这种深入到操作系统级别的运行环境一致性，打通了应用在本地开发和远端执行环境之间难以逾越的鸿沟。

* 容器的 rootfs

  Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。

  当然，这个想法不是凭空臆造出来的，而是用到了一种叫作联合文件系统（Union File System）的能力。

  Union File System 也叫 UnionFS，最主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下。

  `docker image inspect ubuntu:latest`

  可以看到，这个 Ubuntu 镜像，实际上由五个层组成。这五个层就是五个增量 rootfs，每一层都是 Ubuntu 操作系统文件与目录的一部分；而在使用镜像时，Docker 会把这些增量联合挂载在一个统一的挂载点上。

  这个挂载点就是 /var/lib/docker/aufs/mnt/，不出意外的，这个目录里面正是一个完整的 Ubuntu 操作系统。

  `cat /proc/mounts | grep aufs`

  `cat /sys/fs/aufs/si_972c6d361e6b32ba/br[0-9]*`

  从这些信息里，我们可以看到，镜像的层都放置在 /var/lib/docker/aufs/diff 目录下，然后被联合挂载在 /var/lib/docker/aufs/mnt 里面。

* 镜像分层

  ![镜像分层](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/%E9%95%9C%E5%83%8F%E5%88%86%E5%B1%82.jpg)

  第一部分，只读层。

  它是这个容器的 rootfs 最下面的五层，对应的正是 ubuntu:latest 镜像的五层。可以看到，它们的挂载方式都是只读的（ro+wh，即 readonly+whiteout）。

  `ls /var/lib/docker/aufs/diff/<layer_id>`

  可以看到，这些层，都以增量的方式分别包含了 Ubuntu 操作系统的一部分。

  第二部分，可读写层。

  它是这个容器的 rootfs 最上面的一层，它的挂载方式为：rw，即 read write。在没有写入文件之前，这个目录是空的。而一旦在容器里做了写操作，你修改产生的内容就会以增量的方式出现在这个层中。

  可是，你有没有想到这样一个问题：如果我现在要做的，是删除只读层里的一个文件呢？

  为了实现这样的删除操作，AuFS 会在可读写层创建一个 whiteout 文件，把只读层里的文件“遮挡”起来。

  所以，最上面这个可读写层的作用，就是专门用来存放你修改 rootfs 后产生的增量，无论是增、删、改，都发生在这里。而当我们使用完了这个被修改过的容器之后，还可以使用 docker commit 和 push 指令，保存这个被修改过的可读写层，并上传到 Docker Hub 上，供其他人使用；而与此同时，原先的只读层里的内容则不会有任何变化。这，就是增量 rootfs 的好处。

  第三部分，Init 层。

  它是一个以“-init”结尾的层，夹在只读层和读写层之间。Init 层是 Docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息。

  需要这样一层的原因是，这些文件本来属于只读的 Ubuntu 镜像的一部分，但是用户往往需要在启动容器时写入一些指定的值比如 hostname，所以就需要在可读写层对它们进行修改。

  可是，这些修改往往只对当前的容器有效，我们并不希望执行 docker commit 时，把这些信息连同可读写层一起提交掉。

  所以，Docker 做法是，在修改了这些文件之后，以一个单独的层挂载了出来。而用户执行 docker commit 只会提交可读写层，所以是不包含这些内容的。

### 08 | 白话容器基础（四）：重新认识 Docker 容器

* 制作容器镜像

  Dockerfile 的设计思想，是使用一些标准的原语，描述我们所要构建的 Docker 镜像。并且这些原语，都是按顺序处理的。

  ```dockerfile
  # 使用官方提供的 Python 开发镜像作为基础镜像
  FROM python:2.7-slim
  
  # 将工作目录切换为 /app
  WORKDIR /app
  
  # 将当前目录下的所有内容复制到 /app 下
  ADD . /app
  
  # 使用 pip 命令安装这个应用所需要的依赖
  RUN pip install --trusted-host pypi.python.org -r requirements.txt
  
  # 允许外界访问容器的 80 端口
  EXPOSE 80
  
  # 设置环境变量
  ENV NAME World
  
  # 设置容器进程为：python app.py，即：这个 Python 应用的启动命令
  CMD ["python", "app.py"]
  ```

  其中，RUN 原语就是在容器里执行 shell 命令的意思。

  而 WORKDIR，意思是在这一句之后，Dockerfile 后面的操作都以这一句指定的 /app 目录作为当前目录。

  CMD \["python", "app.py"\] 等价于 "docker run \<image\> python app.py"。

  另外，在使用 Dockerfile 时，你可能还会看到一个叫作 ENTRYPOINT 的原语。实际上，它和 CMD 都是 Docker 容器进程启动所必需的参数，完整执行格式是：“ENTRYPOINT CMD”。

  但是，默认情况下，Docker 会为你提供一个隐含的 ENTRYPOINT，即：/bin/sh -c。

  所以，在不指定 ENTRYPOINT 时，比如在我们这个例子里，实际上运行在容器里的完整进程是：/bin/sh -c "python app.py"，即 CMD 的内容就是 ENTRYPOINT 的参数。

  需要注意的是，Dockerfile 里的原语并不都是指对容器内部的操作。就比如 ADD，它指的是把当前目录（即 Dockerfile 所在的目录）里的文件，复制到指定容器内的目录当中。

  `docker build -t helloworld .`

  而这个过程，实际上可以等同于 Docker 使用基础镜像启动了一个容器，然后在容器中依次执行 Dockerfile 中的原语。

  需要注意的是，Dockerfile 中的每个原语执行后，都会生成一个对应的镜像层。即使原语本身并没有明显地修改文件的操作（比如，ENV 原语），它对应的层也会存在。只不过在外界看来，这个层是空的。

* 启动容器

  `docker run -p 4000:80 helloworld`

  `curl http://localhost:4000`

* 上传镜像

  `docker tag helloworld <镜像仓库>/helloworld:v1`

  `docker push <镜像仓库>/helloworld:v1`

  此外，我还可以使用 docker commit 指令，把一个正在运行的容器，直接提交为一个镜像。一般来说，需要这么操作原因是：这个容器运行起来后，我又在里面做了一些操作，并且要把操作结果保存到镜像里。

* docker exec

  通过如下指令，你可以看到当前正在运行的 Docker 容器的进程号（PID）是 26113：

  `docker inspect --format '{{ .State.Pid }}' <CONTAINER ID>`

  这时，你可以通过查看宿主机的 proc 文件，看到这个 26113 进程的所有 Namespace 对应的文件：

  `ls -l /proc/26113/ns`

  可以看到，一个进程的每种 Linux Namespace，都在它对应的 /proc/\[进程号\]/ns 下有一个对应的虚拟文件，并且链接到一个真实的 Namespace 文件上。有了这样一个可以“hold 住”所有 Linux Namespace 的文件，我们就可以对 Namespace 做一些很有意义事情了，比如：加入到一个已经存在的 Namespace 当中。

  这也就意味着：一个进程，可以选择加入到某个进程已有的 Namespace 当中，从而达到“进入”这个进程所在容器的目的，这正是 docker exec 的实现原理。

  而这个操作所依赖的，乃是一个名叫 setns() 的 Linux 系统调用。

  `fd = open(argv[1], O_RDONLY);`

  `setns(fd, 0)`

  `execvp(argv[2], &argv[2]);`

  它一共接收两个参数，第一个参数是 argv\[1\]，即当前进程要加入的 Namespace 文件的路径，比如 /proc/26113/ns/net；而第二个参数，则是你要在这个 Namespace 里运行的进程，比如 /bin/bash。

  这段代码的核心操作，则是通过 open() 系统调用打开了指定的 Namespace 文件，并把这个文件的描述符 fd 交给 setns() 使用。在 setns() 执行后，当前进程就加入了这个文件对应的 Linux Namespace 当中了。

  此外，Docker 还专门提供了一个参数，可以让你启动一个容器并“加入”到另一个容器的 Network Namespace 里，这个参数就是 -net，比如：

  `docker run -it --net container:a0a9eb832c41 busybox ifconfig`

  这样，我们新启动的这个容器，就会直接加入到 ID=a0a9eb832c41 的容器，也就是我们前面的创建的 Python 应用容器（PID=26113）的 Network Namespace 中。

  而如果我指定 --net=host，就意味着这个容器不会为进程启用 Network Namespace。这就意味着，这个容器拆除了 Network Namespace 的“隔离墙”，所以，它会和宿主机上的其他普通进程一样，直接共享宿主机的网络栈。这就为容器直接操作和使用宿主机网络提供了一个渠道。

* Copy-on-Write

  由于使用了联合文件系统，你在容器里对镜像 rootfs 所做的任何修改，都会被操作系统先复制到这个可读写层，然后再修改。这就是所谓的：Copy-on-Write。

* Volume（数据卷）

  Volume 机制，允许你将宿主机上指定的目录或者文件，挂载到容器里面进行读取和修改操作。

  `docker run -v /test ...`

  由于你并没有显式声明宿主机目录，那么 Docker 就会默认在宿主机上创建一个临时目录 /var/lib/docker/volumes/\[VOLUME_ID\]/_data，然后把它挂载到容器的 /test 目录上。

  `docker run -v /home:/test ...`

  Docker 就直接把宿主机的 /home 目录挂载到容器的 /test 目录上。

  我们只需要在 rootfs 准备好之后，在执行 chroot 之前，把 Volume 指定的宿主机目录（比如 /home 目录），挂载到指定的容器目录（比如 /test 目录）在宿主机上对应的目录（即 /var/lib/docker/aufs/mnt/\[可读写层 ID\]/test）上，这个 Volume 的挂载工作就完成了。

  更重要的是，由于执行这个挂载操作时，“容器进程”已经创建了，也就意味着此时 Mount Namespace 已经开启了。所以，这个挂载事件只在这个容器里可见。你在宿主机上，是看不见容器内部的这个挂载点的。这就保证了容器的隔离性不会被 Volume 打破。

  注意：这里提到的"容器进程"，是 Docker 创建的一个容器初始化进程（dockerinit），而不是应用进程（ENTRYPOINT + CMD）。dockerinit 会负责完成根目录的准备、挂载设备和目录、配置 hostname 等一系列需要在容器内进行的初始化操作。最后，它通过 execv() 系统调用，让应用进程取代自己，成为容器里的 PID=1 的进程。

  而这里要使用到的挂载技术，就是 Linux 的绑定挂载（bind mount）机制。它的主要作用就是，允许你将一个目录或者文件，而不是整个设备，挂载到一个指定的目录上。

  其实，如果你了解 Linux 内核的话，就会明白，绑定挂载实际上是一个 inode 替换的过程。在 Linux 操作系统中，inode 可以理解为存放文件内容的“对象”，而 dentry，也叫目录项，就是访问这个 inode 所使用的“指针”。

  ![bind mount](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/bind%20mount.jpg)

  所以，在一个正确的时机，进行一次绑定挂载，Docker 就可以成功地将一个宿主机上的目录或文件，不动声色地挂载到容器中。这样，进程在容器里对这个 /test 目录进行的所有操作，都实际发生在宿主机的对应目录（比如，/home，或者 /var/lib/docker/volumes/\[VOLUME_ID\]/_data）里，而不会影响容器镜像的内容。

  容器的镜像操作，比如 docker commit，都是发生在宿主机空间的。而由于 Mount Namespace 的隔离作用，宿主机并不知道这个绑定挂载的存在。所以，在宿主机看来，容器中可读写层的 /test 目录（/var/lib/docker/aufs/mnt/\[可读写层 ID\]/test），始终是空的。

* “全景图”

  ![Docker 容器全景图](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/Docker%20%E5%AE%B9%E5%99%A8%E5%85%A8%E6%99%AF%E5%9B%BE.jpg)

### 09 | 从容器到容器云：谈谈 Kubernetes 的本质

* “容器”

  一个“容器”，实际上是一个由 Linux Namespace、Linux Cgroups 和 rootfs 三种技术构建出来的进程的隔离环境。

  从这个结构中我们不难看出，一个正在运行的 Linux 容器，其实可以被“一分为二”地看待：

  一组联合挂载在 /var/lib/docker/aufs/mnt 上的 rootfs，这一部分我们称为“容器镜像”（Container Image），是容器的静态视图；

  一个由 Namespace+Cgroups 构成的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器的动态视图。

* Kubernetes 架构

  ![Kubernetes 架构](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/Kubernetes%20%E6%9E%B6%E6%9E%84.jpg)

  我们可以看到，Kubernetes 项目的架构，跟它的原型项目 Borg 非常类似，都由 Master 和 Node 两种节点组成，而这两种角色分别对应着控制节点和计算节点。

  其中，控制节点，即 Master 节点，由三个紧密协作的独立组件组合而成，它们分别是负责 API 服务的 kube-apiserver，负责调度的 kube-scheduler，以及负责容器编排的 kube-controller-manager。整个集群的持久化数据，则由 kube-apiserver 处理后保存在 Etcd 中。

  而计算节点上最核心的部分，则是一个叫作 kubelet 的组件。

  在 Kubernetes 项目中，kubelet 主要负责同容器运行时（比如 Docker 项目）打交道。而这个交互所依赖的，是一个称作 CRI（Container Runtime Interface）的远程调用接口，这个接口定义了容器运行时的各项核心操作，比如：启动一个容器需要的所有参数。

  这也是为何，Kubernetes 项目并不关心你部署的是什么容器运行时、使用的什么技术实现，只要你的这个容器运行时能够运行标准的容器镜像，它就可以通过实现 CRI 接入到 Kubernetes 项目当中。

  而具体的容器运行时，比如 Docker 项目，则一般通过 OCI 这个容器运行时规范同底层的 Linux 操作系统进行交互，即：把 CRI 请求翻译成对 Linux 操作系统的调用（操作 Linux Namespace 和 Cgroups 等）。

  此外，kubelet 还通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互。这个插件，是 Kubernetes 项目用来管理 GPU 等宿主机物理设备的主要组件，也是基于 Kubernetes 项目进行机器学习训练、高性能作业支持等工作必须关注的功能。

  而 kubelet 的另一个重要功能，则是调用网络插件和存储插件为容器配置网络和持久化存储。这两个插件与 kubelet 进行交互的接口，分别是 CNI（Container Networking Interface）和 CSI（Container Storage Interface）。

* Kubernetes 项目要解决的问题是什么？

  从一开始，Kubernetes 项目就没有像同时期的各种“容器云”项目那样，把 Docker 作为整个架构的核心，而仅仅把它作为最底层的一个容器运行时实现。

  而 Kubernetes 项目要着重解决的问题，则来自于 Borg 的研究人员在论文中提到的一个非常重要的观点：运行在大规模集群中的各种任务之间，实际上存在着各种各样的关系。这些关系的处理，才是作业编排和管理系统最困难的地方。

  Kubernetes 项目最主要的设计思想是，从更宏观的角度，以统一的方式来定义任务之间的各种关系，并且为将来支持更多种类的关系留有余地。

  ![Kubernetes 核心功能全景图](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/Kubernetes%20%E6%A0%B8%E5%BF%83%E5%8A%9F%E8%83%BD%E5%85%A8%E6%99%AF%E5%9B%BE.jpg)

  在 Kubernetes 项目中，我们所推崇的使用方法是：首先，通过一个“编排对象”，比如 Pod、Job、CronJob 等，来描述你试图管理的应用；然后，再为它定义一些“服务对象”，比如 Service、Secret、Horizontal Pod Autoscaler（自动水平扩展器）等。这些对象，会负责具体的平台级功能。

  这种使用方法，就是所谓的“声明式 API”。这种 API 对应的“编排对象”和“服务对象”，都是 Kubernetes 项目中的 API 对象（API Object）。

* 调度 vs 编排

  实际上，过去很多的集群管理项目（比如 Yarn、Mesos，以及 Swarm）所擅长的，都是把一个容器，按照某种规则，放置在某个最佳节点上运行起来。这种功能，我们称为“调度”。

  而 Kubernetes 项目所擅长的，是按照用户的意愿和整个系统的规则，完全自动化地处理好容器之间的各种关系。这种功能，就是我们经常听到的一个概念：编排。

  所以说，Kubernetes 项目的本质，是为用户提供一个具有普遍意义的容器编排工具。

  不过，更重要的是，Kubernetes 项目为用户提供的不仅限于一个工具。它真正的价值，乃在于提供了一套基于容器构建分布式系统的基础依赖。

### 10 | Kubernetes 一键部署利器：kubeadm

* kubeadm 的工作原理

  把 kubelet 直接运行在宿主机上，然后使用容器部署其他的 Kubernetes 组件。

  所以，你使用 kubeadm 的第一步，是在机器上手动安装 kubeadm、kubelet 和 kubectl 这三个二进制文件。

  `apt-get install kubeadm`

  接下来，你就可以使用“kubeadm init”部署 Master 节点了。

* kubeadm init 的工作流程

  1. 当你执行 kubeadm init 指令后，kubeadm 首先要做的，是一系列的检查工作，以确定这台机器可以用来部署 Kubernetes。这一步检查，我们称为“Preflight Checks”，它可以为你省掉很多后续的麻烦。

  2. 在通过了 Preflight Checks 之后，kubeadm 要为你做的，是生成 Kubernetes 对外提供服务所需的各种证书和对应的目录。

     Kubernetes 对外提供服务时，除非专门开启“不安全模式”，否则都要通过 HTTPS 才能访问 kube-apiserver。这就需要为 Kubernetes 集群配置好证书文件。

     kubeadm 为 Kubernetes 项目生成的证书文件都放在 Master 节点的 /etc/kubernetes/pki 目录下。在这个目录下，最主要的证书文件是 ca.crt 和对应的私钥 ca.key。

     此外，用户使用 kubectl 获取容器日志等 streaming 操作时，需要通过 kube-apiserver 向 kubelet 发起请求，这个连接也必须是安全的。kubeadm 为这一步生成的是 apiserver-kubelet-client.crt 文件，对应的私钥是 apiserver-kubelet-client.key。

  3. 证书生成后，kubeadm 接下来会为其他组件生成访问 kube-apiserver 所需的配置文件。这些文件的路径是：/etc/kubernetes/xxx.conf。

     这些文件里面记录的是，当前这个 Master 节点的服务器地址、监听端口、证书目录等信息。这样，对应的客户端（比如 scheduler，kubelet 等），可以直接加载相应的文件，使用里面的信息与 kube-apiserver 建立安全连接。

  4. 接下来，kubeadm 会为 Master 组件生成 Pod 配置文件。Kubernetes 有三个 Master 组件 kube-apiserver、kube-controller-manager、kube-scheduler，而它们都会被使用 Pod 的方式部署起来。

  5. 在这一步完成后，kubeadm 还会再生成一个 Etcd 的 Pod YAML 文件，用来通过同样的 Static Pod 的方式启动 Etcd。

     Master 容器启动后，kubeadm 会通过检查 localhost:6443/healthz 这个 Master 组件的健康检查 URL，等待 Master 组件完全运行起来。

  6. 然后，kubeadm 就会为集群生成一个 bootstrap token。在后面，只要持有这个 token，任何一个安装了 kubelet 和 kubadm 的节点，都可以通过 kubeadm join 加入到这个集群当中。

  7. 在 token 生成之后，kubeadm 会将 ca.crt 等 Master 节点的重要信息，通过 ConfigMap 的方式保存在 Etcd 当中，供后续部署 Node 节点使用。这个 ConfigMap 的名字是 cluster-info。

  8. kubeadm init 的最后一步，就是安装默认插件。

     Kubernetes 默认 kube-proxy 和 DNS 这两个插件是必须安装的。它们分别用来提供整个集群的服务发现和 DNS 功能。其实，这两个插件也只是两个容器镜像而已，所以 kubeadm 只要用 Kubernetes 客户端创建两个 Pod 就可以了。

* Static Pod

  在 Kubernetes 中，有一种特殊的容器启动方法叫做“Static Pod”。它允许你把要部署的 Pod 的 YAML 文件放在一个指定的目录里。这样，当这台机器上的 kubelet 启动时，它会自动检查这个目录，加载所有的 Pod YAML 文件，然后在这台机器上启动它们。

  在 kubeadm 中，Master 组件的 YAML 文件会被生成在 /etc/kubernetes/manifests 路径下。

* kubeadm join 的工作流程

  任何一台机器想要成为 Kubernetes 集群中的一个节点，就必须在集群的 kube-apiserver 上注册。可是，要想跟 apiserver 打交道，这台机器就必须要获取到相应的证书文件（CA 文件）。可是，为了能够一键安装，我们就不能让用户去 Master 节点上手动拷贝这些文件。

  所以，kubeadm 至少需要发起一次“不安全模式”的访问到 kube-apiserver，从而拿到保存在 ConfigMap 中的 cluster-info（它保存了 APIServer 的授权信息）。而 bootstrap token，扮演的就是这个过程中的安全验证的角色。

  只要有了 cluster-info 里的 kube-apiserver 的地址、端口、证书，kubelet 就可以以“安全模式”连接到 apiserver 上，这样一个新的节点就部署完成了。

* 配置 kubeadm 的部署参数

  在这里，我强烈推荐你在使用 kubeadm init 部署 Master 节点时，使用下面这条指令：

  `kubeadm init --config kubeadm.yaml`

### 11 | 从 0 到 1：搭建一个完整的 Kubernetes 集群

* Taint/Toleration

  一旦某个节点被加上了一个 Taint，即被“打上了污点”，那么所有 Pod 就都不能在这个节点上运行，因为 Kubernetes 的 Pod 都有“洁癖”。

  除非，有个别的 Pod 声明自己能“容忍”这个“污点”，即声明了 Toleration，它才可以在这个节点上运行。

  `kubectl describe node master`

  可以看到，Master 节点默认被加上了 node-role.kubernetes.io/master:NoSchedule 这样一个“污点”，其中“键”是 node-role.kubernetes.io/master，而没有提供“值”。

  如果你就是想要一个单节点的 Kubernetes，删除这个 Taint 才是正确的选择：

  `kubectl taint nodes --all node-role.kubernetes.io/master-`

  如上所示，我们在 “node-role.kubernetes.io/master” 这个键后面加上了一个短横线 “-”，这个格式就意味着移除所有以 “node-role.kubernetes.io/master” 为键的 Taint。

* 无状态

  很多时候我们需要用数据卷（Volume）把外面宿主机上的目录或者文件挂载进容器的 Mount Namespace 中，从而达到容器和宿主机共享这些目录或者文件的目的。容器里的应用，也就可以在这些数据卷中新建和写入文件。

  可是，如果你在某一台机器上启动的一个容器，显然无法看到其他机器上的容器在它们的数据卷里写入的文件。这是容器最典型的特征之一：无状态。

* 持久化

  而容器的持久化存储，就是用来保存容器存储状态的重要手段：存储插件会在容器里挂载一个基于网络或者其他机制的远程数据卷，使得在容器里创建的文件，实际上是保存在远程存储服务器上，或者以分布式的方式保存在多个节点上，而与当前宿主机没有任何绑定关系。这样，无论你在其他哪个宿主机上启动新的容器，都可以请求挂载指定的持久化存储卷，从而访问到数据卷里保存的内容。这就是“持久化”的含义。

* Kubernetes 集群

  这个集群有一个 Master 节点和多个 Worker 节点；使用 Weave 作为容器网络插件；使用 Rook 作为容器持久化存储插件；使用 Dashboard 插件提供了可视化的 Web 界面。

### 12 | 牛刀小试：我的第一个容器化应用

* “控制器”模式

  像这样使用一种 API 对象（Deployment）管理另一种 API 对象（Pod）的方法，在 Kubernetes 中，叫作“控制器”模式（controller pattern）。

* Events

  在 Kubernetes 执行的过程中，对 API 对象的所有重要操作，都会被记录在这个对象的 Events 里，并且显示在 kubectl describe 指令返回的结果中。

  所以，这个部分正是我们将来进行 Debug 的重要依据。如果有异常发生，你一定要第一时间查看这些 Events，往往可以看到非常详细的错误信息。

* kubectl apply

  这样的操作方法，是 Kubernetes “声明式 API”所推荐的使用方法。也就是说，作为用户，你不必关心当前的操作是创建，还是更新，你执行的命令始终是 kubectl apply，而 Kubernetes 则会根据 YAML 文件的内容变化，自动进行具体的处理。

  而这个流程的好处是，它有助于帮助开发和运维人员，围绕着可以版本化管理的 YAML 文件，而不是“行踪不定”的命令行进行协作，从而大大降低开发人员和运维人员之间的沟通成本。

  所以说，如果通过容器镜像，我们能够保证应用本身在开发与部署环境里的一致性的话，那么现在，Kubernetes 项目通过这些 YAML 文件，就保证了应用的“部署参数”在开发与部署环境中的一致性。

  而当应用本身发生变化时，开发人员和运维人员可以依靠容器镜像来进行同步；当应用部署参数发生变化时，这些 YAML 文件就是他们相互沟通和信任的媒介。

* Label Selector

  顾名思义，Labels 就是一组 key-value 格式的标签。而像 Deployment 这样的控制器对象，就可以通过这个 Labels 字段从 Kubernetes 中过滤出它所关心的被控制对象。

  而这个过滤规则的定义，是在 Deployment 的“spec.selector.matchLabels”字段。我们一般称之为：Label Selector。

* Annotations

  在 Metadata 中，还有一个与 Labels 格式、层级完全相同的字段叫 Annotations，它专门用来携带 key-value 格式的内部信息。所谓内部信息，指的是对这些信息感兴趣的，是 Kubernetes 组件本身，而不是用户。所以大多数 Annotations，都是在 Kubernetes 运行过程中，被自动加在这个 API 对象上。

* emptyDir

  它其实就等同于我们之前讲过的 Docker 的隐式 Volume 参数，即：不显式声明宿主机目录的 Volume。所以，Kubernetes 也会在宿主机上创建一个临时目录，这个目录将来就会被绑定挂载到容器所声明的 Volume 目录上。

  不难看到，Kubernetes 的 emptyDir 类型，只是把 Kubernetes 创建的临时目录作为 Volume 的宿主机目录，交给了 Docker。这么做的原因，是 Kubernetes 不想依赖 Docker 自己创建的那个 _data 目录。

### 13 | 为什么我们需要 Pod？

* “单进程模型”

  容器的“单进程模型”，并不是指容器里只能运行“一个”进程，而是指容器没有管理多个进程的能力。这是因为容器里 PID=1 的进程就是应用本身，其他的进程都是这个 PID=1 进程的子进程。

* Pod

  Pod，是 Kubernetes 项目中最小的 API 对象。如果换一个更专业的说法，我们可以这样描述：Pod，是 Kubernetes 项目的原子调度单位。

* 进程组

  `pstree -g`

  不难发现，在一个真正的操作系统里，进程并不是“孤苦伶仃”地独自运行的，而是以进程组的方式，“有原则地”组织在一起。

  而 Kubernetes 项目所做的，其实就是将“进程组”的概念映射到了容器技术中，并使其成为了这个云计算“操作系统”里的“一等公民”。

* 成组调度（gang scheduling）

  Pod 是 Kubernetes 里的原子调度单位。这就意味着，Kubernetes 项目的调度器，是统一按照 Pod 而非容器的资源需求进行计算的。

* “超亲密关系”

  像这样容器间的紧密协作，我们可以称为“超亲密关系”。这些具有“超亲密关系”容器的典型特征包括但不限于：互相之间会发生直接的文件交换、使用 localhost 或者 Socket 文件进行本地通信、会发生非常频繁的远程调用、需要共享某些 Linux Namespace（比如，一个容器要加入另一个容器的 Network Namespace）等等。

* Pod 的实现原理

  首先，关于 Pod 最重要的一个事实是：它只是一个逻辑概念。

  也就是说，Kubernetes 真正处理的，还是宿主机操作系统上 Linux 容器的 Namespace 和 Cgroups，而并不存在一个所谓的 Pod 的边界或者隔离环境。

  Pod，其实是一组共享了某些资源的容器。

  具体的说：Pod 里的所有容器，共享的是同一个 Network Namespace，并且可以声明共享同一个 Volume。

  在 Kubernetes 项目里，Pod 的实现需要使用一个中间容器，这个容器叫作 Infra 容器。在这个 Pod 中，Infra 容器永远都是第一个被创建的容器，而其他用户定义的容器，则通过 Join Network Namespace 的方式，与 Infra 容器关联在一起。

  这也就意味着，对于 Pod 里的容器 A 和容器 B 来说：它们可以直接使用 localhost 进行通信；它们看到的网络设备跟 Infra 容器看到的完全一样；一个 Pod 只有一个 IP 地址，也就是这个 Pod 的 Network Namespace 对应的 IP 地址；当然，其他的所有网络资源，都是一个 Pod 一份，并且被该 Pod 中的所有容器共享；Pod 的生命周期只跟 Infra 容器一致，而与容器 A 和 B 无关。

  而对于同一个 Pod 里面的所有用户容器来说，它们的进出流量，也可以认为都是通过 Infra 容器完成的。

  有了这个设计之后，共享 Volume 就简单多了：Kubernetes 项目只要把所有 Volume 的定义都设计在 Pod 层级即可。这样，一个 Volume 对应的宿主机目录对于 Pod 来说就只有一个，Pod 里的容器只要声明挂载这个 Volume，就一定可以共享这个 Volume 对应的宿主机目录。

* 容器设计模式

  Pod 这种“超亲密关系”容器的设计思想，实际上就是希望，当用户想在一个容器里跑多个功能并不相关的应用时，应该优先考虑它们是不是更应该被描述成一个 Pod 里的多个容器。

  第一个最典型的例子是：WAR 包与 Web 服务器。

  我们可以把 WAR 包和 Tomcat 分别做成镜像，然后把它们作为一个 Pod 里的两个容器“组合”在一起。

  在 Pod 中，所有 Init Container 定义的容器，都会比 spec.containers 定义的用户容器先启动。并且，Init Container 容器会按顺序逐一启动，而直到它们都启动并且退出了，用户容器才会启动。

  实际上，这个所谓的“组合”操作，正是容器设计模式里最常用的一种模式，它的名字叫：sidecar。顾名思义，sidecar 指的就是我们可以在一个 Pod 中，启动一个辅助容器，来完成一些独立于主进程（主容器）之外的工作。

  第二个例子，则是容器的日志收集。

* 应用迁移

  实际上，一个运行在虚拟机里的应用，哪怕再简单，也是被管理在 systemd 或者 supervisord 之下的一组进程，而不是一个进程。这跟本地物理机上应用的运行方式其实是一样的。这也是为什么，从物理机到虚拟机之间的应用迁移，往往并不困难。

  可是对于容器来说，一个容器永远只能管理一个进程。更确切地说，一个容器，就是一个进程。这是容器技术的“天性”，不可能被修改。所以，将一个原本运行在虚拟机里的应用，“无缝迁移”到容器中的想法，实际上跟容器的本质是相悖的。

  这也是当初 Swarm 项目无法成长起来的重要原因之一：一旦到了真正的生产环境上，Swarm 这种单容器的工作方式，就难以描述真实世界里复杂的应用架构了。

  所以，你现在可以这么理解 Pod 的本质：Pod，实际上是在扮演传统基础设施里“虚拟机”的角色；而容器，则是这个虚拟机里运行的用户程序。

  所以下一次，当你需要把一个运行在虚拟机里的应用迁移到 Docker 容器中时，一定要仔细分析到底有哪些进程（组件）运行在这个虚拟机里。

  然后，你就可以把整个虚拟机想象成为一个 Pod，把这些进程分别做成容器镜像，把有顺序关系的容器，定义为 Init Container。这才是更加合理的、松耦合的容器编排诀窍，也是从传统应用架构，到“微服务架构”最自然的过渡方式。

  注意：Pod 这个概念，提供的是一种编排思想，而不是具体的技术方案。所以，如果愿意的话，你完全可以使用虚拟机来作为 Pod 的实现，然后把用户容器都运行在这个虚拟机里。甚至，你可以去实现一个带有 Init 进程的容器项目，来模拟传统应用的运行方式。

### 14 | 深入解析 Pod 对象（一）：基本概念

* Pod 级别的属性

  凡是调度、网络、存储，以及安全相关的属性，基本上是 Pod 级别的。

  NodeSelector：是一个供用户将 Pod 与 Node 进行绑定的字段。

  NodeName：一旦 Pod 的这个字段被赋值，Kubernetes 项目就会被认为这个 Pod 已经经过了调度，调度的结果就是赋值的节点名字。所以，这个字段一般由调度器负责设置，但用户也可以设置它来“骗过”调度器，当然这个做法一般是在测试或者调试的时候才会用到。

  HostAliases：定义了 Pod 的 hosts 文件（比如 /etc/hosts）里的内容。

  凡是跟容器的 Linux Namespace 相关的属性，也一定是 Pod 级别的。这个原因也很容易理解：Pod 的设计，就是要让它里面的容器尽可能多地共享 Linux Namespace，仅保留必要的隔离和限制能力。这样，Pod 模拟出的效果，就跟虚拟机里程序间的关系非常类似了。

  `shareProcessNamespace=true`

  这就意味着这个 Pod 里的容器要共享 PID Namespace。

  类似地，凡是 Pod 中的容器要共享宿主机的 Namespace，也一定是 Pod 级别的定义。

  `hostNetwork: true`，`hostIPC: true`，`hostPID: true`

  这就意味着，这个 Pod 里的所有容器，会直接使用宿主机的网络、直接与宿主机进行 IPC 通信、看到宿主机里正在运行的所有进程。

* Container 级别的属性

  我在前面的容器技术概念入门系列文章中，和你分享的 Image（镜像）、Command（启动命令）、workingDir（容器的工作目录）、Ports（容器要开发的端口），以及 volumeMounts（容器要挂载的 Volume）都是构成 Kubernetes 项目中 Container 的主要字段。不过在这里，还有这么几个属性值得你额外关注。

  首先，是 ImagePullPolicy 字段。它定义了镜像拉取的策略。

  ImagePullPolicy 的值默认是 Always，即每次创建 Pod 都重新拉取一次镜像。另外，当容器的镜像是类似于 nginx 或者 nginx:latest 这样的名字时，ImagePullPolicy 也会被认为 Always。

  而如果它的值被定义为 Never 或者 IfNotPresent，则意味着 Pod 永远不会主动拉取这个镜像，或者只在宿主机上不存在这个镜像时才拉取。

  其次，是 Lifecycle 字段。它定义的是 Container Lifecycle Hooks。顾名思义，Container Lifecycle Hooks 的作用，是在容器状态发生变化时触发一系列“钩子”。

  `postStart`，它指的是，在容器启动后，立刻执行一个指定的操作。需要明确的是，postStart 定义的操作，虽然是在 Docker 容器 ENTRYPOINT 执行之后，但它并不严格保证顺序。也就是说，在 postStart 启动时，ENTRYPOINT 有可能还没有结束。

  当然，如果 postStart 执行超时或者错误，Kubernetes 会在该 Pod 的 Events 中报出该容器启动失败的错误信息，导致 Pod 也处于失败的状态。

  `preStop`，preStop 发生的时机，则是容器被杀死之前（比如，收到了 SIGKILL 信号）。而需要明确的是，preStop 操作的执行，是同步的。所以，它会阻塞当前的容器杀死流程，直到这个 Hook 定义操作完成之后，才允许容器被杀死，这跟 postStart 不一样。

* Pod 生命周期

  Pod 生命周期的变化，主要体现在 Pod API 对象的 Status 部分，这是它除了 Metadata 和 Spec 之外的第三个重要字段。其中，pod.status.phase，就是 Pod 的当前状态，它有如下几种可能的情况：

  Pending。这个状态意味着，Pod 的 YAML 文件已经提交给了 Kubernetes，API 对象已经被创建并保存在 Etcd 当中。但是，这个 Pod 里有些容器因为某种原因而不能被顺利创建。比如，调度不成功。

  Running。这个状态下，Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中。

  Succeeded。这个状态意味着，Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见。

  Failed。这个状态下，Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志。

  Unknown。这是一个异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kube-apiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题。

  更进一步地，Pod 对象的 Status 字段，还可以再细分出一组 Conditions。这些细分状态的值包括：PodScheduled、Ready、Initialized，以及 Unschedulable。它们主要用于描述造成当前 Status 的具体原因是什么。

  而其中，Ready 这个细分状态非常值得我们关注：它意味着 Pod 不仅已经正常启动（Running 状态），而且已经可以对外提供服务了。

### 15 | 深入解析 Pod 对象（二）：使用进阶

* Projected Volume

  在 Kubernetes 中，有几种特殊的 Volume，它们存在的意义不是为了存放容器里的数据，也不是用来进行容器和宿主机之间的数据交换。这些特殊 Volume 的作用，是为容器提供预先定义好的数据。所以，从容器的角度来看，这些 Volume 里的信息就是仿佛是被 Kubernetes “投射”（Project）进入容器当中的。

  到目前为止，Kubernetes 支持的 Projected Volume 一共有四种：Secret；ConfigMap；Downward API；ServiceAccountToken。

  其实，Secret、ConfigMap，以及 Downward API 这三种 Projected Volume 定义的信息，大多还可以通过环境变量的方式出现在容器里。但是，通过环境变量获取这些信息的方式，不具备自动更新的能力。所以，一般情况下，我都建议你使用 Volume 文件的方式获取这些信息。

* Secret

  它的作用，是帮你把 Pod 想要访问的加密数据，存放到 Etcd 中。然后，你就可以通过在 Pod 的容器里挂载 Volume 的方式，访问到这些 Secret 里保存的信息了。

  更重要的是，像这样通过挂载方式进入到容器里的 Secret，一旦其对应的 Etcd 里的数据被更新，这些 Volume 里的文件内容，同样也会被更新。其实，这是 kubelet 组件在定时维护这些 Volume。

  需要注意的是，这个更新可能会有一定的延时。所以在编写应用程序时，在发起数据库连接的代码处写好重试和超时的逻辑，绝对是个好习惯。

* ConfigMap

  它与 Secret 的区别在于，ConfigMap 保存的是不需要加密的、应用所需的配置信息。

  `kubectl create configmap ui-config --from-file=ui.properties`

* Downward API

  让 Pod 里的容器能够直接获取到这个 Pod API 对象本身的信息。

  不过，需要注意的是，Downward API 能够获取到的信息，一定是 Pod 里的容器进程启动之前就能够确定下来的信息。而如果你想要获取 Pod 容器运行后才会出现的信息，比如，容器进程的 PID，那就肯定不能使用 Downward API 了，而应该考虑在 Pod 里定义一个 sidecar 容器。

* Service Account

  Service Account 对象的作用，就是 Kubernetes 系统内置的一种“服务账户”，它是 Kubernetes 进行权限分配的对象。

  像这样的 Service Account 的授权信息和文件，实际上保存在它所绑定的一个特殊的 Secret 对象里的。这个特殊的 Secret 对象，就叫作 ServiceAccountToken。任何运行在 Kubernetes 集群上的应用，都必须使用这个 ServiceAccountToken 里保存的授权信息，也就是 Token，才可以合法地访问 API Server。

  另外，为了方便使用，Kubernetes 已经为你提供了一个默认“服务账户”（default Service Account）。并且，任何一个运行在 Kubernetes 里的 Pod，都可以直接使用这个默认的 Service Account，而无需显示地声明挂载它。

  这是如何做到的呢？当然还是靠 Projected Volume 机制。如果你查看一下任意一个运行在 Kubernetes 集群里的 Pod，就会发现，每一个 Pod，都已经自动声明一个类型是 Secret、名为 default-token-xxxx 的 Volume，然后自动挂载在每个容器的一个固定目录上。

  所以说，Kubernetes 其实在每个 Pod 创建的时候，自动在它的 spec.volumes 部分添加上了默认 ServiceAccountToken 的定义，然后自动给每个容器加上了对应的 volumeMounts 字段。这个过程对于用户来说是完全透明的。

  这样，一旦 Pod 创建完成，容器里的应用就可以直接从这个默认 ServiceAccountToken 的挂载目录里访问到授权信息和文件。这个容器内的路径在 Kubernetes 里是固定的，即：/var/run/secrets/kubernetes.io/serviceaccount。

  所以，你的应用程序只要直接加载这些授权文件，就可以访问并操作 Kubernetes API 了。而且，如果你使用的是 Kubernetes 官方的 Client 包（k8s.io/client-go）的话，它还可以自动加载这个目录下的文件，你不需要做任何配置或者编码操作。

  这种把 Kubernetes 客户端以容器的方式运行在集群里，然后使用 default Service Account 自动授权的方式，被称作“InClusterConfig”，也是我最推荐的进行 Kubernetes API 编程的授权方式。

* 容器健康检查和恢复机制

  * 健康检查

    在 Kubernetes 中，你可以为 Pod 里的容器定义一个健康检查“探针”（Probe）。这样，kubelet 就会根据这个 Probe 的返回值决定这个容器的状态，而不是直接以容器是否运行（来自 Docker 返回的信息）作为依据。这种机制，是生产环境中保证应用健康存活的重要手段。

    除了在容器中执行命令外，livenessProbe 也可以定义为发起 HTTP 或者 TCP 请求的方式，定义格式如下：

    ```yaml
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
        initialDelaySeconds: 3
        periodSeconds: 3
    ```

    ```yaml
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
    ```

    所以，你的 Pod 其实可以暴露一个健康检查 URL（比如 /healthz），或者直接让健康检查去检测应用的监听端口。这两种配置方法，在 Web 服务类的应用中非常常用。

    readinessProbe 检查结果的成功与否，决定的这个 Pod 是不是能被通过 Service 的方式访问到，而并不影响 Pod 的生命周期。

  * 恢复机制

    Pod 恢复机制，也叫 restartPolicy。它是 Pod 的 Spec 部分的一个标准字段（pod.spec.restartPolicy），默认值是 Always，即：任何时候这个容器发生了异常，它一定会被重新创建。

    但一定要强调的是，Pod 的恢复过程，永远都是发生在当前节点上，而不会跑到别的节点上去。事实上，一旦一个 Pod 与一个节点（Node）绑定，除非这个绑定发生了变化（pod.spec.node 字段被修改），否则它永远都不会离开这个节点。这也就意味着，如果这个宿主机宕机了，这个 Pod 也不会主动迁移到其他节点上去。

    而如果你想让 Pod 出现在其他的可用节点上，就必须使用 Deployment 这样的“控制器”来管理 Pod，哪怕你只需要一个 Pod 副本。

    而作为用户，你还可以通过设置 restartPolicy，改变 Pod 的恢复策略。

    Always：在任何情况下，只要容器不在运行状态，就自动重启容器；

    OnFailure：只在容器异常时才自动重启容器；

    Never：从来不重启容器。

    如果你要关心这个容器退出后的上下文环境，比如容器退出后的日志、文件和目录，就需要将 restartPolicy 设置为 Never。因为一旦容器被自动重新创建，这些内容就有可能丢失掉了（被垃圾回收了）。

    restartPolicy 和 Pod 里容器的状态，以及 Pod 状态的对应关系：

    只要 Pod 的 restartPolicy 指定的策略允许重启异常的容器（比如：Always），那么这个 Pod 就会保持 Running 状态，并进行容器重启。否则，Pod 就会进入 Failed 状态 。

    对于包含多个容器的 Pod，只有它里面所有的容器都进入异常状态后，Pod 才会进入 Failed 状态。在此之前，Pod 都是 Running 状态。此时，Pod 的 READY 字段会显示正常容器的个数。

* PodPreset（Pod 预设置）

  需要说明的是，PodPreset 里定义的内容，只会在 Pod API 对象被创建之前追加在这个对象本身上，而不会影响任何 Pod 的控制器的定义。

  比如，我们现在提交的是一个 nginx-deployment，那么这个 Deployment 对象本身是永远不会被 PodPreset 改变的，被修改的只是这个 Deployment 创建出来的所有 Pod。

  这里有一个问题：如果你定义了同时作用于一个 Pod 对象的多个 PodPreset，会发生什么呢？

  实际上，Kubernetes 项目会帮你合并（Merge）这两个 PodPreset 要做的修改。而如果它们要做的修改有冲突的话，这些冲突字段就不会被修改。

### 16 | 编排其实很简单：谈谈“控制器”模型

* kube-controller-manager

  实际上，这个组件，就是一系列控制器的集合。

  `kubernetes/pkg/controller/`

  这个目录下面的每一个控制器，都以独有的方式负责某种编排功能。

  实际上，这些控制器之所以被统一放在 pkg/controller 目录下，就是因为它们都遵循 Kubernetes 项目中的一个通用编排模式，即：控制循环（control loop）。

* 控制循环

  在具体实现中，实际状态往往来自于 Kubernetes 集群本身。

  而期望状态，一般来自于用户提交的 YAML 文件。

  接下来，以 Deployment 为例，我和你简单描述一下它对控制器模型的实现：

  Deployment 控制器从 Etcd 中获取到所有携带了“app: nginx”标签的 Pod，然后统计它们的数量，这就是实际状态；

  Deployment 对象的 Replicas 字段的值就是期望状态；

  Deployment 控制器将两个状态做比较，然后根据比较结果，确定是创建 Pod，还是删除已有的 Pod。

  可以看到，一个 Kubernetes 对象的主要编排逻辑，实际上是在第三步的“对比”阶段完成的。这个操作，通常被叫作调谐（Reconcile）。这个调谐的过程，则被称作“Reconcile Loop”（调谐循环）或者“Sync Loop”（同步循环）。

* 控制器

  这个控制器对象本身，负责定义被管理对象的期望状态。

  而被控制对象的定义，则来自于一个“模板”。

  可以看到，Deployment 这个 template 字段里的内容，跟一个标准的 Pod 对象的 API 定义，丝毫不差。而所有被这个 Deployment 管理的 Pod 实例，其实都是根据这个 template 字段的内容创建出来的。

  像 Deployment 定义的 template 字段，在 Kubernetes 项目中有一个专有的名字，叫作 PodTemplate（Pod 模板）。

  类似 Deployment 这样的一个控制器，实际上都是由上半部分的控制器定义（包括期望状态），加上下半部分的被控制对象的模板组成的。

### 17 | 经典 PaaS 的记忆：作业副本与水平扩展

* Pod 的“水平扩展 / 收缩”

  Deployment 看似简单，但实际上，它实现了 Kubernetes 项目中一个非常重要的功能：Pod 的“水平扩展 / 收缩”（horizontal scaling out/in）。这个功能，是从 PaaS 时代开始，一个平台级项目就必须具备的编排能力。

  举个例子，如果你更新了 Deployment 的 Pod 模板（比如，修改了容器的镜像），那么 Deployment 就需要遵循一种叫作“滚动更新”（rolling update）的方式，来升级现有的容器。

* ReplicaSet

  一个 ReplicaSet 对象，其实就是由副本数目的定义和一个 Pod 模板组成的。

  更重要的是，Deployment 控制器实际操纵的，正是这样的 ReplicaSet 对象，而不是 Pod 对象。

* Deployment、ReplicaSet，以及 Pod 的关系

  ![Deployment、ReplicaSet，以及 Pod 的关系](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/Deployment%E3%80%81ReplicaSet%EF%BC%8C%E4%BB%A5%E5%8F%8A%20Pod%20%E7%9A%84%E5%85%B3%E7%B3%BB.jpg)

  通过这张图，我们就很清楚地看到，一个定义了 replicas=3 的 Deployment，与它的 ReplicaSet，以及 Pod 的关系，实际上是一种“层层控制”的关系。

  其中，ReplicaSet 负责通过“控制器模式”，保证系统中 Pod 的个数永远等于指定的个数（比如，3 个）。这也正是 Deployment 只允许容器的 restartPolicy=Always 的主要原因：只有在容器能保证自己始终是 Running 状态的前提下，ReplicaSet 调整 Pod 的个数才有意义。

  而在此基础上，Deployment 同样通过“控制器模式”，来操作 ReplicaSet 的个数和属性，进而实现“水平扩展 / 收缩”和“滚动更新”这两个编排动作。

* “水平扩展 / 收缩”

  “水平扩展 / 收缩”非常容易实现，Deployment Controller 只需要修改它所控制的 ReplicaSet 的 Pod 副本个数就可以了。

  `kubectl scale deployment nginx-deployment --replicas=4`

* “滚动更新”

  `kubectl get deployments`

  DESIRED：用户期望的 Pod 副本个数（spec.replicas 的值）；

  CURRENT：当前处于 Running 状态的 Pod 的个数；

  UP-TO-DATE：当前处于最新版本的 Pod 的个数，所谓最新版本指的是 Pod 的 Spec 部分与 Deployment 里 Pod 模板里定义的完全一致；

  AVAILABLE：当前已经可用的 Pod 的个数，即：既是 Running 状态，又是最新版本，并且已经处于 Ready（健康检查正确）状态的 Pod 的个数。

  `kubectl get rs`

  这个 ReplicaSet 的名字，则是由 Deployment 的名字和一个随机字符串共同组成。

  这个随机字符串叫作 pod-template-hash。ReplicaSet 会把这个随机字符串加在它所控制的所有 Pod 的标签里，从而保证这些 Pod 不会与集群里的其他 Pod 混淆。

  相比之下，Deployment 只是在 ReplicaSet 的基础上，添加了 UP-TO-DATE 这个跟版本有关的状态字段。

  `kubectl edit deployment/nginx-deployment`

  `kubectl rollout status deployment/nginx-deployment`

  像这样，将一个集群中正在运行的多个 Pod 版本，交替地逐一升级的过程，就是“滚动更新”。

  这种“滚动更新”的好处是显而易见的。

  比如，在升级刚开始的时候，集群里只有 1 个新版本的 Pod。如果这时，新版本 Pod 有问题启动不起来，那么“滚动更新”就会停止，从而允许开发和运维人员介入。而在这个过程中，由于应用本身还有两个旧版本的 Pod 在线，所以服务并不会受到太大的影响。

  当然，这也就要求你一定要使用 Pod 的 Health Check 机制检查应用的运行状态，而不是简单地依赖于容器的 Running 状态。要不然的话，虽然容器已经变成 Running 了，但服务很有可能尚未启动，“滚动更新”的效果也就达不到了。

  而为了进一步保证服务的连续性，Deployment Controller 还会确保，在任何时间窗口内，只有指定比例的 Pod 处于离线状态。同时，它也会确保，在任何时间窗口内，只有指定比例的新 Pod 被创建出来。这两个比例的值都是可以配置的，默认都是 DESIRED 值的 25%。

  ```yaml
  spec:
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
  ```

  maxSurge 指定的是除了 DESIRED 数量之外，在一次“滚动”中，Deployment 控制器还可以创建多少个新 Pod；而 maxUnavailable 指的是，在一次“滚动”中，Deployment 控制器可以删除多少个旧 Pod。

* “应用版本”

  Deployment 的控制器，实际上控制的是 ReplicaSet 的数目，以及每个 ReplicaSet 的属性。

  而一个应用的版本，对应的正是一个 ReplicaSet；这个版本应用的 Pod 数量，则由 ReplicaSet 通过它自己的控制器（ReplicaSet Controller）来保证。

  通过这样的多个 ReplicaSet 对象，Kubernetes 项目就实现了对多个“应用版本”的描述。

* Deployment 对应用进行版本控制

  `kubectl rollout undo deployment/nginx-deployment`

  `kubectl rollout history deployment/nginx-deployment`

  `kubectl rollout history deployment/nginx-deployment --revision=2`

  `kubectl rollout undo deployment/nginx-deployment --to-revision=2`

  Kubernetes 项目还提供了一个指令，使得我们对 Deployment 的多次更新操作，最后只生成一个 ReplicaSet。

  具体的做法是，在更新 Deployment 前，你要先执行一条 kubectl rollout pause 指令。

  这个 kubectl rollout pause 的作用，是让这个 Deployment 进入了一个“暂停”状态。

  所以接下来，你就可以随意使用 kubectl edit 或者 kubectl set image 指令，修改这个 Deployment 的内容了。

  由于此时 Deployment 正处于“暂停”状态，所以我们对 Deployment 的所有修改，都不会触发新的“滚动更新”，也不会创建新的 ReplicaSet。

  而等到我们对 Deployment 修改操作都完成之后，只需要再执行一条 kubectl rollout resume 指令，就可以把这个 Deployment“恢复”回来。

  而在这个 kubectl rollout resume 指令执行之前，在 kubectl rollout pause 指令之后的这段时间里，我们对 Deployment 进行的所有修改，最后只会触发一次“滚动更新”。

  那么，我们又该如何控制这些“历史” ReplicaSet 的数量呢？

  Deployment 对象有一个字段，叫作 spec.revisionHistoryLimit，就是 Kubernetes 为 Deployment 保留的“历史版本”个数。

* 两层控制关系

  Deployment 实际上是一个两层控制器。首先，它通过 ReplicaSet 的个数来描述应用的版本；然后，它再通过 ReplicaSet 的属性（比如 replicas 的值），来保证 Pod 的副本数量。

  Deployment 控制 ReplicaSet（版本），ReplicaSet 控制 Pod（副本数）。

  Kubernetes 项目对 Deployment 的设计，实际上是代替我们完成了对“应用”的抽象，使得我们可以使用这个 Deployment 对象来描述应用，使用 kubectl rollout 命令控制应用的版本。

### 18 | 深入理解 StatefulSet（一）：拓扑状态

* Deployment 对应用做了一个简单化假设

  它认为，一个应用的所有 Pod，是完全一样的。所以，它们互相之间没有顺序，也无所谓运行在哪台宿主机上。需要的时候，Deployment 就可以通过 Pod 模板创建新的 Pod；不需要的时候，Deployment 就可以“杀掉”任意一个 Pod。

* StatefulSet

  这种实例之间有不对等关系，以及实例对外部数据有依赖关系的应用，就被称为“有状态应用”（Stateful Application）。

  StatefulSet 的设计其实非常容易理解。它把真实世界里的应用状态，抽象为了两种情况：

  拓扑状态。这种情况意味着，应用的多个实例之间不是完全对等的关系。这些应用实例，必须按照某些顺序启动，比如应用的主节点 A 要先于从节点 B 启动。而如果你把 A 和 B 两个 Pod 删除掉，它们再次被创建出来时也必须严格按照这个顺序才行。并且，新创建出来的 Pod，必须和原来 Pod 的网络标识一样，这样原先的访问者才能使用同样的方法，访问到这个新 Pod。

  存储状态。这种情况意味着，应用的多个实例分别绑定了不同的存储数据。对于这些应用实例来说，Pod A 第一次读取到的数据，和隔了十分钟之后再次读取到的数据，应该是同一份，哪怕在此期间 Pod A 被重新创建过。这种情况最典型的例子，就是一个数据库应用的多个存储实例。

  所以，StatefulSet 的核心功能，就是通过某种方式记录这些状态，然后在 Pod 被重新创建时，能够为新 Pod 恢复这些状态。

* Service 是如何被访问的？

  第一种方式，是以 Service 的 VIP（Virtual IP，即：虚拟 IP）方式。比如：当我访问 10.0.23.1 这个 Service 的 IP 地址时，10.0.23.1 其实就是一个 VIP，它会把请求转发到该 Service 所代理的某一个 Pod 上。

  第二种方式，就是以 Service 的 DNS 方式。比如：这时候，只要我访问 “my-svc.my-namespace.svc.cluster.local” 这条 DNS 记录，就可以访问到名叫 my-svc 的 Service 所代理的某一个 Pod。

  而在第二种 Service DNS 的方式下，具体还可以分为两种处理方法：

  第一种处理方法，是 Normal Service。这种情况下，你访问 “my-svc.my-namespace.svc.cluster.local” 解析到的，正是 my-svc 这个 Service 的 VIP，后面的流程就跟 VIP 方式一致了。

  而第二种处理方法，正是 Headless Service。这种情况下，你访问 “my-svc.my-namespace.svc.cluster.local” 解析到的，直接就是 my-svc 代理的某一个 Pod 的 IP 地址。可以看到，这里的区别在于，Headless Service 不需要分配一个 VIP，而是可以直接以 DNS 记录的方式解析出被代理 Pod 的 IP 地址。

* Headless Service

  所谓的 Headless Service，其实仍是一个标准 Service 的 YAML 文件。只不过，它的 clusterIP 字段的值是：None，即：这个 Service，没有一个 VIP 作为“头”。这也就是 Headless 的含义。所以，这个 Service 被创建后并不会被分配一个 VIP，而是会以 DNS 记录的方式暴露出它所代理的 Pod。

  而它所代理的 Pod，依然是采用 Label Selector 机制选择出来的，即：所有携带了 app=nginx 标签的 Pod，都会被这个 Service 代理起来。

  当你按照这样的方式创建了一个 Headless Service 之后，它所代理的所有 Pod 的 IP 地址，都会被绑定一个这样格式的 DNS 记录：

  `<pod-name>.<svc-name>.<namespace>.svc.cluster.local`

  这个 DNS 记录，正是 Kubernetes 项目为 Pod 分配的唯一的“可解析身份”（Resolvable Identity）。

  有了这个“可解析身份”，只要你知道了一个 Pod 的名字，以及它对应的 Service 的名字，你就可以非常确定地通过这条 DNS 记录访问到 Pod 的 IP 地址。

* StatefulSet 如何使用 DNS 记录来维持 Pod 的拓扑状态？

  StatefulSet 这个控制器的主要作用之一，就是使用 Pod 模板创建 Pod 的时候，对它们进行编号，并且按照编号顺序逐一完成创建工作。而当 StatefulSet 的“控制循环”发现 Pod 的“实际状态”与“期望状态”不一致，需要新建或者删除 Pod 进行“调谐”的时候，它会严格按照这些 Pod 编号的顺序，逐一完成这些操作。

  与此同时，通过 Headless Service 的方式，StatefulSet 为每个 Pod 创建了一个固定并且稳定的 DNS 记录，来作为它的访问入口。

  实际上，在部署“有状态应用”的时候，应用的每个实例拥有唯一并且稳定的“网络标识”，是一个非常重要的假设。

  对于“有状态应用”实例的访问，你必须使用 DNS 记录或者 hostname 的方式，而绝不应该直接访问这些 Pod 的 IP 地址。

### 19 | 深入理解 StatefulSet（二）：存储状态

* PV，PVC

  Kubernetes 项目引入了一组叫作 Persistent Volume Claim（PVC）和 Persistent Volume（PV）的 API 对象，大大降低了用户声明和使用持久化 Volume 的门槛。

  第一步：定义一个 PVC，声明想要的 Volume 的属性：

  ```yaml
  kind: PersistentVolumeClaim
  apiVersion: v1
  metadata:
    name: pv-claim
  spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  ```

  第二步：在应用的 Pod 中，声明使用这个 PVC：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: pv-pod
  spec:
    containers:
      - name: pv-container
        image: nginx
        ports:
          - containerPort: 80
            name: "http-server"
        volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: pv-storage
    volumes:
      - name: pv-storage
        persistentVolumeClaim:
          claimName: pv-claim
  ```

  这时候，只要我们创建这个 PVC 对象，Kubernetes 就会自动为它绑定一个符合条件的 Volume。可是，这些符合条件的 Volume 又是从哪里来的呢？

  答案是，它们来自于由运维人员维护的 PV（Persistent Volume）对象。

  Kubernetes 中 PVC 和 PV 的设计，实际上类似于“接口”和“实现”的思想。开发者只要知道并会使用“接口”，即：PVC；而运维人员则负责给“接口”绑定具体的实现，即：PV。

  这种解耦，就避免了因为向开发者暴露过多的存储系统细节而带来的隐患。

* StatefulSet

  ```yaml
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: web
  spec:
    serviceName: "nginx"
    replicas: 2
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.9.1
          ports:
          - containerPort: 80
            name: web
          volumeMounts:
          - name: www
            mountPath: /usr/share/nginx/html
    volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
  ```

  也就是说，凡是被这个 StatefulSet 管理的 Pod，都会声明一个对应的 PVC；而这个 PVC 的定义，就来自于 volumeClaimTemplates 这个模板字段。更重要的是，这个 PVC 的名字，会被分配一个与这个 Pod 完全一致的编号。

  这个自动创建的 PVC，与 PV 绑定成功后，就会进入 Bound 状态，这就意味着这个 Pod 可以挂载并使用这个 PV 了。

  PVC 其实就是一种特殊的 Volume。只不过一个 PVC 具体是什么类型的 Volume，要在跟某个 PV 绑定之后才知道。

* StatefulSet 控制器恢复 Pod 的过程

  首先，当你把一个 Pod，比如 web-0，删除之后，这个 Pod 对应的 PVC 和 PV，并不会被删除，而这个 Volume 里已经写入的数据，也依然会保存在远程存储服务里（比如，我们在这个例子里用到的 Ceph 服务器）。

  此时，StatefulSet 控制器发现，一个名叫 web-0 的 Pod 消失了。所以，控制器就会重新创建一个新的、名字还是叫作 web-0 的 Pod 来“纠正”这个不一致的情况。

  需要注意的是，在这个新的 Pod 对象的定义里，它声明使用的 PVC 的名字，还是叫作：www-web-0。这个 PVC 的定义，还是来自于 PVC 模板（volumeClaimTemplates），这是 StatefulSet 创建 Pod 的标准流程。

  所以，在这个新的 web-0 Pod 被创建出来之后，Kubernetes 为它查找名叫 www-web-0 的 PVC 时，就会直接找到旧 Pod 遗留下来的同名的 PVC，进而找到跟这个 PVC 绑定在一起的 PV。

  这样，新的 Pod 就可以挂载到旧 Pod 对应的那个 Volume，并且获取到保存在 Volume 里的数据。

  通过这种方式，Kubernetes 的 StatefulSet 就实现了对应用存储状态的管理。

* StatefulSet 的工作原理

  首先，StatefulSet 的控制器直接管理的是 Pod。这是因为，StatefulSet 里的不同 Pod 实例，不再像 ReplicaSet 中那样都是完全一样的，而是有了细微区别的。比如，每个 Pod 的 hostname、名字等都是不同的、携带了编号的。而 StatefulSet 区分这些实例的方式，就是通过在 Pod 的名字里加上事先约定好的编号。

  其次，Kubernetes 通过 Headless Service，为这些有编号的 Pod，在 DNS 服务器中生成带有同样编号的 DNS 记录。只要 StatefulSet 能够保证这些 Pod 名字里的编号不变，那么 Service 里类似于 web-0.nginx.default.svc.cluster.local 这样的 DNS 记录也就不会变，而这条记录解析出来的 Pod 的 IP 地址，则会随着后端 Pod 的删除和再创建而自动更新。这当然是 Service 机制本身的能力，不需要 StatefulSet 操心。

  最后，StatefulSet 还为每一个 Pod 分配并创建一个同样编号的 PVC。这样，Kubernetes 就可以通过 Persistent Volume 机制为这个 PVC 绑定上对应的 PV，从而保证了每一个 Pod 都拥有一个独立的 Volume。

  在这种情况下，即使 Pod 被删除，它所对应的 PVC 和 PV 依然会保留下来。所以当这个 Pod 被重新创建出来之后，Kubernetes 会为它找到同样编号的 PVC，挂载这个 PVC 对应的 Volume，从而获取到以前保存在 Volume 里的数据。

* StatefulSet 的设计思想

  StatefulSet 其实就是一种特殊的 Deployment，而其独特之处在于，它的每个 Pod 都被编号了。而且，这个编号会体现在 Pod 的名字和 hostname 等标识信息上，这不仅代表了 Pod 的创建顺序，也是 Pod 的重要网络标识（即：在整个集群里唯一的、可被访问的身份）。

  有了这个编号后，StatefulSet 就使用 Kubernetes 里的两个标准功能：Headless Service 和 PV/PVC，实现了对 Pod 的拓扑状态和存储状态的维护。

### 20 | 深入理解 StatefulSet（三）：有状态应用实践

* Master 节点和 Slave 节点需要有不同的配置文件

  我们只需要给主从节点分别准备两份不同的 MySQL 配置文件，然后根据 Pod 的序号（Index）挂载进去即可。

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: mysql
    labels:
      app: mysql
  data:
    master.cnf: |
      # 主节点 MySQL 的配置文件
      [mysqld]
      log-bin
    slave.cnf: |
      # 从节点 MySQL 的配置文件
      [mysqld]
      super-read-only
  ```

  master.cnf 开启了 log-bin，即：使用二进制日志文件的方式进行主从复制，这是一个标准的设置。

  slave.cnf 开启了 super-read-only，代表的是从节点会拒绝除了主节点的数据同步操作之外的所有写操作，即：它对用户是只读的。

  创建两个 Service 来供 StatefulSet 以及用户使用：

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: mysql
    labels:
      app: mysql
  spec:
    ports:
    - name: mysql
      port: 3306
    clusterIP: None
    selector:
      app: mysql
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: mysql-read
    labels:
      app: mysql
  spec:
    ports:
    - name: mysql
      port: 3306
    selector:
      app: mysql
  ```

  第一个名叫“mysql”的 Service 是一个 Headless Service（即：clusterIP = None）。所以它的作用，是通过为 Pod 分配 DNS 记录来固定它的拓扑状态，比如“mysql-0.mysql”和“mysql-1.mysql”这样的 DNS 名字。其中，编号为 0 的节点就是我们的主节点。

* Master 节点和 Slave 节点需要能够传输备份信息文件

  ```yaml
  ...
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
  ```

  第一步：从 ConfigMap 中，获取 MySQL 的 Pod 对应的配置文件。

  ```yaml
        ...
        # template.spec
        initContainers:
        - name: init-mysql
          image: mysql:5.7
          command:
          - bash
          - "-c"
          - |
            set -ex
            # 从 Pod 的序号，生成 server-id
            [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
            ordinal=${BASH_REMATCH[1]}
            echo [mysqld] > /mnt/conf.d/server-id.cnf
            # 由于 server-id=0 有特殊含义，我们给 ID 加一个 100 来避开它
            echo server-id=$((100 + $ordinal)) >> /mnt/conf.d/server-id.cnf
            # 如果 Pod 序号是 0，说明它是 Master 节点，从 ConfigMap 里把 Master 的配置文件拷贝到 /mnt/conf.d/ 目录
            # 否则，拷贝 Slave 的配置文件
            if [[ $ordinal -eq 0 ]]; then
              cp /mnt/config-map/master.cnf /mnt/conf.d/
            else
              cp /mnt/config-map/slave.cnf /mnt/conf.d/
            fi
          volumeMounts:
          - name: conf
            mountPath: /mnt/conf.d
          - name: config-map
            mountPath: /mnt/config-map
  ```

  ```yaml
        ...
        # template.spec
        volumes:
        - name: conf
          emptyDir: {}
        - name: config-map
          configMap:
            name: mysql
  ```

  第二步：在 Slave Pod 启动前，从 Master 或者其他 Slave Pod 里拷贝数据库数据到自己的目录下。

  ```yaml
        ...
        # template.spec.initContainers
        - name: clone-mysql
          image: gcr.io/google-samples/xtrabackup:1.0
          command:
          - bash
          - "-c"
          - |
            set -ex
            # 拷贝操作只需要在第一次启动时进行，所以如果数据已经存在，跳过
            [[ -d /var/lib/mysql/mysql ]] && exit 0
            # Master 节点不需要做这个操作
            [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
            ordinal=${BASH_REMATCH[1]}
            [[ $ordinal -eq 0 ]] && exit 0
            # 使用 ncat 指令，远程地从前一个节点拷贝数据到本地
            ncat --recv-only mysql-$(($ordinal-1)).mysql 3307 | xbstream -x -C /var/lib/mysql
            # 执行 --prepare，这样拷贝来的数据就可以用作恢复了
            xtrabackup --prepare --target-dir=/var/lib/mysql
          volumeMounts:
          - name: data
            mountPath: /var/lib/mysql
            subPath: mysql
          - name: conf
            mountPath: /etc/mysql/conf.d
  ```

* 在 Slave 节点第一次启动之前，需要执行一些初始化 SQL 操作

  ```yaml
        ...
        # template.spec.containers
        - name: xtrabackup
          image: gcr.io/google-samples/xtrabackup:1.0
          ports:
          - name: xtrabackup
            containerPort: 3307
          command:
          - bash
          - "-c"
          - |
            set -ex
            cd /var/lib/mysql
            # 从备份信息文件里读取 MASTER_LOG_FILEM 和 MASTER_LOG_POS 这两个字段的值，用来拼装集群初始化 SQL
            if [[ -f xtrabackup_slave_info ]]; then
              # 如果 xtrabackup_slave_info 文件存在，说明这个备份数据来自于另一个 Slave 节点。这种情况下，XtraBackup 工具在备份的时候，就已经在这个文件里自动生成了"CHANGE MASTER TO" SQL 语句。所以，我们只需要把这个文件重命名为 change_master_to.sql.in，后面直接使用即可
              mv xtrabackup_slave_info change_master_to.sql.in
              # 所以，也就用不着 xtrabackup_binlog_info 了
              rm -f xtrabackup_binlog_info
            elif [[ -f xtrabackup_binlog_info ]]; then
              # 如果只存在 xtrabackup_binlog_inf 文件，那说明备份来自于 Master 节点，我们就需要解析这个备份信息文件，读取所需的两个字段的值
              [[ `cat xtrabackup_binlog_info` =~ ^(.*?)[[:space:]]+(.*?)$ ]] || exit 1
              rm xtrabackup_binlog_info
              # 把两个字段的值拼装成 SQL，写入 change_master_to.sql.in 文件
              echo "CHANGE MASTER TO MASTER_LOG_FILE='${BASH_REMATCH[1]}',\
                    MASTER_LOG_POS=${BASH_REMATCH[2]}" > change_master_to.sql.in
            fi
            # 如果 change_master_to.sql.in，就意味着需要做集群初始化工作
            if [[ -f change_master_to.sql.in ]]; then
              # 但一定要先等 MySQL 容器启动之后才能进行下一步连接 MySQL 的操作
              echo "Waiting for mysqld to be ready (accepting connections)"
              until mysql -h 127.0.0.1 -e "SELECT 1"; do sleep 1; done
              echo "Initializing replication from clone position"
              # 将文件 change_master_to.sql.in 改个名字，防止这个 Container 重启的时候，因为又找到了 change_master_to.sql.in，从而重复执行一遍这个初始化流程
              mv change_master_to.sql.in change_master_to.sql.orig
              # 使用 change_master_to.sql.orig 的内容，也是就是前面拼装的 SQL，组成一个完整的初始化和启动 Slave 的 SQL 语句
              mysql -h 127.0.0.1 <<EOF
            $(<change_master_to.sql.orig),
              MASTER_HOST='mysql-0.mysql',
              MASTER_USER='root',
              MASTER_PASSWORD='',
              MASTER_CONNECT_RETRY=10;
            START SLAVE;
            EOF
            fi
            # 使用 ncat 监听 3307 端口。它的作用是，在收到传输请求的时候，直接执行"xtrabackup --backup"命令，备份 MySQL 的数据并发送给请求者
            exec ncat --listen --keep-open --send-only --max-conns=1 3307 -c \
              "xtrabackup --backup --slave-info --stream=xbstream --host=127.0.0.1 --user=root"
          volumeMounts:
          - name: data
            mountPath: /var/lib/mysql
            subPath: mysql
          - name: conf
            mountPath: /etc/mysql/conf.d
  ```

  可以看到，在这个名叫 xtrabackup 的 sidecar 容器的启动命令里，其实实现了两部分工作。

  第一部分工作，当然是 MySQL 节点的初始化工作。

  在完成 MySQL 节点的初始化后，这个 sidecar 容器的第二个工作，则是启动一个数据传输服务。

* MySQL 容器

  ```yaml
        ...
        # template.spec
        containers:
        - name: mysql
          image: mysql:5.7
          env:
          - name: MYSQL_ALLOW_EMPTY_PASSWORD
            value: "1"
          ports:
          - name: mysql
            containerPort: 3306
          volumeMounts:
          - name: data
            mountPath: /var/lib/mysql
            subPath: mysql
          - name: conf
            mountPath: /etc/mysql/conf.d
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
          livenessProbe:
            exec:
              command: ["mysqladmin", "ping"]
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
          readinessProbe:
            exec:
              # 通过TCP连接的方式进行健康检查
              command: ["mysql", "-h", "127.0.0.1", "-e", "SELECT 1"]
            initialDelaySeconds: 5
            periodSeconds: 2
            timeoutSeconds: 1
  ```

  如果 MySQL 容器是 Slave 节点的话，它的数据目录里的数据，就来自于 InitContainer 从其他节点里拷贝而来的备份。它的配置文件目录 /etc/mysql/conf.d 里的内容，则来自于 ConfigMap 对应的 Volume。而它的初始化工作，则是由同一个 Pod 里的 sidecar 容器完成的。

  另外，我们为它定义了一个 livenessProbe，通过 mysqladmin ping 命令来检查它是否健康；还定义了一个 readinessProbe，通过查询 SQL（select 1）来检查 MySQL 服务是否可用。当然，凡是 readinessProbe 检查失败的 MySQL Pod，都会从 Service 里被摘除掉。

* StatefulSet 总结

  最后，相信你也已经能够理解，StatefulSet 其实是一种特殊的 Deployment，只不过这个“Deployment”的每个 Pod 实例的名字里，都携带了一个唯一并且固定的编号。这个编号的顺序，固定了 Pod 的拓扑关系；这个编号对应的 DNS 记录，固定了 Pod 的访问方式；这个编号对应的 PV，绑定了 Pod 与持久化存储的关系。所以，当 Pod 被删除重建时，这些“状态”都会保持不变。

* 如何对 StatefulSet 进行“滚动更新”（rolling update）？

  你只要修改 StatefulSet 的 Pod 模板，就会自动触发“滚动更新”：

  `kubectl patch statefulset mysql --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"mysql:5.7.23"}]'`

  这样，StatefulSet Controller 就会按照与 Pod 编号相反的顺序，从最后一个 Pod 开始，逐一更新这个 StatefulSet 管理的每个 Pod。而如果更新发生了错误，这次“滚动更新”就会停止。此外，StatefulSet 的“滚动更新”还允许我们进行更精细的控制，比如金丝雀发布（Canary Deploy）或者灰度发布，这意味着应用的多个实例中被指定的一部分不会被更新到最新的版本。

  这个字段，正是 StatefulSet 的 spec.updateStrategy.rollingUpdate 的 partition 字段。

  `kubectl patch statefulset mysql -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"partition":2}}}}'`

  这样，我就指定了当 Pod 模板发生变化的时候，比如 MySQL 镜像更新到 5.7.23，那么只有序号大于或者等于 2 的 Pod 会被更新到这个版本。并且，如果你删除或者重启了序号小于 2 的 Pod，等它再次启动后，也会保持原先的 5.7.2 版本，绝不会被升级到 5.7.23 版本。

### 21 | 容器化守护进程的意义：DaemonSet

* Daemon Pod 三个特征

  这个 Pod 运行在 Kubernetes 集群里的每一个节点（Node）上；

  每个节点上只有一个这样的 Pod 实例；

  当有新的节点加入 Kubernetes 集群后，该 Pod 会自动地在新节点上被创建出来；而当旧节点被删除后，它上面的 Pod 也相应地会被回收掉。

* DaemonSet Controller

  首先从 Etcd 里获取所有的 Node 列表，然后遍历所有的 Node。这时，它就可以很容易地去检查，当前这个 Node 上是不是有一个携带了 name=fluentd-elasticsearch 标签的 Pod 在运行。

  而检查的结果，可能有这么三种情况：

  没有这种 Pod，那么就意味着要在这个 Node 上创建这样一个 Pod；

  有这种 Pod，但是数量大于 1，那就说明要把多余的 Pod 从这个 Node 上删除掉；

  正好只有一个这种 Pod，那说明这个节点是正常的。

* nodeAffinity

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: with-node-affinity
  spec:
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: metadata.name
              operator: In
              values:
              - node-geektime
  ```

  requiredDuringSchedulingIgnoredDuringExecution：它的意思是说，这个 nodeAffinity 必须在每次调度的时候予以考虑。

  这个 Pod，将来只允许运行在“metadata.name”是“node-geektime”的节点上。

  我们的 DaemonSet Controller 会在创建 Pod 的时候，自动在这个 Pod 的 API 对象里，加上这样一个 nodeAffinity 定义。其中，需要绑定的节点名字，正是当前正在遍历的这个 Node。

  当然，DaemonSet 并不需要修改用户提交的 YAML 文件里的 Pod 模板，而是在向 Kubernetes 发起请求之前，直接修改根据模板生成的 Pod 对象。

* Toleration

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: with-toleration
  spec:
    tolerations:
    - key: node.kubernetes.io/unschedulable
      operator: Exists
      effect: NoSchedule
  ```

  这个 Toleration 的含义是：“容忍”所有被标记为 unschedulable “污点”的 Node；“容忍”的效果是允许调度。

  而在正常情况下，被标记了 unschedulable “污点”的 Node，是不会有任何 Pod 被调度上去的（effect: NoSchedule）。可是，DaemonSet 自动地给被管理的 Pod 加上了这个特殊的 Toleration，就使得这些 Pod 可以忽略这个限制，继而保证每个节点上都会被调度一个 Pod。当然，如果这个节点有故障的话，这个 Pod 可能会启动失败，而 DaemonSet 则会始终尝试下去，直到 Pod 启动成功。

  假如当前 DaemonSet 管理的，是一个网络插件的 Agent Pod，那么你就必须在这个 DaemonSet 的 YAML 文件里，给它的 Pod 模板加上一个能够“容忍” node.kubernetes.io/network-unavailable “污点”的 Toleration。

  在 Kubernetes 项目中，当一个节点的网络插件尚未安装时，这个节点就会被自动加上名为 node.kubernetes.io/network-unavailable 的“污点”。

  而通过这样一个 Toleration，调度器在调度这个 Pod 的时候，就会忽略当前节点上的“污点”，从而成功地将网络插件的 Agent 组件调度到这台机器上启动起来。

* DaemonSet 工作原理

  DaemonSet 其实是一个非常简单的控制器。在它的控制循环中，只需要遍历所有节点，然后根据节点上是否有被管理 Pod 的情况，来决定是否要创建或者删除一个 Pod。

  只不过，在创建每个 Pod 的时候，DaemonSet 会自动给这个 Pod 加上一个 nodeAffinity，从而保证这个 Pod 只会在指定节点上启动。同时，它还会自动给这个 Pod 加上一个 Toleration，从而忽略节点的 unschedulable “污点”。

* DaemonSet 版本

  在 Kubernetes 项目中，任何你觉得需要记录下来的状态，都可以被用 API 对象的方式实现。

  Kubernetes v1.7 之后添加了一个 API 对象，名叫 ControllerRevision，专门用来记录某种 Controller 对象的版本。

  `kubectl get controllerrevision -n kube-system -l name=fluentd-elasticsearch`

  `kubectl describe controllerrevision fluentd-elasticsearch-5467956799 -n kube-system`

  就会看到，这个 ControllerRevision 对象，实际上是在 Data 字段保存了该版本对应的完整的 DaemonSet 的 API 对象。并且，在 Annotation 字段保存了创建这个对象所使用的 kubectl 命令。

  `kubectl rollout undo daemonset fluentd-elasticsearch --to-revision=1 -n kube-system`

  这个 kubectl rollout undo 操作，实际上相当于读取到了 Revision=1 的 ControllerRevision 对象保存的 Data 字段。而这个 Data 字段里保存的信息，就是 Revision=1 时这个 DaemonSet 的完整 API 对象。

  所以，现在 DaemonSet Controller 就可以使用这个历史 API 对象，对现有的 DaemonSet 做一次 PATCH 操作（等价于执行一次 kubectl apply -f “旧的 DaemonSet 对象”），从而把这个 DaemonSet “更新”到一个旧版本。这也是为什么，在执行完这次回滚完成后，你会发现，DaemonSet 的 Revision 并不会从 Revision=2 退回到 1，而是会增加成 Revision=3。这是因为，一个新的 ControllerRevision 被创建了出来。

* DaemonSet 总结

  相比于 Deployment，DaemonSet 只管理 Pod 对象，然后通过 nodeAffinity 和 Toleration 这两个调度器的小功能，保证了每个节点上有且只有一个 Pod。

  与此同时，DaemonSet 使用 ControllerRevision，来保存和管理自己对应的“版本”。这种“面向 API 对象”的设计思路，大大简化了控制器本身的逻辑，也正是 Kubernetes 项目“声明式 API”的优势所在。

### 22 | 撬动离线业务：Job 与 CronJob

* Job API 对象

  可以看到，这个 Job 对象在创建后，它的 Pod 模板，被自动加上了一个 controller-uid=< 一个随机字符串 > 这样的 Label。而这个 Job 对象本身，则被自动加上了这个 Label 对应的 Selector，从而 保证了 Job 与它所管理的 Pod 之间的匹配关系。

  而 Job Controller 之所以要使用这种携带了 UID 的 Label，就是为了避免不同 Job 对象所管理的 Pod 发生重合。需要注意的是，这种自动生成的 Label 对用户来说并不友好，所以不太适合推广到 Deployment 等长作业编排对象上。

  事实上，restartPolicy 在 Job 对象里只允许被设置为 Never 和 OnFailure；而在 Deployment 对象里，restartPolicy 则只允许被设置为 Always。

  我们在这个例子中定义了 restartPolicy=Never，那么离线作业失败后 Job Controller 就会不断地尝试创建一个新 Pod。

  当然，这个尝试肯定不能无限进行下去。所以，我们就在 Job 对象的 spec.backoffLimit 字段里定义了重试次数为 4（即，backoffLimit=4），而这个字段的默认值是 6。

  需要注意的是，Job Controller 重新创建 Pod 的间隔是呈指数增加的，即下一次重新创建 Pod 的动作会分别发生在 10 s、20 s、40 s ... 后。

  而如果你定义的 restartPolicy=OnFailure，那么离线作业失败后，Job Controller 就不会去尝试创建新的 Pod。但是，它会不断地尝试重启 Pod 里的容器。

  在 Job 的 API 对象里，有一个 spec.activeDeadlineSeconds 字段可以设置最长运行时间，比如：

  ```yaml
  spec:
    backoffLimit: 5
    activeDeadlineSeconds: 100
  ```

  一旦运行超过了 100 s，这个 Job 的所有 Pod 都会被终止。并且，你可以在 Pod 的状态里看到终止的原因是 reason: DeadlineExceeded。

* Job Controller 对并行作业的控制方法

  在 Job 对象中，负责并行控制的参数有两个：

  spec.parallelism，它定义的是一个 Job 在任意时间最多可以启动多少个 Pod 同时运行；

  spec.completions，它定义的是 Job 至少要完成的 Pod 数目，即 Job 的最小完成数。

* Job Controller 的工作原理

  首先，Job Controller 控制的对象，直接就是 Pod。

  其次，Job Controller 在控制循环中进行的调谐（Reconcile）操作，是根据实际在 Running 状态 Pod 的数目、已经成功退出的 Pod 的数目，以及 parallelism、completions 参数的值共同计算出在这个周期里，应该创建或者删除的 Pod 数目，然后调用 Kubernetes API 来执行这个操作。

* 三种常用的使用 Job 对象的方法

  第一种用法，也是最简单粗暴的用法：外部管理器 + Job 模板。

  这种模式的特定用法是：把 Job 的 YAML 文件定义为一个“模板”，然后用一个外部工具控制这些“模板”来生成 Job。

  ```yaml
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: process-item-$ITEM
    labels:
      jobgroup: jobexample
  spec:
    template:
      metadata:
        name: jobexample
        labels:
          jobgroup: jobexample
      spec:
        containers:
        - name: c
          image: busybox
          command: ["sh", "-c", "echo Processing item $ITEM && sleep 5"]
        restartPolicy: Never
  ```

  ```shell
  mkdir ./jobs
  for i in apple banana cherry
  do
    cat job-tmpl.yaml | sed "s/\$ITEM/$i/" > ./jobs/job-$i.yaml
  done
  
  kubectl create -f ./jobs
  ```

  很容易理解，在这种模式下使用 Job 对象，completions 和 parallelism 这两个字段都应该使用默认值 1，而不应该由我们自行设置。而作业 Pod 的并行控制，应该完全交由外部工具来进行管理。

  第二种用法：拥有固定任务数目的并行 Job。

  这种模式下，我只关心最后是否有指定数目（spec.completions）个任务成功退出。至于执行时的并行度是多少，我并不关心。

  第三种用法，也是很常用的一个用法：指定并行度（parallelism），但不设置固定的 completions 的值。

  此时，你就必须自己想办法，来决定什么时候启动新 Pod，什么时候 Job 才算执行完成。

* CronJob

  CronJob 与 Job 的关系，正如同 Deployment 与 ReplicaSet 的关系一样。CronJob 是一个专门用来管理 Job 对象的控制器。只不过，它创建和删除 Job 的依据，是 schedule 字段定义的一个标准的 Unix Cron 格式的表达式。

  你可以通过 spec.concurrencyPolicy 字段来定义具体的处理策略。比如：

  concurrencyPolicy=Allow，这也是默认情况，这意味着这些 Job 可以同时存在；

  concurrencyPolicy=Forbid，这意味着不会创建新的 Pod，该创建周期被跳过；

  concurrencyPolicy=Replace，这意味着新产生的 Job 会替换旧的、没有执行完的 Job。

  而如果某一次 Job 创建失败，这次创建就会被标记为“miss”。当在指定的时间窗口内，miss 的数目达到 100 时，那么 CronJob 会停止再创建这个 Job。

  这个时间窗口，可以由 spec.startingDeadlineSeconds 字段指定。

### 23 | 声明式 API 与 Kubernetes 编程范式

* 声明式 API

  命令式命令行操作：

  ```shell
  docker service create --name nginx --replicas 2 nginx
  docker service update --image nginx:1.7.9 nginx
  ```

  命令式配置文件操作：

  ```shell
  kubectl create -f nginx.yaml
  kubectl replace -f nginx.yaml
  ```

  声明式 API：

  ```shell
  kubectl apply -f nginx.yaml
  ```

  kubectl replace 的执行过程，是使用新的 YAML 文件中的 API 对象，替换原有的 API 对象；而 kubectl apply，则是执行了一个对原有 API 对象的 PATCH 操作。

  更进一步地，这意味着 kube-apiserver 在响应命令式请求（比如，kubectl replace）的时候，一次只能处理一个写请求，否则会有产生冲突的可能。而对于声明式请求（比如，kubectl apply），一次能处理多个写操作，并且具备 Merge 能力。

* Istio

  Istio 最根本的组件，是运行在每一个应用 Pod 里的 Envoy 容器。

  这个 Envoy 项目是 Lyft 公司推出的一个高性能 C++ 网络代理，也是 Lyft 公司对 Istio 项目的唯一贡献。

  而 Istio 项目，则把这个代理服务以 sidecar 容器的方式，运行在了每一个被治理的应用 Pod 中。我们知道，Pod 里的所有容器都共享同一个 Network Namespace。所以，Envoy 容器就能够通过配置 Pod 里的 iptables 规则，把整个 Pod 的进出流量接管下来。

  这时候，Istio 的控制层（Control Plane）里的 Pilot 组件，就能够通过调用每个 Envoy 容器的 API，对这个 Envoy 代理进行配置，从而实现微服务治理。

  更重要的是，在整个微服务治理的过程中，无论是对 Envoy 容器的部署，还是像上面这样对 Envoy 代理的配置，用户和应用都是完全“无感”的。

* Dynamic Admission Control

  在 Kubernetes 项目中，当一个 Pod 或者任何一个 API 对象被提交给 APIServer 之后，总有一些“初始化”性质的工作需要在它们被 Kubernetes 项目正式处理之前进行。比如，自动为所有 Pod 加上某些标签（Labels）。

  而这个“初始化”操作的实现，借助的是一个叫作 Admission 的功能。它其实是 Kubernetes 项目里一组被称为 Admission Controller 的代码，可以选择性地被编译进 APIServer 中，在 API 对象创建之后会被立刻调用到。

  但这就意味着，如果你现在想要添加一些自己的规则到 Admission Controller，就会比较困难。因为，这要求重新编译并重启 APIServer。

  所以，Kubernetes 项目为我们额外提供了一种“热插拔”式的 Admission 机制，它就是 Dynamic Admission Control，也叫作：Initializer。

* 为 Pod “自动注入” Envoy 容器的 Initializer

  首先，Istio 会将这个 Envoy 容器本身的定义，以 ConfigMap 的方式保存在 Kubernetes 当中。

  不难想到，Initializer 要做的工作，就是把这部分 Envoy 相关的字段，自动添加到用户提交的 Pod 的 API 对象里。可是，用户提交的 Pod 里本来就有 containers 字段和 volumes 字段，所以 Kubernetes 在处理这样的更新请求时，就必须使用类似于 git merge 这样的操作，才能将这两部分内容合并在一起。所以说，在 Initializer 更新用户的 Pod 对象的时候，必须使用 PATCH API 来完成。而这种 PATCH API，正是声明式 API 最主要的能力。

  接下来，Istio 将一个编写好的 Initializer，作为一个 Pod 部署在 Kubernetes 中。

  一个 Kubernetes 的控制器，实际上就是一个“死循环”：它不断地获取“实际状态”，然后与“期望状态”作对比，并以此为依据决定下一步的操作。

  而 Initializer 的控制器，不断获取到的“实际状态”，就是用户新创建的 Pod。而它的“期望状态”，则是：这个 Pod 里被添加了 Envoy 容器的定义。

  ```go
  for {
    // 获取新创建的 Pod
    pod := client.GetLatestPod()
    // Diff 一下，检查是否已经初始化过
    if !isInitialized(pod) {
      // 没有？那就来初始化一下
      doSomething(pod)
    }
  }
  ```

  Kubernetes 的 API 库，为我们提供了一个方法，使得我们可以直接使用新旧两个 Pod 对象，生成一个 TwoWayMergePatch：

  ```go
  func doSomething(pod) {
    cm := client.Get(ConfigMap, "envoy-initializer")
  
    newPod := Pod{}
    newPod.Spec.Containers = cm.Containers
    newPod.Spec.Volumes = cm.Volumes
  
    // 生成 patch 数据
    patchBytes := strategicpatch.CreateTwoWayMergePatch(pod, newPod)
  
    // 发起 PATCH 请求，修改这个 pod 对象
    client.Patch(pod.Name, patchBytes)
  }
  ```

  有了这个 TwoWayMergePatch 之后，Initializer 的代码就可以使用这个 patch 的数据，调用 Kubernetes 的 Client，发起一个 PATCH 请求。

  这样，一个用户提交的 Pod 对象里，就会被自动加上 Envoy 容器相关的字段。

  当然，Kubernetes 还允许你通过配置，来指定要对什么样的资源进行这个 Initialize 操作，比如下面这个例子：

  ```yaml
  apiVersion: admissionregistration.k8s.io/v1alpha1
  kind: InitializerConfiguration
  metadata:
    name: envoy-config
  initializers:
    // 这个名字必须至少包括两个 "."
    - name: envoy.initializer.kubernetes.io
      rules:
        - apiGroups:
            - "" // 前面说过，"" 就是 core API Group 的意思
          apiVersions:
            - v1
          resources:
            - pods
  ```

  这个配置，就意味着 Kubernetes 要对所有的 Pod 进行这个 Initialize 操作，并且，我们指定了负责这个操作的 Initializer，名叫：envoy-initializer。

  而一旦这个 InitializerConfiguration 被创建，Kubernetes 就会把这个 Initializer 的名字，加在所有新创建的 Pod 的 Metadata 上。

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    initializers:
      pending:
        - name: envoy.initializer.kubernetes.io
    name: myapp-pod
    labels:
      app: myapp
  ...
  ```

  可以看到，每一个新创建的 Pod，都会自动携带了 metadata.initializers.pending 的 Metadata 信息。

  这个 Metadata，正是接下来 Initializer 的控制器判断这个 Pod 有没有执行过自己所负责的初始化操作的重要依据（也就是前面伪代码中 isInitialized() 方法的含义）。

  这也就意味着，当你在 Initializer 里完成了要做的操作后，一定要记得将这个 metadata.initializers.pending 标志清除掉。这一点，你在编写 Initializer 代码的时候一定要非常注意。

  此外，除了上面的配置方法，你还可以在具体的 Pod 的 Annotation 里添加一个如下所示的字段，从而声明要使用某个 Initializer：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    annotations:
      "initializer.kubernetes.io/envoy": "true"
      ...
  ```

* Kubernetes “声明式 API”的独特之处

  首先，所谓“声明式”，指的就是我只需要提交一个定义好的 API 对象来“声明”我所期望的状态是什么样子。

  其次，“声明式 API”允许有多个 API 写端，以 PATCH 的方式对 API 对象进行修改，而无需关心本地原始 YAML 文件的内容。

  最后，也是最重要的，有了上述两个能力，Kubernetes 项目才可以基于对 API 对象的增、删、改、查，在完全无需外界干预的情况下，完成对“实际状态”和“期望状态”的调谐（Reconcile）过程。

* Kubernetes 编程范式

  如何使用控制器模式，同 Kubernetes 里 API 对象的“增、删、改、查”进行协作，进而完成用户业务逻辑的编写过程。

### 24 | 深入解析声明式 API（一）：API 对象的奥秘

* 声明式 API 的设计

  在 Kubernetes 项目中，一个 API 对象在 Etcd 里的完整资源路径，是由：Group（API 组）、Version（API 版本）和 Resource（API 资源类型）三个部分组成的。

  通过这样的结构，整个 Kubernetes 里的所有 API 对象，实际上就可以用如下的树形结构表示出来：

  ![Kubernetes API 对象树形结构](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/Kubernetes%20API%20%E5%AF%B9%E8%B1%A1%E6%A0%91%E5%BD%A2%E7%BB%93%E6%9E%84.jpg)

  在这幅图中，你可以很清楚地看到 Kubernetes 里 API 对象的组织方式，其实是层层递进的。

* 找到 CronJob 对象定义

  首先，Kubernetes 会匹配 API 对象的组。

  需要明确的是，对于 Kubernetes 里的核心 API 对象，比如：Pod、Node 等，是不需要 Group 的（即：它们的 Group 是“”）。所以，对于这些 API 对象来说，Kubernetes 会直接在 /api 这个层级进行下一步的匹配过程。

  而对于 CronJob 等非核心 API 对象来说，Kubernetes 就必须在 /apis 这个层级里查找它对应的 Group，进而根据“batch”这个 Group 的名字，找到 /apis/batch。

  然后，Kubernetes 会进一步匹配到 API 对象的版本号。

  在 Kubernetes 中，同一种 API 对象可以有多个版本，这正是 Kubernetes 进行 API 版本化管理的重要手段。

  最后，Kubernetes 会匹配 API 对象的资源类型。

* APIServer 创建 CronJob 对象

  ![APIServer 创建 CronJob 对象](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/APIServer%20%E5%88%9B%E5%BB%BA%20CronJob%20%E5%AF%B9%E8%B1%A1.jpg)

  首先，当我们发起了创建 CronJob 的 POST 请求之后，我们编写的 YAML 的信息就被提交给了 APIServer。

  而 APIServer 的第一个功能，就是过滤这个请求，并完成一些前置性的工作，比如授权、超时处理、审计等。

  然后，请求会进入 MUX 和 Routes 流程。如果你编写过 Web Server 的话就会知道，MUX 和 Routes 是 APIServer 完成 URL 和 Handler 绑定的场所。而 APIServer 的 Handler 要做的事情，就是按照我刚刚介绍的匹配过程，找到对应的 CronJob 类型定义。

  接着，APIServer 最重要的职责就来了：根据这个 CronJob 类型定义，使用用户提交的 YAML 文件里的字段，创建一个 CronJob 对象。

  而在这个过程中，APIServer 会进行一个 Convert 工作，即：把用户提交的 YAML 文件，转换成一个叫作 Super Version 的对象，它正是该 API 资源类型所有版本的字段全集。这样用户提交的不同版本的 YAML 文件，就都可以用这个 Super Version 对象来进行处理了。

  接下来，APIServer 会先后进行 Admission() 和 Validation() 操作。比如，我在上一篇文章中提到的 Admission Controller 和 Initializer，就都属于 Admission 的内容。

  而 Validation，则负责验证这个对象里的各个字段是否合法。这个被验证过的 API 对象，都保存在了 APIServer 里一个叫作 Registry 的数据结构中。也就是说，只要一个 API 对象的定义能在 Registry 里查到，它就是一个有效的 Kubernetes API 对象。

  最后，APIServer 会把验证过的 API 对象转换成用户最初提交的版本，进行序列化操作，并调用 Etcd 的 API 把它保存起来。

* CRD

  CRD 的全称是 Custom Resource Definition。顾名思义，它指的就是，允许用户在 Kubernetes 中添加一个跟 Pod、Node 类似的、新的 API 资源类型，即：自定义 API 资源。

  [resouer/k8s-controller-custom-resource](https://github.com/resouer/k8s-controller-custom-resource.git)

  可以看到，它其实定义了两部分内容：

  第一部分是，自定义资源类型的 API 描述，包括：组（Group）、版本（Version）、资源类型（Resource）等。

  第二部分是，自定义资源类型的对象描述，包括：Spec、Status 等。

### 25 | 深入解析声明式 API（二）：编写自定义控制器

“声明式 API”并不像“命令式 API”那样有着明显的执行逻辑。这就使得基于声明式 API 的业务功能实现，往往需要通过控制器模式来“监视”API 对象的变化（比如，创建或者删除 Network），然后以此来决定实际要执行的具体工作。

总得来说，编写自定义控制器代码的过程包括：编写 main 函数，编写自定义控制器的定义，以及编写控制器里的业务逻辑三个部分。

* 编写 main 函数

  main 函数的主要工作就是，定义并初始化一个自定义控制器（Custom Controller），然后启动它。

  第一步：main 函数根据我提供的 Master 配置（APIServer 的地址端口和 kubeconfig 的路径），创建一个 Kubernetes 的 client（kubeClient）和 Network 对象的 client（networkClient）。

  但是，如果我没有提供 Master 配置呢？

  这时，main 函数会直接使用一种名叫 InClusterConfig 的方式来创建这个 client。这个方式，会假设你的自定义控制器是以 Pod 的方式运行在 Kubernetes 集群里的。

  Kubernetes 里所有的 Pod 都会以 Volume 的方式自动挂载 Kubernetes 的默认 ServiceAccount。所以，这个控制器就会直接使用默认 ServiceAccount 数据卷里的授权信息，来访问 APIServer。

  第二步：main 函数为 Network 对象创建一个叫作 InformerFactory 的工厂，并使用它生成一个 Network 对象的 Informer，传递给控制器。

  第三步：main 函数启动上述的 Informer，然后执行 controller.Run，启动自定义控制器。

* 自定义控制器的工作原理

  ![自定义控制器的工作流程示意图](https://github.com/songor/cloud-native-learned/blob/master/%E6%B7%B1%E5%85%A5%E5%89%96%E6%9E%90%20Kubernetes/images/%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8E%A7%E5%88%B6%E5%99%A8%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg)

  这个控制器要做的第一件事，是从 Kubernetes 的 APIServer 里获取它所关心的对象，也就是我定义的 Network 对象。

  这个操作，依靠的是一个叫作 Informer（可以翻译为：通知器）的代码库完成的。Informer 与 API 对象是一一对应的，所以我传递给自定义控制器的，正是一个 Network 对象的 Informer（Network Informer）。

  不知你是否已经注意到，我在创建这个 Informer 工厂的时候，需要给它传递一个 networkClient。

  事实上，Network Informer 正是使用这个 networkClient，跟 APIServer 建立了连接。不过，真正负责维护这个连接的，则是 Informer 所使用的 Reflector 包。

  更具体地说，Reflector 使用的是一种叫作 ListAndWatch 的方法，来“获取”并“监听”这些 Network 对象实例的变化。

  在 ListAndWatch 机制下，一旦 APIServer 端有新的 Network 实例被创建、删除或者更新，Reflector 都会收到“事件通知”。这时，该事件及它对应的 API 对象这个组合，就被称为增量（Delta），它会被放进一个 Delta FIFO Queue（即：增量先进先出队列）中。

  而另一方面，Informer 会不断地从这个 Delta FIFO Queue 里读取（Pop）增量。每拿到一个增量，Informer 就会判断这个增量里的事件类型，然后创建或者更新本地对象的缓存。这个缓存，在 Kubernetes 里一般被叫作 Store。

  比如，如果事件类型是 Added（添加对象），那么 Informer 就会通过一个叫作 Indexer 的库把这个增量里的 API 对象保存在本地缓存中，并为它创建索引。相反，如果增量的事件类型是 Deleted（删除对象），那么 Informer 就会从本地缓存中删除这个对象。

  这个同步本地缓存的工作，是 Informer 的第一个职责，也是它最重要的职责。

  而 Informer 的第二个职责，则是根据这些事件的类型，触发事先注册好的 ResourceEventHandler。这些 Handler，需要在创建控制器的时候注册给它对应的 Informer。

  我为 networkInformer 注册了三个 Handler（AddFunc、UpdateFunc 和 DeleteFunc），分别对应 API 对象的“添加”“更新”和“删除”事件。而具体的处理操作，都是将该事件对应的 API 对象加入到工作队列中。

  需要注意的是，实际入队的并不是 API 对象本身，而是它们的 Key，即：该 API 对象的 \<namespace\>/\<name\>。

  而我们后面即将编写的控制循环，则会不断地从这个工作队列里拿到这些 Key，然后开始执行真正的控制逻辑。

  综合上面的讲述，你现在应该就能明白，所谓 Informer，其实就是一个带有本地缓存和索引机制的、可以注册 EventHandler 的 client。它是自定义控制器跟 APIServer 进行数据同步的重要组件。

  更具体地说，Informer 通过一种叫作 ListAndWatch 的方法，把 APIServer 中的 API 对象缓存在了本地，并负责更新和维护这个缓存。

  其中，ListAndWatch 方法的含义是：首先，通过 APIServer 的 LIST API “获取”所有最新版本的 API 对象；然后，再通过 WATCH API 来“监听”所有这些 API 对象的变化。

  而通过监听到的事件变化，Informer 就可以实时地更新本地缓存，并且调用这些事件对应的 EventHandler 了。

  此外，在这个过程中，每经过 resyncPeriod 指定的时间，Informer 维护的本地缓存，都会使用最近一次 LIST 返回的结果强制更新一次，从而保证缓存的有效性。在 Kubernetes 中，这个缓存强制更新的操作就叫作：resync。

  需要注意的是，这个定时 resync 操作，也会触发 Informer 注册的“更新”事件。但此时，这个“更新”事件对应的 Network 对象实际上并没有发生变化，即：新、旧两个 Network 对象的 ResourceVersion 是一样的。在这种情况下，Informer 就不需要对这个更新事件再做进一步的处理了。

  接下来，我们就来到了示意图中最后面的控制循环（Control Loop）部分，也正是我在 main 函数最后调用 controller.Run() 启动的“控制循环”。

  可以看到，启动控制循环的逻辑非常简单：

  首先，等待 Informer 完成一次本地缓存的数据同步操作；

  然后，直接通过 goroutine 启动一个（或者并发启动多个）“无限循环”的任务。

  而这个“无限循环”任务的每一个循环周期，执行的正是我们真正关心的业务逻辑。

* 编写自定义控制器的业务逻辑

  可以看到，在这个执行周期里（processNextWorkItem），我们首先从工作队列里出队（workqueue.Get）了一个成员，也就是一个 Key（Network 对象的 namespace/name）。

  然后，在 syncHandler 方法中，我使用这个 Key，尝试从 Informer 维护的缓存中拿到了它所对应的 Network 对象。

  可以看到，在这里，我使用了 networksLister 来尝试获取这个 Key 对应的 Network 对象。这个操作，其实就是在访问本地缓存的索引。实际上，在 Kubernetes 的源码中，你会经常看到控制器从各种 Lister 里获取对象，比如：podLister、nodeLister 等等，它们使用的都是 Informer 和缓存机制。

  而如果控制循环从缓存中拿不到这个对象（即：networkLister 返回了 IsNotFound 错误），那就意味着这个 Network 对象的 Key 是通过前面的“删除”事件添加进工作队列的。所以，尽管队列里有这个 Key，但是对应的 Network 对象已经被删除了。

  这时候，我就需要调用 Neutron 的 API，把这个 Key 对应的 Neutron 网络从真实的集群里删除掉。

  而如果能够获取到对应的 Network 对象，我就可以执行控制器模式里的对比“期望状态”和“实际状态”的逻辑了。

  其中，自定义控制器“千辛万苦”拿到的这个 Network 对象，正是 APIServer 里保存的“期望状态”，即：用户通过 YAML 文件提交到 APIServer 里的信息。当然，在我们的例子里，它已经被 Informer 缓存在了本地。

  那么，“实际状态”又从哪里来呢？

  当然是来自于实际的集群了。

  所以，我们的控制循环需要通过 Neutron API 来查询实际的网络情况。

  比如，我可以先通过 Neutron 来查询这个 Network 对象对应的真实网络是否存在。

  如果不存在，这就是一个典型的“期望状态”与“实际状态”不一致的情形。这时，我就需要使用这个 Network 对象里的信息（比如：CIDR 和 Gateway），调用 Neutron API 来创建真实的网络。

  如果存在，那么，我就要读取这个真实网络的信息，判断它是否跟 Network 对象里的信息一致，从而决定我是否要通过 Neutron 来更新这个已经存在的真实网络。

  这样，我就通过对比“期望状态”和“实际状态”的差异，完成了一次调协（Reconcile）的过程。

* Kubernetes API 编程范式的核心思想

  所谓的 Informer，就是一个自带缓存和索引机制，可以触发 Handler 的客户端库。这个本地缓存在 Kubernetes 中一般被称为 Store，索引一般被称为 Index。

  Informer 使用了 Reflector 包，它是一个可以通过 ListAndWatch 机制获取并监视 API 对象变化的客户端封装。

  Reflector 和 Informer 之间，用到了一个“增量先进先出队列”进行协同。而 Informer 与你要编写的控制循环之间，则使用了一个工作队列来进行协同。

  在实际应用中，除了控制循环之外的所有代码，实际上都是 Kubernetes 为你自动生成的，即：pkg/client/{informers, listers, clientset}里的内容。

  而这些自动生成的代码，就为我们提供了一个可靠而高效地获取 API 对象“期望状态”的编程库。

  所以，接下来，作为开发者，你就只需要关注如何拿到“实际状态”，然后如何拿它去跟“期望状态”做对比，从而决定接下来要做的业务逻辑即可。

### 26 | 基于角色的权限控制：RBAC

Kubernetes 中所有的 API 对象，都保存在 Etcd 里。可是，对这些 API 对象的操作，却一定都是通过访问 kube-apiserver 实现的。其中一个非常重要的原因，就是你需要 APIServer 来帮助你做授权工作。

而在 Kubernetes 项目中，负责完成授权（Authorization）工作的机制，就是 RBAC：基于角色的访问控制（Role-Based Access Control）。

* 三个最基本的概念

  Role：角色，它其实是一组规则，定义了一组对 Kubernetes API 对象的操作权限。

  Subject：被作用者，既可以是“人”，也可以是“机器”，也可以是你在 Kubernetes 里定义的“用户”。

  RoleBinding：定义了“被作用者”和“角色”的绑定关系。

* Role

  实际上，Role 本身就是一个 Kubernetes 的 API 对象。

  ```yaml
  kind: Role
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    namespace: mynamespace
    name: example-role
  rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
  ```

  Namespace 是 Kubernetes 项目里的一个逻辑管理单位。不同 Namespace 的 API 对象，在通过 kubectl 命令进行操作的时候，是互相隔离开的。

  当然，这仅限于逻辑上的“隔离”，Namespace 并不会提供任何实际的隔离或者多租户能力。

  这个 Role 对象的 rules 字段，就是它所定义的权限规则。在上面的例子里，这条规则的含义就是：允许“被作用者”，对 mynamespace 下面的 Pod 对象，进行 GET、WATCH 和 LIST 操作。

* RoleBinding

  ```yaml
  kind: RoleBinding
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: example-rolebinding
    namespace: mynamespace
  subjects:
  - kind: User
    name: example-user
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: example-role
    apiGroup: rbac.authorization.k8s.io
  ```

  实际上，Kubernetes 里的“User”，也就是“用户”，只是一个授权系统里的逻辑概念。它需要通过外部认证服务，比如 Keystone 来提供。或者，你也可以直接给 APIServer 指定一个用户名、密码文件。那么 Kubernetes 的授权系统，就能够从这个文件里找到对应的“用户”了。当然，在大多数私有的使用环境中，我们只要使用 Kubernetes 提供的内置“用户”，就足够了。

  接下来，我们会看到一个 roleRef 字段。正是通过这个字段，RoleBinding 对象就可以直接通过名字，来引用我们前面定义的 Role 对象（example-role），从而定义了“被作用者（Subject）”和“角色（Role）”之间的绑定关系。

  需要再次提醒的是，Role 和 RoleBinding 对象都是 Namespaced 对象（Namespaced Object），它们对权限的限制规则仅在它们自己的 Namespace 内有效，roleRef 也只能引用当前 Namespace 里的 Role 对象。

  那么，对于非 Namespaced（Non-namespaced）对象（比如：Node），或者，某一个 Role 想要作用于所有的 Namespace 的时候，我们又该如何去做授权呢？这时候，我们就必须要使用 ClusterRole 和 ClusterRoleBinding 这两个组合了。这两个 API 对象的用法跟 Role 和 RoleBinding 完全一样。只不过，它们的定义里，没有了 Namespace 字段，如下所示：

  ```yaml
  kind: ClusterRole
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: example-clusterrole
  rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
  ```

  ```yaml
  kind: ClusterRoleBinding
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: example-clusterrolebinding
  subjects:
  - kind: User
    name: example-user
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: ClusterRole
    name: example-clusterrole
    apiGroup: rbac.authorization.k8s.io
  ```

  上面的例子里的 ClusterRole 和 ClusterRoleBinding 的组合，意味着名叫 example-user 的用户，拥有对所有 Namespace 里的 Pod 进行 GET、WATCH 和 LIST 操作的权限。

  更进一步地，在 Role 或者 ClusterRole 里面，如果要赋予用户 example-user 所有权限，那你就可以给它指定一个 verbs 字段的全集，如下所示：

  ```yaml
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
  ```

  类似地，Role 对象的 rules 字段也可以进一步细化。比如，你可以只针对某一个具体的对象进行权限设置，如下所示：

  ```yaml
  rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["my-config"]
    verbs: ["get"]
  ```

  这个例子就表示，这条规则的“被作用者”，只对名叫“my-config”的 ConfigMap 对象，有进行 GET 操作的权限。

* ServiceAccount 分配权限

  首先，我们要定义一个 ServiceAccount。

  ```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    namespace: mynamespace
    name: example-sa
  ```

  然后，我们通过编写 RoleBinding 的 YAML 文件，来为这个 ServiceAccount 分配权限：

  ```yaml
  kind: RoleBinding
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    name: example-rolebinding
    namespace: mynamespace
  subjects:
  - kind: ServiceAccount
    name: example-sa
    namespace: mynamespace
  roleRef:
    kind: Role
    name: example-role
    apiGroup: rbac.authorization.k8s.io
  ```

  接着，我们用 kubectl 命令创建这三个对象。

  然后，我们来查看一下这个 ServiceAccount 的详细信息：

  ```shell
  kubectl get sa -n mynamespace -o yaml
  ```

  可以看到，Kubernetes 会为一个 ServiceAccount 自动创建并分配一个 Secret 对象，即：上述 ServiceAcount 定义里最下面的 secrets 字段。

  这个 Secret，就是这个 ServiceAccount 对应的、用来跟 APIServer 进行交互的授权文件，我们一般称它为：Token。Token 文件的内容一般是证书或者密码，它以一个 Secret 对象的方式保存在 Etcd 当中。

  这时候，用户的 Pod，就可以声明使用这个 ServiceAccount 了，比如下面这个例子：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    namespace: mynamespace
    name: sa-token-test
  spec:
    containers:
    - name: nginx
      image: nginx:1.7.9
    serviceAccountName: example-sa
  ```

  等这个 Pod 运行起来之后，我们就可以看到，该 ServiceAccount 的 token，也就是一个 Secret 对象，被 Kubernetes 自动挂载到了容器的 /var/run/secrets/kubernetes.io/serviceaccount 目录下，如下所示：

  ```shell
  kubectl describe pod sa-token-test -n mynamespace
  ```

  这时候，我们可以通过 kubectl exec 查看到这个目录里的文件：

  ```shell
  kubectl exec -it sa-token-test -n mynamespace -- /bin/bash
  root@sa-token-test:/# ls /var/run/secrets/kubernetes.io/serviceaccount
  ca.crt  namespace  token
  ```

  如上所示，容器里的应用，就可以使用这个 ca.crt 来访问 APIServer 了。更重要的是，此时它只能够做 GET、WATCH 和 LIST 操作。因为 example-sa 这个 ServiceAccount 的权限，已经被我们绑定了 Role 做了限制。

  如果一个 Pod 没有声明 serviceAccountName，Kubernetes 会自动在它的 Namespace 下创建一个名叫 default 的默认 ServiceAccount，然后分配给这个 Pod。

  但在这种情况下，这个默认 ServiceAccount 并没有关联任何 Role。也就是说，此时它有访问 APIServer 的绝大多数权限。当然，这个访问所需要的 Token，还是默认 ServiceAccount 对应的 Secret 对象为它提供的。

  所以，在生产环境中，我强烈建议你为所有 Namespace 下的默认 ServiceAccount，绑定一个只读权限的 Role。

* “用户组”

  除了前面使用的“用户”（User），Kubernetes 还拥有“用户组”（Group）的概念，也就是一组“用户”的意思。如果你为 Kubernetes 配置了外部认证服务的话，这个“用户组”的概念就会由外部认证服务提供。

  而对于 Kubernetes 的内置“用户” ServiceAccount 来说，上述“用户组”的概念也同样适用。

  实际上，一个 ServiceAccount，在 Kubernetes 里对应的“用户”的名字是：`system:serviceaccount:<Namespace 名字>:<ServiceAccount 名字>`。

  而它对应的内置“用户组”的名字，就是：`system:serviceaccounts:<Namespace 名字>`。

  现在我们可以在 RoleBinding 里定义如下的 subjects：

  ```yaml
  subjects:
  - kind: Group
    name: system:serviceaccounts:mynamespace
    apiGroup: rbac.authorization.k8s.io
  ```

  这就意味着这个 Role 的权限规则，作用于 mynamespace 里的所有 ServiceAccount。

  而下面这个例子：

  ```yaml
  subjects:
  - kind: Group
    name: system:serviceaccounts
    apiGroup: rbac.authorization.k8s.io
  ```

  就意味着这个 Role 的权限规则，作用于整个系统里的所有 ServiceAccount。

* 内置 ClusterRole

  最后，值得一提的是，在 Kubernetes 中已经内置了很多个为系统保留的 ClusterRole，它们的名字都以 system: 开头。你可以通过 kubectl get clusterroles 查看到它们。

  一般来说，这些系统 ClusterRole，是绑定给 Kubernetes 系统组件对应的 ServiceAccount 使用的。

  除此之外，Kubernetes 还提供了四个预先定义好的 ClusterRole 来供用户直接使用：cluster-admin；admin；edit；view。

### 27 | 聪明的微创新：Operator 工作原理解读

* Etcd Operator 的使用

* Operator

  Operator 的工作原理，实际上是利用了 Kubernetes 的自定义 API 资源（CRD），来描述我们想要部署的“有状态应用”；然后在自定义控制器里，根据自定义 API 对象的变化，来完成具体的部署和运维工作。

* Etcd 集群

  Etcd Operator 部署 Etcd 集群，采用的是静态集群（Static）的方式。

  静态集群的好处是，它不必依赖于一个额外的服务发现机制来组建集群，非常适合本地容器化部署。而它的难点，则在于你必须在部署的时候，就规划好这个集群的拓扑结构，并且能够知道这些节点固定的 IP 地址。

  ```shell
  etcd --name infra2 --initial-advertise-peer-urls http://10.0.1.12:2380 \
  --listen-peer-urls http://10.0.1.12:2380 \
  ...
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
  --initial-cluster-state new
  ```

  其中，这些节点启动参数里的 --initial-cluster 参数，非常值得你关注。它的含义，正是当前节点启动时集群的拓扑结构。说得更详细一点，就是当前这个节点启动时，需要跟哪些节点通信来组成集群。

  此外，一个 Etcd 集群还需要用 --initial-cluster-token 字段，来声明一个该集群独一无二的 Token 名字。

  像上述这样为每一个 Ectd 节点配置好它对应的启动参数之后把它们启动起来，一个 Etcd 集群就可以自动组建起来了。

  而我们要编写的 Etcd Operator，就是要把上述过程自动化。这其实等同于：用代码来生成每个 Etcd 节点 Pod 的启动命令，然后把它们启动起来。

* Etcd Operator 部署 Etcd 集群

  Etcd Operator 的实现，虽然选择的也是静态集群，但这个集群具体的组建过程，是逐个节点动态添加的方式，即：

  首先，Etcd Operator 会创建一个“种子节点”；

  然后，Etcd Operator 会不断创建新的 Etcd 节点，然后将它们逐一加入到这个集群当中，直到集群的节点数等于 size。

  这就意味着，在生成不同角色的 Etcd Pod 时，Operator 需要能够区分种子节点与普通节点。

  而这两种节点的不同之处，就在于一个名叫 --initial-cluster-state 的启动参数：

  当这个参数值设为 new 时，就代表了该节点是种子节点。而我们前面提到过，种子节点还必须通过 --initial-cluster-token 声明一个独一无二的 Token。

  而如果这个参数值设为 existing，那就是说明这个节点是一个普通节点，Etcd Operator 需要把它加入到已有集群里。

  那么接下来的问题就是，每个 Etcd 节点的 --initial-cluster 字段的值又是怎么生成的呢？

  由于这个方案要求种子节点先启动，所以对于种子节点 infra0 来说，它启动后的集群只有它自己，即：`--initial-cluster=infra0=http://10.0.1.10:2380`。

  而对于接下来要加入的节点，比如 infra1 来说，它启动后的集群就有两个节点了，所以它的 --initial-cluster 参数的值应该是：`infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380`。

  现在，你就应该能在脑海中构思出上述三节点 Etcd 集群的部署过程了。

  首先，只要用户提交 YAML 文件时声明创建一个 EtcdCluster 对象（一个 Etcd 集群），那么 Etcd Operator 都应该先创建一个单节点的种子集群（Seed Member），并启动这个种子节点。以 infra0 节点为例，它的 IP 地址是 10.0.1.10，那么 Etcd Operator 生成的种子节点的启动命令，如下所示：

  ```shell
  $ etcd
    --data-dir=/var/etcd/data
    --name=infra0
    --initial-advertise-peer-urls=http://10.0.1.10:2380
    --listen-peer-urls=http://0.0.0.0:2380
    --listen-client-urls=http://0.0.0.0:2379
    --advertise-client-urls=http://10.0.1.10:2379
    --initial-cluster=infra0=http://10.0.1.10:2380
    --initial-cluster-state=new
    --initial-cluster-token=4b5215fa-5401-4a95-a8c6-892317c9bef8
  ```

  可以看到，这个种子节点的 initial-cluster-state 是 new，并且指定了唯一的 initial-cluster-token 参数。我们可以把这个创建种子节点（集群）的阶段称为：Bootstrap。

  接下来，对于其他每一个节点，Operator 只需要执行如下两个操作即可，以 infra1 为例。

  第一步，通过 Etcd 命令行添加一个新成员：

  ```shell
  $ etcdctl member add infra1 http://10.0.1.11:2380
  ```

  第二步，为这个成员节点生成对应的启动参数，并启动它：

  ```shell
  $ etcd
    --data-dir=/var/etcd/data
    --name=infra1
    --initial-advertise-peer-urls=http://10.0.1.11:2380
    --listen-peer-urls=http://0.0.0.0:2380
    --listen-client-urls=http://0.0.0.0:2379
    --advertise-client-urls=http://10.0.1.11:2379
    --initial-cluster=infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380
    --initial-cluster-state=existing
  ```

  可以看到，对于这个 infra1 成员节点来说，它的 initial-cluster-state 是 existing，也就是要加入已有集群。而它的 initial-cluster 的值，则变成了 infra0 和 infra1 两个节点的 IP 地址。

* Etcd Operator 工作原理

  Etcd Operator 启动要做的第一件事（c.initResource），是创建 EtcdCluster 对象所需要的 CRD，即前面提到的 etcdclusters.etcd.database.coreos.com。这样 Kubernetes 就能够“认识” EtcdCluster 这个自定义 API 资源了。

  而接下来，Etcd Operator 会定义一个 EtcdCluster 对象的 Informer。

  Etcd Operator 并没有用工作队列来协调 Informer 和控制循环。

  具体来讲，我们在控制循环里执行的业务逻辑，往往是比较耗时间的。比如，创建一个真实的 Etcd 集群。而 Informer 的 WATCH 机制对 API 对象变化的响应，则非常迅速。所以，控制器里的业务逻辑就很可能会拖慢 Informer 的执行周期，甚至可能 Block 它。而要协调这样两个快、慢任务的一个典型解决方法，就是引入一个工作队列。

  由于 Etcd Operator 里没有工作队列，那么在它的 EventHandler 部分，就不会有什么入队操作，而直接就是每种事件对应的具体的业务逻辑了。

  Etcd Operator 的特殊之处在于，它为每一个 EtcdCluster 对象，都启动了一个控制循环，“并发”地响应这些对象的变化。

  当这个 YAML 文件第一次被提交到 Kubernetes 之后，Etcd Operator 的 Informer 就会立刻“感知”到一个新的 EtcdCluster 对象被创建了出来。所以，EventHandler 里的“添加”事件会被触发。

  而这个 Handler 要做的操作也很简单，即：在 Etcd Operator 内部创建一个对应的 Cluster 对象（cluster.New）。

  这个 Cluster 对象，就是一个 Etcd 集群在 Operator 内部的描述，所以它与真实的 Etcd 集群的生命周期是一致的。

  而一个 Cluster 对象需要具体负责的，其实有两个工作。

  其中，第一个工作只在该 Cluster 对象第一次被创建的时候才会执行。这个工作，就是我们前面提到过的 Bootstrap，即：创建一个单节点的种子集群。

  由于种子集群只有一个节点，所以这一步直接就会生成一个 Etcd 的 Pod 对象。这个 Pod 里有一个 InitContainer，负责检查 Pod 的 DNS 记录是否正常。如果检查通过，用户容器也就是 Etcd 容器就会启动起来。

  可以看到，在这些启动参数（比如：initial-cluster）里，Etcd Operator 只会使用 Pod 的 DNS 记录，而不是它的 IP 地址。

  这当然是因为，在 Operator 生成上述启动命令的时候，Etcd 的 Pod 还没有被创建出来，它的 IP 地址自然也无从谈起。

  这也就意味着，每个 Cluster 对象，都会事先创建一个与该 EtcdCluster 同名的 Headless Service。这样，Etcd Operator 在接下来的所有创建 Pod 的步骤里，就都可以使用 Pod 的 DNS 记录来代替它的 IP 地址了。

  Cluster 对象的第二个工作，则是启动该集群所对应的控制循环。

  这个控制循环每隔一定时间，就会执行一次下面的 Diff 流程。

  首先，控制循环要获取到所有正在运行的、属于这个 Cluster 的 Pod 数量，也就是该 Etcd 集群的“实际状态”。

  而这个 Etcd 集群的“期望状态”，正是用户在 EtcdCluster 对象里定义的 size。

  所以接下来，控制循环会对比这两个状态的差异。

  如果实际的 Pod 数量不够，那么控制循环就会执行一个添加成员节点的操作；反之，就执行删除成员节点的操作。

* StatefulSet vs Operator

  第一个问题是，在 StatefulSet 里，它为 Pod 创建的名字是带编号的，这样就把整个集群的拓扑状态固定了下来（比如：一个三节点的集群一定是由名叫 web-0、web-1 和 web-2 的三个 Pod 组成）。可是，在 Etcd Operator 里，为什么我们使用随机名字就可以了呢？

  这是因为，Etcd Operator 在每次添加 Etcd 节点的时候，都会先执行 etcdctl member add；每次删除节点的时候，则会执行 etcdctl member remove 。这些操作，其实就会更新 Etcd 内部维护的拓扑信息，所以 Etcd Operator 无需在集群外部通过编号来固定这个拓扑关系。

  第二个问题是，为什么我没有在 EtcdCluster 对象里声明 Persistent Volume？

  我们知道，Etcd 是一个基于 Raft 协议实现的高可用 Key-Value 存储。根据 Raft 协议的设计原则，当 Etcd 集群里只有半数以下的节点失效时，当前集群依然可以正常工作。此时，Etcd Operator 只需要通过控制循环创建出新的 Pod，然后将它们加入到现有集群里，就完成了“期望状态”与“实际状态”的调谐工作。这个集群，是一直可用的 。

  但是，当这个 Etcd 集群里有半数以上的节点失效的时候，这个集群就会丧失数据写入的能力，从而进入“不可用”状态。此时，即使 Etcd Operator 创建出新的 Pod 出来，Etcd 集群本身也无法自动恢复起来。

  这个时候，我们就必须使用 Etcd 本身的备份数据来对集群进行恢复操作。

  在有了 Operator 机制之后，上述 Etcd 的备份操作，是由一个单独的 Etcd Backup Operator 负责完成的。

### 28 | PV、PVC、StorageClass 这些到底在说啥？

* PV、PVC

  PV 描述的，是持久化存储数据卷。这个 API 对象主要定义的是一个持久化存储在宿主机上的目录，比如一个 NFS 的挂载目录。

  通常情况下，PV 对象是由运维人员事先创建在 Kubernetes 集群里待用的。

  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: nfs
  spec:
    storageClassName: manual
    capacity:
      storage: 1Gi
    accessModes:
    - ReadWriteMany
    nfs:
      server: 10.244.1.4
      path: "/"
  ```

  而 PVC 描述的，则是 Pod 所希望使用的持久化存储的属性。比如，Volume 存储的大小、可读写权限等等。

  PVC 对象通常由开发人员创建；或者以 PVC 模板的方式成为 StatefulSet 的一部分，然后由 StatefulSet 控制器负责创建带编号的 PVC。

  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: nfs
  spec:
    accessModes:
    - ReadWriteMany
    storageClassName: manual
    resources:
      requests:
        storage: 1Gi
  ```

  而用户创建的 PVC 要真正被容器使用起来，就必须先和某个符合条件的 PV 进行绑定。这里要检查的条件，包括两部分：

  第一个条件，当然是 PV 和 PVC 的 spec 字段。比如，PV 的存储（storage）大小，就必须满足 PVC 的要求。

  而第二个条件，则是 PV 和 PVC 的 storageClassName 字段必须一样。

  在成功地将 PVC 和 PV 进行绑定之后，Pod 就能够像使用 hostPath 等常规类型的 Volume 一样，在自己的 YAML 文件里声明使用这个 PVC 了，如下所示：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      role: web-frontend
  spec:
    containers:
    - name: web
      image: nginx
      ports:
      - name: web
        containerPort: 80
      volumeMounts:
      - name: nfs
        mountPath: "/usr/share/nginx/html"
    volumes:
    - name: nfs
      persistentVolumeClaim:
        claimName: nfs
  ```

  可以看到，Pod 需要做的，就是在 volumes 字段里声明自己要使用的 PVC 名字。接下来，等这个 Pod 创建之后，kubelet 就会把这个 PVC 所对应的 PV，也就是一个 NFS 类型的 Volume，挂载在这个 Pod 容器内的目录上。

  不难看出，PVC 和 PV 的设计，其实跟“面向对象”的思想完全一致。

  PVC 可以理解为持久化存储的“接口”，它提供了对某种持久化存储的描述，但不提供具体的实现；而这个持久化存储的实现部分则由 PV 负责完成。

* PersistentVolumeController

  在 Kubernetes 中，实际上存在着一个专门处理持久化存储的控制器，叫作 Volume Controller。这个 Volume Controller 维护着多个控制循环，其中有一个循环，扮演的就是撮合 PV 和 PVC 的“红娘”的角色。它的名字叫作 PersistentVolumeController。PersistentVolumeController 会不断地查看当前每一个 PVC，是不是已经处于 Bound（已绑定）状态。如果不是，那它就会遍历所有的、可用的 PV，并尝试将其与这个“单身”的 PVC 进行绑定。这样，Kubernetes 就可以保证用户提交的每一个 PVC，只要有合适的 PV 出现，它就能够很快进入绑定状态，从而结束“单身”之旅。

  而所谓将一个 PV 与 PVC 进行“绑定”，其实就是将这个 PV 对象的名字，填在了 PVC 对象的 spec.volumeName 字段上。

* 持久化存储

  所谓容器的 Volume，其实就是将一个宿主机上的目录，跟一个容器里的目录绑定挂载在了一起。

  而所谓的“持久化 Volume”，指的就是这个宿主机上的目录，具备“持久性”。即：这个目录里面的内容，既不会因为容器的删除而被清理掉，也不会跟当前的宿主机绑定。这样，当容器被重启或者在其他节点上重建出来之后，它仍然能够通过挂载这个 Volume，访问到这些内容。

  显然，我们前面使用的 hostPath 和 emptyDir 类型的 Volume 并不具备这个特征：它们既有可能被 kubelet 清理掉，也不能被“迁移”到其他节点上。

  所以，大多数情况下，持久化 Volume 的实现，往往依赖于一个远程存储服务，比如：远程文件存储（比如，NFS、GlusterFS）、远程块存储（比如，公有云提供的远程磁盘）等等。

  而 Kubernetes 需要做的工作，就是使用这些存储服务，来为容器准备一个持久化的宿主机目录，以供将来进行绑定挂载时使用。而所谓“持久化”，指的是容器在这个目录里写入的文件，都会保存在远程存储中，从而使得这个目录具备了“持久性”。

* PV “两阶段处理”流程

  当一个 Pod 调度到一个节点上之后，kubelet 就要负责为这个 Pod 创建它的 Volume 目录。默认情况下，kubelet 为 Volume 创建的目录是如下所示的一个宿主机上的路径：

  `/var/lib/kubelet/pods/<Pod 的 ID>/volumes/kubernetes.io~<Volume 类型>/<Volume 名字>`

  接下来，kubelet 要做的操作就取决于你的 Volume 类型了。

  如果你的 Volume 类型是远程块存储，比如 Google Cloud 的 Persistent Disk（GCE 提供的远程磁盘服务），那么 kubelet 就需要先调用 Goolge Cloud 的 API，将它所提供的 Persistent Disk 挂载到 Pod 所在的宿主机上。

  这相当于执行：

  `$ gcloud compute instances attach-disk <虚拟机名字> --disk <远程磁盘名字>`

  这一步为虚拟机挂载远程磁盘的操作，对应的正是“两阶段处理”的第一阶段。在 Kubernetes 中，我们把这个阶段称为 Attach。

  Attach 阶段完成后，为了能够使用这个远程磁盘，kubelet 还要进行第二个操作，即：格式化这个磁盘设备，然后将它挂载到宿主机指定的挂载点上。不难理解，这个挂载点，正是我在前面反复提到的 Volume 的宿主机目录。所以，这一步相当于执行：

  ```shell
  # 通过 lsblk 命令获取磁盘设备 ID
  $ sudo lsblk
  # 格式化成 ext4 格式
  $ sudo mkfs.ext4 -m 0 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/<磁盘设备 ID>
  # 挂载到挂载点
  $ sudo mkdir -p /var/lib/kubelet/pods/<Pod 的 ID>/volumes/kubernetes.io~<Volume 类型>/<Volume 名字>
  ```

  这个将磁盘设备格式化并挂载到 Volume 宿主机目录的操作，对应的正是“两阶段处理”的第二个阶段，我们一般称为：Mount。

  Mount 阶段完成后，这个 Volume 的宿主机目录就是一个“持久化”的目录了，容器在它里面写入的内容，会保存在 Google Cloud 的远程磁盘中。

  而如果你的 Volume 类型是远程文件存储（比如 NFS）的话，kubelet 的处理过程就会更简单一些。

  因为在这种情况下，kubelet 可以跳过“第一阶段”（Attach）的操作，这是因为一般来说，远程文件存储并没有一个“存储设备”需要挂载在宿主机上。

  所以，kubelet 会直接从“第二阶段”（Mount）开始准备宿主机上的 Volume 目录。

  在这一步，kubelet 需要作为 client，将远端 NFS 服务器的目录（比如：“/”目录），挂载到 Volume 的宿主机目录上，即相当于执行如下所示的命令：

  `$ mount -t nfs <NFS 服务器地址>:/ /var/lib/kubelet/pods/<Pod 的 ID>/volumes/kubernetes.io~<Volume 类型>/<Volume 名字> `

  通过这个挂载操作，Volume 的宿主机目录就成为了一个远程 NFS 目录的挂载点，后面你在这个目录里写入的所有文件，都会被保存在远程 NFS 服务器上。所以，我们也就完成了对这个 Volume 宿主机目录的“持久化”。

  而经过了“两阶段处理”，我们就得到了一个“持久化”的 Volume 宿主机目录。所以，接下来，kubelet 只要把这个 Volume 目录通过 CRI 里的 Mounts 参数，传递给 Docker，然后就可以为 Pod 里的容器挂载这个“持久化”的 Volume 了。其实，这一步相当于执行了如下所示的命令：

  `$ docker run -v /var/lib/kubelet/pods/<Pod 的 ID>/volumes/kubernetes.io~<Volume 类型>/<Volume 名字>:/<容器内的目标目录> 我的镜像 ...`

* 控制循环

  在 Kubernetes 中，上述关于 PV 的“两阶段处理”流程，是靠独立于 kubelet 主控制循环（Kubelet Sync Loop）之外的两个控制循环来实现的。

  其中，“第一阶段”的 Attach（以及 Dettach）操作，是由 Volume Controller 负责维护的，这个控制循环的名字叫作：AttachDetachController。而它的作用，就是不断地检查每一个 Pod 对应的 PV，和这个 Pod 所在宿主机之间挂载情况。从而决定，是否需要对这个 PV 进行 Attach（或者 Dettach）操作。

  需要注意，作为一个 Kubernetes 内置的控制器，Volume Controller 自然是 kube-controller-manager 的一部分。所以，AttachDetachController 也一定是运行在 Master 节点上的。当然，Attach 操作只需要调用公有云或者具体存储项目的 API，并不需要在具体的宿主机上执行操作，所以这个设计没有任何问题。

  而“第二阶段”的 Mount（以及 Unmount）操作，必须发生在 Pod 对应的宿主机上，所以它必须是 kubelet 组件的一部分。这个控制循环的名字，叫作：VolumeManagerReconciler，它运行起来之后，是一个独立于 kubelet 主循环的 Goroutine。

  通过这样将 Volume 的处理同 kubelet 的主循环解耦，Kubernetes 就避免了这些耗时的远程挂载操作拖慢 kubelet 的主控制循环，进而导致 Pod 的创建效率大幅下降的问题。实际上，kubelet 的一个主要设计原则，就是它的主控制循环绝对不可以被 block。

* StorageClass

  一个大规模的 Kubernetes 集群里很可能有成千上万个 PVC，这就意味着运维人员必须得事先创建出成千上万个 PV。更麻烦的是，随着新的 PVC 不断被提交，运维人员就不得不继续添加新的、能满足条件的 PV，否则新的 Pod 就会因为 PVC 绑定不到 PV 而失败。在实际操作中，这几乎没办法靠人工做到。

  所以，Kubernetes 为我们提供了一套可以自动创建 PV 的机制，即：Dynamic Provisioning。相比之下，前面人工管理 PV 的方式就叫作 Static Provisioning。

  Dynamic Provisioning 机制工作的核心，在于一个名叫 StorageClass 的 API 对象。

  而 StorageClass 对象的作用，其实就是创建 PV 的模板。

  具体地说，StorageClass 对象会定义如下两个部分内容：

  第一，PV 的属性。比如，存储类型、Volume 的大小等等。

  第二，创建这种 PV 需要用到的存储插件。比如，Ceph 等等。

  有了这样两个信息之后，Kubernetes 就能够根据用户提交的 PVC，找到一个对应的 StorageClass 了。然后，Kubernetes 就会调用该 StorageClass 声明的存储插件，创建出需要的 PV。

  假如我们的 Volume 的类型是 GCE 的 Persistent Disk 的话，运维人员就需要定义一个如下所示的 StorageClass：

  ```yaml
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: block-service
  provisioner: kubernetes.io/gce-pd
  parameters:
    type: pd-ssd
  ```

  这时候，作为应用开发者，我们只需要在 PVC 里指定要使用的 StorageClass 名字即可，如下所示：

  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: claim
  spec:
    accessModes:
    - ReadWriteOnce
    storageClassName: block-service
    resources:
      requests:
        storage: 30Gi
  ```

  有了 Dynamic Provisioning 机制，运维人员只需要在 Kubernetes 集群里创建出数量有限的 StorageClass 对象就可以了。这就好比，运维人员在 Kubernetes 集群里创建出了各种各样的 PV 模板。这时候，当开发人员提交了包含 StorageClass 字段的 PVC 之后，Kubernetes 就会根据这个 StorageClass 创建出对应的 PV。

### 29 | PV、PVC 体系是不是多此一举？从本地持久化卷谈起

* Local Persistent Volume 适用范围

  首先需要明确的是，Local Persistent Volume 并不适用于所有应用。事实上，它的适用范围非常固定，比如：高优先级的系统应用，需要在多个不同节点上存储数据，并且对 I/O 较为敏感。典型的应用包括：分布式数据存储比如 MongoDB、Cassandra 等，分布式文件系统比如 GlusterFS、Ceph 等，以及需要在本地磁盘上进行大量数据缓存的分布式应用。

  其次，相比于正常的 PV，一旦这些节点宕机且不能恢复时，Local Persistent Volume 的数据就可能丢失。这就要求使用 Local Persistent Volume 的应用必须具备数据备份和恢复的能力，允许你把这些数据定时备份在其他位置。

* Local Persistent Volume 设计难点

  第一个难点在于：如何把本地磁盘抽象成 PV。

  可能你会说，Local Persistent Volume 不就等同于 hostPath 加 NodeAffinity 吗？

  比如，一个 Pod 可以声明使用类型为 Local 的 PV，而这个 PV 其实就是一个 hostPath 类型的 Volume。如果这个 hostPath 对应的目录，已经在节点 A 上被事先创建好了。那么，我只需要再给这个 Pod 加上一个 nodeAffinity=nodeA，不就可以使用这个 Volume 了吗？

  事实上，你绝不应该把一个宿主机上的目录当作 PV 使用。这是因为，这种本地目录的存储行为完全不可控，它所在的磁盘随时都可能被应用写满，甚至造成整个宿主机宕机。而且，不同的本地目录之间也缺乏哪怕最基础的 I/O 隔离机制。

  一个 Local Persistent Volume 对应的存储介质，一定是一块额外挂载在宿主机的磁盘或者块设备（“额外”的意思是，它不应该是宿主机根目录所使用的主硬盘）。这个原则，我们可以称为“一个 PV 一块盘”。

  第二个难点在于：调度器如何保证 Pod 始终能被正确地调度到它所请求的 Local Persistent Volume 所在的节点上呢？

  造成这个问题的原因在于，对于常规的 PV 来说，Kubernetes 都是先调度 Pod 到某个节点上，然后，再通过“两阶段处理”来“持久化”这台机器上的 Volume 目录，进而完成 Volume 目录与容器的绑定挂载。

  可是，对于 Local PV 来说，节点上可供使用的磁盘（或者块设备），必须是运维人员提前准备好的。它们在不同节点上的挂载情况可以完全不同，甚至有的节点可以没这种磁盘。

  所以，这时候，调度器就必须能够知道所有节点与 Local Persistent Volume 对应的磁盘的关联关系，然后根据这个信息来调度 Pod。

  这个原则，我们可以称为“在调度的时候考虑 Volume 分布”。在 Kubernetes 的调度器里，有一个叫作 VolumeBindingChecker 的过滤条件专门负责这个事情。

  基于上述讲述，在开始使用 Local Persistent Volume 之前，你首先需要在集群里配置好磁盘或者块设备。在公有云上，这个操作等同于给虚拟机额外挂载一个磁盘，比如 GCE 的 Local SSD 类型的磁盘就是一个典型例子。

* Local Persistent Volume 实践

  在我们部署的私有环境中，你有两种办法来完成这个步骤：

  第一种，当然就是给你的宿主机挂载并格式化一个可用的本地磁盘，这也是最常规的操作；

  第二种，对于实验环境，你其实可以在宿主机上挂载几个 RAM Disk（内存盘）来模拟本地磁盘。

  首先，在名叫 node-1 的宿主机上创建一个挂载点，比如 /mnt/disks；然后，用几个 RAM Disk 来模拟本地磁盘，如下所示：

  ```yaml
  # 在 node-1 上执行
  $ mkdir /mnt/disks
  $ for vol in vol1 vol2 vol3; do
      mkdir /mnt/disks/$vol
      mount -t tmpfs $vol /mnt/disks/$vol
  done
  ```

  接下来，我们就可以为这些本地磁盘定义对应的 PV 了，如下所示：

  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: example-pv
  spec:
    capacity:
      storage: 5Gi
    volumeMode: Filesystem
    accessModes:
    - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: local-storage
    local:
      path: /mnt/disks/vol1
    nodeAffinity:
      required:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - node-1
  ```

  在这个 PV 的定义里，需要有一个 nodeAffinity 字段指定 node-1 这个节点的名字。这样，调度器在调度 Pod 的时候，就能够知道一个 PV 与节点的对应关系，从而做出正确的选择。这正是 Kubernetes 实现“在调度的时候就考虑 Volume 分布”的主要方法。

  使用 PV 和 PVC 的最佳实践，是你要创建一个 StorageClass 来描述这个 PV，如下所示：

  ```yaml
  kind: StorageClass
  apiVersion: storage.k8s.io/v1
  metadata:
    name: local-storage
  provisioner: kubernetes.io/no-provisioner
  volumeBindingMode: WaitForFirstConsumer
  ```

  需要注意的是，在它的 provisioner 字段，我们指定的是 no-provisioner。这是因为 Local Persistent Volume 目前尚不支持 Dynamic Provisioning，所以它没办法在用户创建 PVC 的时候，就自动创建出对应的 PV。也就是说，我们前面创建 PV 的操作，是不可以省略的。

  与此同时，这个 StorageClass 还定义了一个 volumeBindingMode=WaitForFirstConsumer 的属性。它是 Local Persistent Volume 里一个非常重要的特性，即：延迟绑定。

  StorageClass 里的 volumeBindingMode=WaitForFirstConsumer 的含义，就是告诉 Kubernetes 里的 Volume 控制循环（“红娘”）：虽然你已经发现这个 StorageClass 关联的 PVC 与 PV 可以绑定在一起，但请不要现在就执行绑定操作（即：设置 PVC 的 VolumeName 字段）。

  而要等到第一个声明使用该 PVC 的 Pod 出现在调度器之后，调度器再综合考虑所有的调度规则，当然也包括每个 PV 所在的节点位置，来统一决定，这个 Pod 声明的 PVC，到底应该跟哪个 PV 进行绑定。

  所以，通过这个延迟绑定机制，原本实时发生的 PVC 和 PV 的绑定过程，就被延迟到了 Pod 第一次调度的时候在调度器中进行，从而保证了这个绑定结果不会影响 Pod 的正常调度。

  当然，在具体实现中，调度器实际上维护了一个与 Volume Controller 类似的控制循环，专门负责为那些声明了“延迟绑定”的 PV 和 PVC 进行绑定工作。

  通过这样的设计，这个额外的绑定操作，并不会拖慢调度器的性能。而当一个 Pod 的 PVC 尚未完成绑定时，调度器也不会等待，而是会直接把这个 Pod 重新放回到待调度队列，等到下一个调度周期再做处理。

  接下来，我们只需要定义一个非常普通的 PVC，就可以让 Pod 使用到上面定义好的 Local Persistent Volume 了，如下所示：

  ```yaml
  kind: PersistentVolumeClaim
  apiVersion: v1
  metadata:
    name: example-local-claim
  spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 5Gi
    storageClassName: local-storage
  ```

  然后，我们编写一个 Pod 来声明使用这个 PVC，如下所示：

  ```yaml
  kind: Pod
  apiVersion: v1
  metadata:
    name: example-pv-pod
  spec:
    volumes:
      - name: example-pv-storage
        persistentVolumeClaim:
          claimName: example-local-claim
    containers:
      - name: example-pv-container
        image: nginx
        ports:
          - containerPort: 80
            name: "http-server"
        volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: example-pv-storage
  ```

  我们一旦使用 kubectl create 创建这个 Pod，就会发现，我们前面定义的 PVC，会立刻变成 Bound 状态，与前面定义的 PV 绑定在了一起。

  也就是说，在我们创建的 Pod 进入调度器之后，“绑定”操作才开始进行。

  需要注意的是，我们上面手动创建 PV 的方式，即 Static 的 PV 管理方式，在删除 PV 时需要按如下流程执行操作：

  删除使用这个 PV 的 Pod；从宿主机移除本地磁盘（比如，umount 它）；删除 PVC；删除 PV。

* Static Provisioner

  由于上面这些创建 PV 和删除 PV 的操作比较繁琐，Kubernetes 其实提供了一个 Static Provisioner 来帮助你管理这些 PV。

  比如，我们现在的所有磁盘，都挂载在宿主机的 /mnt/disks 目录下。

  那么，当 Static Provisioner 启动后，它就会通过 DaemonSet，自动检查每个宿主机的 /mnt/disks 目录。然后，调用 Kubernetes API，为这些目录下面的每一个挂载，创建一个对应的 PV 对象出来。

  这个 PV 里的各种定义，比如 StorageClass 的名字、本地磁盘挂载点的位置，都可以通过 provisioner 的配置文件指定。当然，provisioner 也会负责前面提到的 PV 的删除工作。

