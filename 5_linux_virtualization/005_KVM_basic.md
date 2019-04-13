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
  - qcow2 流行的高级磁盘镜像格式，支持众多磁盘操作
  - raw 很多高级功能不支持，比如磁盘resize等
- 显卡不支持半虚拟化，但是在centos7上，可以使用半虚拟化
- 基于busybox打造的cirros镜像
  - http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
  - qemu-img info cirros-0.3.4-x86_64-disk.img 
    ```
    [root@server15 ~]#  qemu-img info cirros-0.3.4-x86_64-disk.img 
    image: cirros-0.3.4-x86_64-disk.img
    file format: qcow2
    virtual size: 39M (41126400 bytes)
    disk size: 13M
    cluster_size: 65536
    Format specific information:
    compat: 0.10
     ```
## 创建kvm虚拟机
- qemu-kvm -m 128 -smp 2 -name 'test' -hda cirros-0.3.4-x86_64-disk.img 
  - 默认以vnc的方式启动虚拟机，所以需要安装vnc客户端，并不会自动打开控制台
  ```
  qemu-kvm -m 128 -smp 2 -name 'test' -hda cirros-0.3.4-x86_64-disk.img 
  VNC server running on `::1:5900'
  ```
- ss -tnl
  ```
  [root@server15 ~]# ss -tnl
  State       Recv-Q Send-Q Local Address:Port               Peer Address:Port                           
  LISTEN      0      1           ::1:5900                     :::*                  
  ```
- 安装linux的vnc客户端tigervnc
  - yum install -y tigervnc
  - 安装完之后，会安装一个vncviewer
    ```
    [root@server15 ~]# rpm -ql tigervnc
    /usr/bin/vncviewer
    ```
  - ```vncviewer :5900```，即可启动本地的5900端口
    ![](https://i.loli.net/2019/04/13/5cb1721272ae6.png)
    











