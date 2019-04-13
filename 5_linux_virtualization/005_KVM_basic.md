# basic
- centos7，加载kvm模块
  - modprobe kvm，加载模块
  - lsmod | grep kvm
- kvm管理工具栈，所有工具都可以调用api：/dev/kvm字符设备接口，管理kvm 虚拟机
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
  - ln -sv /usr/libexec/qemu-kvm /usr/bin，创建软连接够qemu-kvm，qemu-img可以直接使用
## 相关命令
- 命令分类
  - 标准选项：-m，-cpu，-smp，-name
  - 块设备选项：-hd{a,b,c,d}，-cdrom，-drive，-boot order
  - 显示选项
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
  
  - 进入虚拟机 cat /proc/cpuinfo，查看到虚拟机到cpu是QEMU Vitual CPU version 1.5.3
    - 可以在创建虚拟机时，使用命令 -cpu host，来指定虚拟机使用物理cpu架构
   
  - ctrl + alt/option + 2 进入qemu的monitor监控接口，可以实现诸多高级管理功能，ctrl + alt/option + 1退出该模式
    - info name
    - info status
    [](https://i.loli.net/2019/04/13/5cb173c9449d9.png)

  - ps -aux，可以直接kill，相当于向虚拟机发送了shutdown关机命令
    ```
    root  12860  1.9  1.0 818428 107880 pts/0   Sl+  13:12   0:22 qemu-kvm -m 128 -smp 2 -name test -hda cirros-0.3.4-x86
    ```


## 模拟网卡设备 VS 半虚拟化网卡设备
### 模拟
![](https://i.loli.net/2019/04/13/5cb1796f10f51.png)
- 虚拟机中可以查看到网卡，这就是net-frontend
- 就像vmware一样，宿主机上会生成一个vmnet2，vmnet3网卡一样，这就是net-backend后端，虽然不一定是一一对应
- 这样效率很低，多次转发
### 半虚拟化

- 虚拟机明确知道自己运行在半虚拟化状态下
- 省去了中间到虚拟化层到转发，前后端网卡直接通信
- kvm用到到半虚拟化是virtio，比如创建半虚拟化块设备

# qemu-kvm 标准命令
1. -m，megs，指定虚拟机RAM
2. -cpu，指定cpu类型，qemu虚拟cpu，物理intel cpu，或其他类型cpu，如arm
3. -smp，cpu核心数（可以具体指定threads线程，sockets插槽）每一个虚拟机到cpu都是物理机中到一个线程，所以可以动态增加虚拟cpu
4. -hda，-cdrom，不能同时使用，指定系统镜像文件位置
5. -drive，逼hda，cdrom更为高级到模式，来定义硬盘设备
  - file，硬盘镜像文件路径
  - if，interface，指定硬盘设备连的接口类型，即控制器类型，如ide，scsi，sd，mtd，floppy等
  - index，同一个接口类型对应等多个设备等序号，比如一个scsi接口可以给多个硬盘来使用
  - media，介质类型disk还是cdrom，默认是disk
  - snapshot，指定当前设备是否支持快照功能
  - cache，cpu缓存功能none，writeback（先将数据存在cpu等缓存中，再定期写入内存），unsafe，writethrough（实时将数据写入内存）
  - format，镜像文件类型qcow2等等
6. -boot，定义启动设备的引导顺序。a和b表示软驱，c表示第一块硬盘，d表示光驱，n-p表示网络适配器，默认为硬盘设备
  - -boot order=dc,once=d，once表示创建虚拟机时首次使用d光驱，再次启动时则禁用光驱

实例，启动一个xp虚拟机
- ```qemu-img create -f qcow2 -o size=20G,preallocation=metadata /images/windows/winxp.qcow2```
- ```qemu-kvm -m 512 -smp 2 -cpu host -drive file=/images/windows/winxp.qcow2,media=disk -drive file=/root/winxp.iso,media=cdrom -boot order=dc,once=d```

# qemu-kvm 显示选项

- sdl：simple directmedia layer，简单直接介质层，有c语言开发，跨平台的开源多媒体库文件。
  - 在创建虚拟机时，指定-sdl，则会在本地直接调用sdl图形窗口界面，当然本地需要有安装sdl相关显示库
  - ```qemu-kvm -m 128 -cpu host -smp 2 -name 'test' -drive file=cirros-0.3.4-x86_64-disk.img,\if=virtio,media=disk,format=qcow2,cache=writeback -sdl```，测试失败
- vnc，默认启动vnc服务5900，5901..，
  - vnc display 192.168.0.160:1，表示监听在哪个主机哪个桌面，即这个IP地址可以远程vnc访问虚拟机
- nographic，直接在cmd中显示命令行





