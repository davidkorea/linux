- 硬件
  1. 定制云主机的cpu多核心，intel定制 24核，硬件红利
- 存储
  1. 游戏快速响应，砍一刀马上嗷嗷叫。内存存储，redis
  2. 存储贵 - 各种优化
- 游戏大数据
  1. 数据驱动游戏运营
  2. NLP cnn rnn言论分析，热点话题，游戏中的各种问题
  3. spark
  4. 用户聚类，流失预测
- 游戏搞不好，可能很大原因是技术，技术是木桶最短板
  1. 不能快速回档，运行事故
  2. 账号，安全支付
  3. 伸缩性，快速开服，快速合服
  4. 被数据分析公司偷走数据，云的数据隔离，使得数据不被偷走
  5. 客服机器人
  6. 哪些自己做，哪些找第三方来做
- come stay pay
- 充值  
- 防外挂，云服务降低费用
- ddos 超大流量工具 - 大数据 - 游戏早期在安全上进行大量投入，抵挡住前期的攻击，挺过去才能成





# kvm-openstack-docker-kubernetes


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
  - hypervisor -> vm
- Type-II
  - host -> vmm(virtual machine monitor) -> vm
  
- Xen，KVM在云端，成为Iaas，infrastructure 基础服务
- lxc，container在云端，称为Paas，platform 平台服务









