
- centos7，加载kvm模块
  - modprobe kvm，加载模块
  - lsmod | grep kvm
- kvm管理工具栈
  - qemu-kvm
    - qemu本身是一个模拟器，可以模拟和底层架构不一样的硬件，比如硬件是intel，qemu可以模拟amd的cpu
    - 对于虚拟化来讲，底层硬件会直接输出给虚拟机，只是虚拟io设备
  - virsh
