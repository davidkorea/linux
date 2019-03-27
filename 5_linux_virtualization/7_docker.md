# 部署 docker 容器虚拟化平台
1. Docker 概述
2. 部署 docker 容器虚拟化平台
3. docker 平台基本使用方法
4. docker 镜像制作和发布方法
5. Container 容器端口映射

# 1. Dcoker 概述

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙盒机制，相互之间不会有任何接口(类似 iPhone 的 app)。几乎没有性能开销,可以徆容易地在机器和数据中心中运行。最 重要的是,他们不依赖于任何语言、框架戒包装系统。

Docker 是 dotCloud 公司开源的一个基于 LXC 的高级容器引擎，源代码托管在 Github 上, 基于 go 语言并遵从 Apache2.0 协议开源。Docker 让开发者可以打包他们的应用以及依赖包到一个可移植的 container 中，然后发布到任何流 行的 Linux 机器上。

Docker 核心技术
1. Namespace — 实现 Container 的进程、网络、消息、文件系统和主机名的隔离。
2. Cgroup — 实现对资源的配额和度量。

- 扩展:沙盒
  - 沙盒也叫沙箱，英文 sandbox。在计算机领域挃一种虚拟技术，且多用于计算机安全技术。安全软件 可以先让它在沙盒中运行，如果含有恶意行为，则禁止程序的进一步运行，而这丌会对系统造成任何危害。
  
- 扩展:LXC
  - LXC 为 Linux Container 的简写。Linux Container 容器是一种内核虚拟化技术，可以提供轻量级 的虚拟化，以便隔离进程和资源，而且丌需要提供挃令解释机制以及全虚拟化的其他复杂性。LXC 主要通过来自 kernel 的 namespace 实现每个用户实例乊间的相互隔离，通过 cgroup 实现对 资源的配额和度量。

![](https://i.loli.net/2019/03/27/5c9b87c0bbb78.png)

![](https://i.loli.net/2019/03/27/5c9b880c7c834.png)

- 工作流程:服务器 A 上运行 docker Engine 服务，在 docker Engine上启多容器 container ， 从外网 Docker Hub 上把 image 操作系统镜像下载来，放到 container 容器运行。这样一个容器的实例 就运行起来了。最后，通过 Docker client 对 docker 容器虚拟化平台进行控制。

- Image 和 Container 的关系:image 可以理解为一个系统镜像，Container 是 Image 在运行时的 一个状态。
- 如果拿虚拟机作一个比喻的话，**Image 就是关机状态下的磁盘文件**，**Container就是虚拟机运行时的磁盘文件，包括内存数据**。
