# Xen basic

- Xen属于type-I型虚拟化，Xen hypervisor直接运行在硬件上
  - **其实Xen hypervisor就是被更改过的带有xen的kernel 直接作为宿主机操作系统安装在物理主机，同时也作为xen的dom0，来管理其他的domU虚拟机**
  - **hypervisor除了提供cpu和内存的虚拟化之外，还需要提供内存管理器，线程调度器，设备驱动，安全子系统，io栈，网络栈等组件。其实hypervisor本身就是一个完整的操作系统。只不过其主要目的是用来运行多个虚拟机而不是运行进程，这是和linux内核的主要区别。但是xen的hypervisor把io的能力托付给了dom0来处理**
- Xen仅进行CPU（包活中断）和内存的虚拟化，IO的虚拟化（声卡，显卡）由其第一个虚拟机domain0（dom0）的内核去实现
  - domain0 / dom0 / privileged domain，第一个虚拟机
  - domainU / unprivileged domain，其他虚拟机

- Xen直接运行在硬件上面，然后直接在xen上启动一个虚拟机，作为xen的亲密无间的战友，拥有管理其他的虚拟机的特权。其他虚拟机的启动和关闭都要通过第一个虚拟机进行实现。
- 因为xen本身并没有提供虚拟机管理程序，需要其上面第一个虚拟机的用户空间中运行虚拟机管理工具栈
- 因为xen本身不能驱动IO硬件，需要其上面第一个虚拟机内核空间安装目前流行的linux发行版。因为这些linux系统支持IO驱动。声卡，显卡，网卡，硬盘等设备
- 所以其他的虚拟机需要CPU或内存时，向xen hypervisor去请求调用
- 其他的虚拟机当使用io时，向第一个虚拟机发起调用请求。这种io的模拟是通过QEMU来实现的

- 每一个虚拟机中的CPU，映射到xen hypervisor中是一个一个的线程来提供
- 每一个虚拟机中的内存，都是由内存中的分页page管理。将物理内存中的一段空间切割出来，形成一个虚拟空间给虚拟机使用
  - 虽然虚拟机中看到的是连续的内存空间，而实际上在物理内存空间中可能是非连续的空间

- 当多个虚拟机发起调用时，dom0孰先孰后进行处理？先来先得，又一个环状缓冲，每一个请求过来，会占用一个槽位，当槽位全部占满时，即缓冲区占满。新io请求出现时，会提示io设备繁忙，以此来控制io设备请求速度。网络设备也是类似法则进行实现


## Xen当组成部分
### 1. Xen hypervisor
分配CPU，memory，interrupt
### 2. dom0 特权域
- IO分配
  - 网络设备
    net-front（guestos），net-backend
  - 块设备
    block-front（guestos），block-backend
- linux kernel
  - 2.6.37开始支持运行dom0
  - 3.0 对关键特性进行了优化
- 提供管理domU对工具栈，用于实现对虚拟机对添加，启动，快照，停止，删除等操作
- XenStore
  - 为各个domU提供共享信息存储空间，有着层级结构的名称空间。通常用于控制domU中的设备机制。配置中不需要设置xenstore
### 3. domU 非特权域
根据其虚拟化等实现方式有多种类型  PV，  HVM，   PV on HVM

domU中的虚拟机类型：
- Xen的PV技术：半虚拟化
  - 不依赖于CPU的硬件辅助特性，guestos内核向Xen hypervisor发起hyper call进行cpu/内存调用，要求guestsos的内核作出修改，以知晓自己运行于pv/半虚拟化环境
  - 对于io设备，guestos中需要能驱动io设备的fromtend，dom0中需要可以驱动io设别的backend
  - 运行于domU中的os：linux（2.6.24+）NetBSD，FreeBSD，OP en solosolars
  
<p align="center">
  <img src="https://i.loli.net/2019/04/06/5ca85a04348bc.png" height=350 weight=350 >
</p>  

- Xen的HVM技术：全虚拟化
  - 依赖于intel-VT，AMD-V，不需要guestos的内核向Xen hypervisor发起hyper call进行调用，可以直接运行指令。但仍然由Xen hypervisor将cpu和内存虚拟后进行提供
  - 依赖于QEMU来虚拟io设备
  - 运行于domU中的os：几乎所有支持x86平台的系统，当然也包括windows
  
<p align="center">
  <img src="https://i.loli.net/2019/04/06/5ca85ecbdbff7.png" height=350 weight=350 >
</p>  

- PV on HVM
  - CPU为HVM模式运行
  - io设备为PV模式运行，即分为fromt-backend模式
  - 运行于domU中的os，只要os能驱动PV接口类型的io设备

## Xen的工具栈
dom0中的管理其他虚拟机的工具： xm， xl， xe，libvirt
### xm（xen manager） / xend
- 在xem hypervisor中的dom0中，要启动xend服务，以便来接收xm的命令，来完成虚拟机创建，停止等操作
- xm 是一个命令行工具
  - create， destroy， pause，stop 等等
### xl （xen light）
- xl是基于libxenlight库提供的轻量级的命令行工具栈
- xen4.2起，xm和xl同时支持。从xen4.3起xm被废弃，只用xl
### xe / xapi
- xe是命令行工具，而xapi是xen api，所用于cloud环境中。
- 目前思捷公司的 xen server(发行了一张光盘，早起收费，现在已开源) 和 XCP （xen cloud platform)
<p align="center">
  <img src="https://i.loli.net/2019/04/06/5ca85ec89cc94.png" height=350 weight=350 >
  <img src="https://i.loli.net/2019/04/06/5ca85ecbd7984.png" height=350 weight=350 >
</p>


### virsh / libvirt
- virsh 是命令行，基于python开发，提供virt manager，virt viewer等图形化管理界面
- 重量级，每一个主机都需要安装libvirt，这是虚拟化之外的另一套工具
- 可以在安装了libvirt库的本地主机 去 远程链接对方的主机，只要对方主机的hypervisor上面也安装了libvirt库，并启动了libvirtd服务。可以通过对方主机上面的libvirt去管理对方的xen，kvm等虚拟机

<p align="center">
  <img src="https://i.loli.net/2019/04/06/5ca8638f8ec14.png" height=350 weight=350 >
</p>

### 此处可以想到
- xm，xl命令只能管理本地的xen虚拟机
- libvirt可以管理其他主机的xen，kvm，qemu等虚拟机
- 但是，当有多个物理主机，都运行hypervisor时，如何进行高效的虚拟机管理？
  - 即使是libvirt也需要远程至每一台物理主机，查看cpu，内存，io的使用情况，人工来决定在那台物理主机上创建虚拟机
  - 由此 openstack等 云平台出现，来解决这个问题
    
## CentOS对Xen对支持
- RHEL 5.7-（kernel version：2.6.18），默认对虚拟化技术xen
  - kernel
  - kernel-xen 需要安装这个内核才可以使用xen
  
- RHEL 5.8 Xen & KVM both
- RHEL 6+ 仅支持KVM，同时**取消所有版本keenel对xen对支持**，以前支持过xen的老版本内核，现在也不支持了
  - 不支持自己安装在dom0中
  - 但是支持自己运行在xen对虚拟环境中，即可以作为xen domU中对虚拟机
  - dom0 和 domU 对内核中代码配置要求不同
  
<p align="center">
  <img src="https://i.loli.net/2019/04/06/5ca86abdc2ac5.png" height=350 weight=350 >
</p>  

  - 但是xen是直接安装在硬件环境上对，不依赖于hostos

- 目前要使用xen的解决方案
  - 手动编译3.0以上版本的内核，启动对dom0的支持
  - 编译安装xen 程序
  - 制作好相关程序包的项目
    - xen4centos，xen官方专门为centos发行版提供
    - xen ma easy






