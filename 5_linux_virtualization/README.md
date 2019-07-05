











AWS Nitro based on KVM without QEMU, with Nitro cards family(net, block)

[Amazon EC2 虚拟化技术演进：从 Xen 到 Nitro](https://mp.weixin.qq.com/s?__biz=MzU5NTY5NDk4MA==&mid=2247484096&idx=1&sn=5aa784fc2a7ead472184db09df8fb483&chksm=fe6f462bc918cf3d628c1c5f481f88423d3a43114740666e64535f4b329565720ec38ef82a7d&mpshare=1&scene=1&srcid=07053Lo1qYeGYepfHAAMsh5U&key=947199dc4f7503b16354b0d31c5308b33ff7d1be3cd20138f716cfd58b82831ffcaff7bd267a6e8669618a2941f360e03e12c397d2081b5dd2261dda5a8fea0be1128bf4b3ebecce95ae1a1cfd827f66&ascene=1&uin=MTkxMTg4MTU%3D&devicetype=Windows-QQBrowser&version=6103000b&lang=zh_CN&pass_ticket=%2FnA5jVDxtZpZD6jJo5J2lvbIylg%2By18CEWbDP8y5i5Y%3D)

[淘宝从几百到千万级并发的十四次架构演进之路](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455546396&idx=1&sn=46f4f44c220dbfc370303b5324522ff0&chksm=fb9cb67ccceb3f6a5bfd7e6466f5753791c6b7b997ea4fee932ea5eed5a5c054ad70a6618f3e&mpshare=1&scene=1&srcid=0705LclEWS8icAlI8FhHqvuI&key=311d683e94e8e0569d66e573d5f4cdbb7ab0b0f45c0ba044f33c7d785ab020117df9b80663c03d435ef368a0d60060a7b63689379543ff85c8c54b3e29991fda1e0f7d1ea90f1090342ef990eac01e5b&ascene=1&uin=MTkxMTg4MTU%3D&devicetype=Windows-QQBrowser&version=6103000b&lang=zh_CN&pass_ticket=%2FnA5jVDxtZpZD6jJo5J2lvbIylg%2By18CEWbDP8y5i5Y%3D)


-----

传统的虚拟化技术在虚拟机（VM）和硬件之间加了一个软件层Hypervisor，或者叫做虚拟机管理程序。Hypervisor的运行方式分为两类：

- 直接运行在物理硬件之上。如基于内核的KVM虚拟机，这种虚拟化需要CPU支持虚拟化技术；
- 运行在另一个操作系统。如VMWare和VitrualBox等虚拟机。

因为运行在虚拟机上的操作系统是通过Hypervisor来最终分享硬件，所以**虚拟机Guest OS发出的指令都需要被Hypervisor捕获，然后翻译为物理硬件或宿主机操作系统能够识别的指令。** VMWare和VirtualBox等虚拟机在性能方面远不如裸机，但基于硬件虚拟机的KVM约能发挥裸机80%的性能。这种虚拟化的优点是不同虚拟机之间实现了完全隔离，安全性很高，并且能够在一台物理机上运行多种内核的操作系统（如Linux和Window），但每个虚拟机都很笨重，占用资源多而且启动很慢。

Docker引擎运行在操作系统上，是基于内核的LXC、Chroot等技术实现容器的环境隔离和资源控制，在容器启动后，**容器里的进程直接与内核交互，无需经过Docker引擎中转，** 因此几乎没有性能损耗，**能发挥出裸机的全部性能**。但由于Docker是基于Linux内核技术实现容器化的，因此使得容器内运行的应用只能运行在Linux内核的操作系统上。目前在Window上安装的docker引擎其实是利用了Window自带的Hyper-V虚拟化工具自动创建了一个Linux系统，容器内的操作实际上是间接使用这个虚拟系统实现的。









-----

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









