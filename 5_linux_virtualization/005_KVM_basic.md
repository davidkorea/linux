# basic
- centos7，加载kvm模块
  - modprobe kvm，加载模块
  - lsmod | grep kvm
- kvm管理工具栈
  - qemu-kvm，/usr/libexec目录下，更为底层的管理工具
    - qemu本身是一个模拟器，可以模拟和底层架构不一样的硬件，比如硬件是intel，qemu可以模拟amd的cpu
    - 对于虚拟化来讲，底层硬件会直接输出给虚拟机，只是虚拟io设备
  - libvirt
    - 安装工具
      - virt-install。命令行方式安装
      - virt-manager，图形化安装，管理
    - 管理工具
      - virsh
      - virt-manager
      - virt-viewer
  - openstack，也算是一个虚拟机的管理工具

# qemu-kvm
## 安装qemu-kvm
- kvm虚拟机不依赖于管理员权限启动虚拟机
- yum install -y qemu-kvm 
  - rpm -ql qemu-kvm查看到qemu-kvm默认在/sur/libexec目录下，需要创建软连接到/usr//bin目录下，才可以直接使用命令
  - ln -sv /usr/libexec/qemu-kvm /usr/bin，创建软连接够qemu-kvm和qemu-img都可以直接使用
## 相关命令
- 命令分类
  - 标砖选项
  - 显示选项
  - 块设备选项
  - 网络选项
- 虚拟机磁盘镜像文件有多种类型
  - qemu-img支持很多 Supported formats: vvfat vpc vmdk vhdx vdi ssh sheepdog rbd raw host_cdrom host_floppy host_device file qed qcow2 qcow parallels nbd iscsi gluster dmg tftp ftps ftp https http cloop bochs blkverify blkdebug








