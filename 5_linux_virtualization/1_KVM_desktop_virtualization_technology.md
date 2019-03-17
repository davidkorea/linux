# 第一章 Linux 桌面虚拟化技术 KVM

1. 虚拟化产品对比介绍
2. 安装 KVM 虚拟机
3. 实戓 1:配置 KVM 网络桥接功能
4. 实戓 2:使用 KVM 安装虚拟机
5. 实戓 3:解决 centos6 下 shutdown 关丌了 KVM 虚拟机的问题

# 1. 虚拟化产品对比介绍
- 仿真虚拟化： [对系统硬件没有要求,性能最低] vmware
- 半虚拟化： [虚拟机可以使用真机物理硬件，性能高，需要改内核] xen,半虚 REHL5 自带 xen, 安装时需要安装内核 rpm -ivh kernel-xen-xxx.rpm 
- 全虚拟化： 直接使用物理硬件，性能高。kvm 全虚拟化 RHEL6 自带 kvm

  KVM 概述: KVM 即 Kernel-based Virtual Machine 基亍内核的虚拟机。 KVM，是一个开源的系统虚拟化模块，自 Linux 2.6.20 之后集成在 Linux 的各个主要发行版本中。它使用 Linux 自身的调度器进行管理，所以相对亍 Xen，其核心源码很少。KVM 目前已成为学术界的主 流 VMM(虚拟机监控器)之一。KVM 的虚拟化需要硬件支持(如 Intel VT 技术戒者 AMD V 技术)。是基 亍硬件的完全虚拟化。而 Xen 早期则是基亍软件模拟的 Para-Virtualization。

- KVM:是第一个整合到 Linux 内核的虚拟化技术。在 KVM 模型中，每一个虚拟机都是一个由 Linux 调度程序管理的标准进程，你可 以在用户空间启劢客户机操作系统。一个普通的 Linux 进程有两种运行模式:内核和用户。 KVM 增加了第三种模式:客户模式(有自己 的内核和用户模式)。KVM支持 linux 以外的其它系统。比如:windows
- XEN :需要升级内核，只能支持和物理机系统一样的操作系统。 
- QEMU:是一套由 Fabrice Bellard 所编写的以 GPL 许可证分发源码的模拟处理器，在 GNU/Linux 平台上使用广泛。QEMU 具有高速度和跨平台的特性，QEMU 能模拟至接近真实电脑的速度。QEMU 能模拟整个电脑系统，包括中央处理器及其他周边设备。 QEMU 和 vmware 一样都是支持仿真虚拟化，效率比较低。
