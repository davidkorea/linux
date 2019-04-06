# kvm-openstack-docker-kubernetes

# Xen
- Xen属于type-I型虚拟化，Xen hypervisor直接运行在硬件上
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
### 3. domU 非特权域
根据其虚拟化等实现方式有多种类型  PV，  HVM，   PV on HVM

domU中的虚拟机类型：
- Xen的PV技术：半虚拟化
  - 不依赖于CPU的硬件辅助特性，guestos内核向Xen hypervisor发起hyper call进行cpu/内存调用，要求guestsos的内核作出修改，以知晓自己运行于pv/半虚拟化环境
  - 对于io设备，guestos中需要能驱动io设备的fromtend，dom0中需要可以驱动io设别的backend
  - 运行于domU中的os：linux（2.6.24+）NetBSD，FreeBSD，OP en solosolars
    ![](https://i.loli.net/2019/04/06/5ca85a04348bc.png)
- Xen的HVM技术：全虚拟化
  - 依赖于intel-VT，AMD-V，不需要guestos的内核向Xen hypervisor发起hyper call进行调用，可以直接运行指令。但仍然由Xen hypervisor将cpu和内存虚拟后进行提供
  - 依赖于QEMU来虚拟io设备
  - 运行于domU中的os：几乎所有支持x86平台的系统，当然也包括windows
    ![](https://i.loli.net/2019/04/06/5ca85ecbdbff7.png)
- PV on HVM
  - CPU为HVM模式运行
  - io设备为PV模式运行，即分为fromt-backend模式
  - 运行于domU中的os，只要os能驱动PV接口类型的io设备

## Xen的工具栈
dom0中的管理其他虚拟机的工具： xm， xl， xe
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
![](https://i.loli.net/2019/04/06/5ca85ec89cc94.png)

![](https://i.loli.net/2019/04/06/5ca85ecbd7984.png)














-----


# 1. 虚拟化技术分类
- 模拟器emulization。可以虚拟和宿主机不同的系统，灵活，效率差
  - QEMU，PearPC，Bochs等技术
  - 需要将不同架构（x86，ARM 等等）的虚拟机的指令，经过转换，在转换为宿主机支持的指令。因此，指令集的转换效率低

- 完全虚拟化 full vitualization, native virtualizatiom. 虚拟机需要和宿主机的硬件架构完全相同。如果宿主机是x86（winodws，linux），那么虚拟机也必须相同，位数32，64都可以
  - 因为虚拟机和宿主机的架构相同，所以有些不需要调用特权的指令可以直接由cpu处理，而不需要转换
  - 如果是特权指令，需要VMM虚拟机监视器来捕获虚拟机的指令，在宿主机转换后，再将结果返回给虚拟机（类似上面的虚拟器）
  - 硬件辅助虚拟化。CPU多了一个环，0，1，2，3的基础上又增加了环-1
    - 因为上面的VMM捕获指令很浪费时间，所以CPU增加了一个-1环
    - VMM或者宿主机，运行在-1环
    - 虚拟机运行在0环
    - 当虚拟机调用特权指令时，CPU会自动将环0的指令交由环-1处理。避免了VMM软件的自动监控，而是硬件直接处理
  - Vmware，virtualbox，paralels，KVM，Xen（在硬件辅助虚拟化的情况下，也支持完全虚拟化）
- 半虚拟化，准虚拟化
  - 需要有针对性的修改guest kernel，让虚拟机明确知道自己运行在虚拟化环境中。其内核不能直接操作硬件。而是需要发起hyper call的方式去实现
  - 因为要修改系统内核，所以闭源系统很难进行半虚拟化
  - Xen，UML（user mode linux）
  
- OS级别虚拟化
  - 容器级虚拟化 docker
  - 无虚拟机监控器VMM
  - 仅将正常操作系统的用户空间切割为多分，彼此间互相隔离，每一份被视为一个虚拟机
  - openVZ，LXC（linux container），libcontainer
  - 需要内核中的名称空间划分技术name space
    - 比如第一个container需要监听80端口，而第二个container也需要80端口。那么谁应该使用真正的80端口？
    - 因此端口需要彼此划分为多份
    - 文件系统也需要彼此划分多多份
- 库级别的虚拟化
  - WINE（windows env），能够在linux上运行wondows程序
  - JVM java vm

虚拟化种类

- Type-I，hypervisor 直接运行在硬件
- Type-II
  
- Xen，KVM在云端，成为Iaas，infrastructure 基础服务
- lxc，container在云端，称为Paas，platform 平台服务









