# 创建xen虚拟机

- DomU的内核文件位于Dom0，不在DomU的磁盘镜像内
- 桥接物理网络

# 1. 准备centos6 xen kernel环境

- 配置centos6的网卡eth0时，DNS1要指定8.8.8.8，否则下面yum安装，超级慢，或报错...
- 可以不配置aliyun yum源，用系统自带的也很快

reference: [在Centos6.5上安装xen的两种方式](https://blog.51cto.com/luochen2015/1741411)

### 1. 安装xen yum源
指定可以安装xen的yum源，否则直接yum install xen会报错找不到xen
- ```yum install centos-release-xen```，
- 执行上面命令后，会出现2个xen的repo文件（移走其他的repo文件，yum.repos目录下只留上面新安装的2个xen的repo，否则安装yum install xen时会报错。有可能是DNS的问题）
### 2. 安装xen
- ```yum install -y xen```

默认安装最新版本xen，grub.conf文件第一个title的kernel和module也被自动修改正确
### 3. 修改grub.conf文件
```diff
[root@localhost ~]# vim /etc/grub.conf 
default=0
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title CentOS (4.9.127-32.el6.x86_64)
        root (hd0,0)
-       kernel /xen.gz dom0_mem=1024M,max:1024M cpuinfo com1=115200,8n1 console=com1,tty loglvl=all guest_loglvl=all
+       kernel /xen.gz dom0_mem=1024M,cpufreq=xen,dom0_max_vcpus=2,dom0_vcpus_pin
        module /vmlinuz-4.9.127-32.el6.x86_64 ro root=/dev/mapper/VolGroup-lv_root nomodeset rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=VolGroup/lv_swap SYSFONT=latarcyrheb-sun16 rd_LVM_LV=VolGroup/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet crashkernel=auto
        module /initramfs-4.9.127-32.el6.x86_64.img

title CentOS (2.6.32-754.11.1.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-754.11.1.el6.x86_64 ro root=/dev/mapper/VolGroup-lv_root nomodeset rd_NO_LUKS LANG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=VolGroup/lv_swap SYSFONT=latarcyrheb-sun16 rd_LVM_LV=VolGroup/lv_root  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet crashkernel=auto
        initrd /initramfs-2.6.32-754.11.1.el6.x86_64.img
```
- dom0_vcpus_pin dom0的虚拟cpu固定在物理cpu的某一核心
    
  [](https://i.loli.net/2019/04/07/5ca9b75e07d2f.jpg)
  
- 安装后会/boot目录下会出现xen
  [](https://i.loli.net/2019/04/07/5ca9bb4a33330.jpg)
  ```
  [root@localhost ~]# cd /boot/
  [root@localhost boot]# ls
  config-2.6.32-754.11.1.el6.x86_64         symvers-2.6.32-754.el6.x86_64.gz
  config-2.6.32-754.el6.x86_64              symvers-4.9.127-32.el6.x86_64.gz
  config-4.9.127-32.el6.x86_64              System.map-2.6.32-754.11.1.el6.x86_64
  efi                                       System.map-2.6.32-754.el6.x86_64
  grub                                      System.map-4.9.127-32.el6.x86_64
  initramfs-2.6.32-754.11.1.el6.x86_64.img  vmlinuz-2.6.32-754.11.1.el6.x86_64
  initramfs-2.6.32-754.el6.x86_64.img       vmlinuz-2.6.32-754.el6.x86_64
  initramfs-4.9.127-32.el6.x86_64.img       vmlinuz-4.9.127-32.el6.x86_64
  initrd-2.6.32-754.el6.x86_64kdump.img     xen-4.8.5.12.ga1f8fe0628-1.el6.config
  lost+found                                xen-4.8.5.12.ga1f8fe0628-1.el6.gz
  symvers-2.6.32-754.11.1.el6.x86_64.gz     xen.gz
  ```
  
### 4. reboot
重启后使用内核4.9.127-32.el6.x86_64，宿主机即为xen-dom0
```
[root@localhost boot]# xl list
Name                                        ID   Mem VCPUs      State   Time(s)
Domain-0                                     0  3271     4     r-----    3270.7
```

# 2. 准备虚拟机磁盘镜像文件  
当前使用 centos6 4.9.127-32.el6.x86_64 version kernel

> 1. 准备磁盘镜像文件
>   - ```mkdir -pv /images/xen```
>   - ```qemu-img create -f raw -o size=2G /images/xen/busybox.img```
>   - ```mke2fs -t ext /images/xen/bnusybox.img```
>  
> 2. 提供根文件系统
>   - 下载真正的busybox tar文件，静态编译后，复制到busybox.img镜像中
>   - ```mount -o loop /images/xen/busybox.img /mnt```
>   - ```cp -a $BUSYBOX/_install/* /mnt```
>   - ```mkdir /mnt{prox,sys,dev,var}```
> 
> 3. 提供domU配置文件
>   - dom0的内核文件，创建一个软连接，直接来使用
>   - 根据/etc/xen下的xlexample.pvlinux模版文件来修改虚拟机配置文件
> 4. 启动实例
>   - ```x; -v create <domU_cfg_file> -n```，检查一下
>   - ```x; -v create <domU_cfg_file>```，真正创建


## 2.1 创建磁盘镜像文件qemu-img
- ```mkdir -pv /images/xen```，创建磁盘镜像文件存放路径
#### 1. 创建磁盘镜像
- ```qemu-img create -f raw /images/xen/busybox.img 2G```
- ```qemu-img create -f raw -o size=2G /images/xen/busybox.img```

- 当使用```ll -h```查看时，2G大小，但是当使用```du -sh busybox.img```时，显示大小为0
  - ```df -h``` 查看系统中文件的使用情况
  - ```du -sh *```查看当前目录下各个文件及目录占用空间大小

```
  [root@localhost ~]# qemu-img create -f raw -o size=2G /images/xen/busybox.img
  Formatting '/images/xen/busybox.img', fmt=raw size=2147483648 
  
  [root@localhost ~]# mke2fs -t ext /images/xen/busybox.img 
  mke2fs 1.41.12 (17-May-2010)
  /images/xen/busybox.img is not a block special device.
  Proceed anyway? (y,n) y
  Filesystem label=
  OS type: Linux
  Block size=4096 (log=2)
  Fragment size=4096 (log=2)
  Stride=0 blocks, Stripe width=0 blocks
  131072 inodes, 524288 blocks
  26214 blocks (5.00%) reserved for the super user
  First data block=0
  Maximum filesystem blocks=536870912
  16 block groups
  32768 blocks per group, 32768 fragments per group
  8192 inodes per group
  Superblock backups stored on blocks: 
          32768, 98304, 163840, 229376, 294912

  Writing inode tables: done                            
  Writing superblocks and filesystem accounting information: done

  This filesystem will be automatically checked every 34 mounts or
  180 days, whichever comes first.  Use tune2fs -c or -i to override.
  ```
#### 2. 不分区，直接格式化，作为根文件系统
- 格式化磁盘
  - ```mke2fs -t ext /images/xen/busybox.img```
- 挂载
  - ```mount -o loop /images/xen/busybox.img /mnt```
    
```
[root@localhost ~]# mount -o loop /images/xen/busybox.img  /mnt/
[root@localhost ~]# cd /mnt/
[root@localhost mnt]# ls
lost+found
```
## 2.2 创建完整磁盘文件系统
若要将此busybox.img当作根文件目录，里面需要有etc，proc，usr等目录。自己创建比较麻烦，复制自己当前宿主机等目录也比较慢，所以直接去下载真正的busybox
#### 1. 下载busybox.tar.bz2
- ``` wget https://busybox.net/downloads/busybox-1.30.1.tar.bz2```，最新版本下面静态编译失败，报错```make: *** [busybox_unstripped] Error 1```
- 使用旧版本``` wget https://busybox.net/downloads/busybox-1.23.0.tar.bz2```，或者busybox-1.22.1.tar.bz2
#### 2. 下载编译工具
- ```yum -y groupinstall "Development Tools" "Server Platform Development"```，编译安装busybox，需要安装编译工具。如果安装centos6时选择来软件开发使用，不安装也可以（需要移除掉之前下载的xen repo，然后只留下aliyun的repo，需要等超级久，重试好多次，最终才能安装成功）
- 此时会报错
  ```
  事务测试出错：
  file /etc/libvirt/libvirt.conf from install of libvirt-libs-4.1.0-2.xen48.el6.x86_64 conflicts with
  file from package libvirt-client-0.10.2-64.el6.x86_64
  ```
  - 删除libvirt-client-0.10.2-64.el6.x86_64，重新安装
  - ```yum remove libvirt-client-0.10.2-64.el6.x86_64```，再次执行安装成功
#### 3. 静态编译安装busybox
编译成静态链格式，即不让其再依赖于其他库
- 解压busybox tar包，进到busybox解压目录下
- ```yum -y install glibc-static```，安装一个工具，如果yum安装报错，检查DNS是否是8.8.8.8
- ```make menuconfig```
  - 图形画面下，Busybox Settings -> Build Options -> Build Busybox as a static binary(no shared libs)
    ![](https://i.loli.net/2019/04/09/5cac33edc7334.png)
- ```make -j 4```，如果使用make install会直接安装在当前目录下，如果出错，那么```make clean```，排错后，再次执行
- ```make install```，会安装在当前目录busybox的_install文件夹下
- ```cp -a _install/* /mnt```
- ```cd /mnt```
- ```mkdir proc sys dev etc var boot home```，在mnt目录下，创建这几个文件夹。关键就是proc sys dev目录
    - 测试这个文件系统是否可用
      - ```chroot /mnt /bin/sh```
        ```
        [root@localhost ~]# chroot /mnt/ /bin/sh
        / # ls
        bin         etc         linuxrc     proc        sys         var
        dev         lib         lost+found  sbin        usr
        ```

# 3. 准备虚拟机内核
宿主机自带的2.6.32-754内核可以直接拿来用，虽然不能用作dom0，但是domU没有问题。创建软连接
- ```cd /boot```
- ```ln -sv vmlinuz-2.6.32-754.el6.x86_64  vmlinuz```
- ```ln -sv initramfs-2.6.32-754.el6.x86_64.img initramfs.img```

# 4. 创建PV类型虚拟机（无网络）
### 4.1 创建虚拟器配置文件
进入到xen的目录 /etc/xen
- xl.conf， xl命令的通用全局配置文件，和domU的配置无关
- xlexample.hvm，xlwxample.pvlinux 为xl命令的domU创建模版
#### 1. 复制一份模版进行编辑
- ```cp xlexample.pvlinux busybox_conf```,
- ```vim busybox_conf```，做如下修改
  ```
  [root@localhost xen]# grep -v ^# busybox_conf 
  name = "busybox-001"
  kernel = "/boot/vmlinuz"
  ramdisk = "/boot/initramfs.img"
  extra = "selinux=0 init=/bin/sh"
  memory = 256
  vcpus = 2
  disk = [ '/images/xen/busybox.img,raw,xvda,rw' ]
  root = '/dev/xvda ro'
  ```
#### 3. 创建虚拟机
- ```xl -v create busybox_conf -n```，busybox_conf为上面的配置文件，此命令并不会真正创建虚拟机，只会输出配置选项供检查使用 
- ```xl -v create busybox_conf```，-v显示详细信息，真正创建虚拟机
  - 报错
    ```
    libxl: error: libxl_device.c:1237:device_hotplug_child_death_cb: script: File /images/xen/busybox.img is loopback-mounted through 7:0,
    which is mounted in the privileged domain,
    ```
    ```
    libxl: error: libxl_device.c:1237:device_hotplug_child_death_cb: script: Could not find bridge device xenbr0
    ```
    - umount /mnt
    - 配置文件中，注释掉网卡配置
- ```xl list``` ，会列出Domain-0 和 刚刚创建的busybox-001虚拟机
  ```
  [root@localhost xen]# xl list
  Name                                        ID   Mem VCPUs      State   Time(s)
  Domain-0                                     0  3783     4     r-----    1125.6
  busybox-001                                  3   256     2     -b----       3.6
  ```
#### 4. 进入虚拟机  
创建虚拟机完成，但是刚才并没有配置网卡不能ssh进行远程操作，也没有图形界面，那么怎么进入虚拟机呢？
- ```xl console busybox-001```
- ```ctrl + ] ``` ctrl+右中括号，推出虚拟机终端，使用exit命令会删除当前虚拟机

# 5. 创建PV类型虚拟机（有网络 - 桥接网络）

## 5.1 创建网桥设备
安装系统时，默认安装来bridge-utils工具。此处通过创建网桥设备的配置文件的方式来创建网桥
#### 1. 修改配置文件
- 创建ifcfg-xenbr0，
  - 一定要加上NM_CONTROLLED=no，否则service restart报错``` Error: Connection activation failed: Master connection not found or invalid```
  ```
  [root@localhost network-scripts]# cat ifcfg-xenbr0 
  DEVICE=xenbr0
  TYPE=Bridge
  ONBOOT=yes
  BOOTPROTO=static
  IPADDR=192.168.0.160
  NETMASK=255.255.255.0
  GATEWAY=192.168.0.1
  DNS1=8.8.8.8
  DNS2=168.126.63.1
  NM_CONTROLLED=no
  ```
- 修改ifcfg-eth0，一定要加上NM_CONTROLLED=no
  ```
  [root@localhost network-scripts]# cat ifcfg-eth0 
  DEVICE=eth0
  HWADDR=00:0C:29:80:84:B5
  TYPE=Ethernet
  UUID=bd9ca1a9-fc6a-4866-9444-d2fd4a0d487b
  ONBOOT=yes
  BOOTPROTO=static
  BRIDGE=xenbr0
  NM_CONTROLLED=no
  ```
#### 2. 关闭networkManager
-  ```chkconfig NetworkManager off```
- 查看 ```chkconfig --list NetworkManager```
        ```
        [root@localhost network-scripts]# chkconfig --list NetworkManager
        NetworkManager  0:off   1:off   2:off   3:off   4:off   5:off   6:off
        ```
#### 3. 重启网络服务
- ```service network restart```

-> 如果重启服务后，系统死机，参考下面issue

## 5.2 修改虚拟机配置文件，添加网卡配置选项
#### 1. 修改xen虚拟机配置文件
```diff
[root@localhost xen]# grep -v ^# busybox_conf 
  name = "busybox-002"
  kernel = "/boot/vmlinuz"
  ramdisk = "/boot/initramfs.img"
  extra = "selinux=0 init=/bin/sh"
  memory = 256
  vcpus = 2
+ vif = [ 'bridge=xenbr0' ]
  disk = [ '/images/xen/busybox.img,raw,xvda,rw' ]
  root = '/dev/xvda ro'
```
#### 2. 创建带有网卡设备的虚拟机
- ```xl -v create busybox-conf -c```，-c 创建好之后直接进入虚拟机
#### 3. 查看虚拟机的网卡设备
- 在虚拟机中执行ifconfig -a，依然看不到eth0，这是因为没有网卡驱动
```
[root@localhost ~]# xl console busybox-002
/ # ifconfig -a
lo        Link encap:Local Loopback  
    LOOPBACK  MTU:65536  Metric:1
    RX packets:0 errors:0 dropped:0 overruns:0 frame:0
    TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
    collisions:0 txqueuelen:0 
    RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
## 5.3 制作虚拟机网卡驱动
#### 1. 复制宿主机（dom0的网卡驱动）到虚拟机
- 进入这个路径``` /lib/modules/2.6.32-754.el6.x86_64/kernel/drivers/net```
- 查看网卡驱动，有没有其他依赖的包,depends为空，则没有依赖包
  - xen-netfront.ko，无依赖包
    ```
    [root@localhost net]# modinfo xen-netfront.ko 
    filename:       xen-netfront.ko
    alias:          xennet
    alias:          xen:vif
    license:        GPL
    description:    Xen virtual network device frontend
    retpoline:      Y
    srcversion:     5C6FC78BC365D9AF8135201
    depends:        
    vermagic:       2.6.32-754.el6.x86_64 SMP mod_unload modversions 
    ```
  - 8139too.ko，有依赖mii这个包
    ```
    [root@localhost net]# modinfo 8139too.ko 
    depends:        mii
    ```
    mii 没有依赖包
    ```
    [root@localhost net]# modinfo mii.ko 
    filename:       mii.ko
    license:        GPL
    description:    MII hardware support library
    author:         Jeff Garzik <jgarzik@pobox.com>
    retpoline:      Y
    srcversion:     94D0B170BD4AA5F0553F61D
    depends:        
    vermagic:       2.6.32-754.el6.x86_64 SMP mod_unload modversions     
    ```
#### 2. 拷贝网卡驱动文件至虚拟机镜像
其实只拷贝xen-netfront.ko文件即可
```
[root@localhost net]# mount -o loop /images/xen/busybox.img /mnt/
[root@localhost net]# mkdir /mnt/lib/modules -pv
[root@localhost net]# cp xen-netfront.ko 8139too.ko mii.ko /mnt/lib/modules/

[root@localhost net]# cd /mnt/lib/modules/
[root@localhost modules]# ls
8139too.ko  mii.ko  xen-netfront.ko

[root@localhost net]# umount /mnt
```

#### 3. 创建虚拟机
修改配置文件中虚拟机name后，再次创建新的虚拟机
- ```xl -v create busybox-conf -c```
#### 4. 虚拟机中加载网卡驱动frontend
刚开始还是查不到网卡设备，安装网卡驱动后，显示eth0正常
- ```insmod /lib/modules/xen-netfront.ko```
  ```  
  / # ifconfig -a                                                       # 未加载网卡驱动
  lo        Link encap:Local Loopback  
            LOOPBACK  MTU:65536  Metric:1
            RX packets:0 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0 
            RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

  / # insmod /lib/modules/xen-netfront.ko                               # 加载驱动文件
  Initialising Xen virtual ethernet driver.                             # 网卡设备eth0显示正常
  / # ifconfig -a
  eth0      Link encap:Ethernet  HWaddr 00:16:3E:21:EC:35  
            BROADCAST MULTICAST  MTU:1500  Metric:1
            RX packets:0 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000 
            RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
            Interrupt:18 

  lo        Link encap:Local Loopback  
            LOOPBACK  MTU:65536  Metric:1
            RX packets:0 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0 
            RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
            
  / # ifconfig eth0 192.168.0.19 up                                     # 配置虚拟机ip
  
  / # ping 192.168.0.18                                                 # ping 物理机正常  
  PING 192.168.0.18 (192.168.0.18): 56 data bytes
  64 bytes from 192.168.0.18: seq=0 ttl=64 time=1.391 ms
  ```
#### 5. 物理机（dom0）上查看网卡设备backend
- 在物理机上ifconfig也能查到该虚拟机的net-backend vif2.0，vif后面的数字2，就是虚拟机的id=2，2.0的0表示第0个网卡
- 同时brctl也能查到桥接状态
  ```
  [root@localhost xen]# brctl show
  bridge name     bridge id               STP enabled     interfaces
  pan0            8000.000000000000       no
  virbr0          8000.52540092b0d2       yes             virbr0-nic
  xenbr0          8000.000c29e97e25       no              eth0
                                                          vif1.0
                                                          vif2.0
  ```
- 因为是PV半虚拟化，所以I/O设备是分为frontend（domU）和backend（don0）两部分

**以上虚拟机网络方式为：桥接并使用物理机网络，即虚拟机和物理机再同一局域网，也可以想物理机一样正常访问外网**


# 6. 其他网络模式虚拟机

虚拟机的网络连接方式有
- 桥接使用物理网络
- VMNET（虚拟机之间通信，无法访问物理机，无法访问外网）
- Host only（虚拟机之间通信，以及虚拟机和物理机可以通信，虚拟机无法访问外网）

## 6.1 创建VMNET虚拟机

创建2台虚拟机，不关联物理网卡，所有虚拟机使用新创建网桥xenbr1

### 1. 创建虚拟机磁盘镜像
- ```cp /images/xen/busybox.img /images/xen/busybox_vmnet.img ```
### 2. 创建虚拟机配置文件
- xen vm1
```diff
vim /etc/xen/busybox_conf
  name = "busybox-001"
  kernel = "/boot/vmlinuz"
  ramdisk = "/boot/initramfs.img"
  extra = "selinux=0 init=/bin/sh"
  memory = 256
  vcpus = 2
- vif = [ 'bridge=xenbr0' ]
+ vif = [ 'bridge=xenbr1' ]
  disk = [ '/images/xen/busybox.img,raw,xvda,rw' ]
  root = '/dev/xvda ro'
```
- xen vm2
```cp /etc/xen/busybox_conf /etc/xen/busybox_conf_vmnet```
```diff
- name = "busybox-001"
+ name = "busybox-002"
  kernel = "/boot/vmlinuz"
  ramdisk = "/boot/initramfs.img"
  extra = "selinux=0 init=/bin/sh"
  memory = 256
  vcpus = 2
+ vif = [ 'bridge=xenbr1' ]
- disk = [ '/images/xen/busybox.img,raw,xvda,rw' ]
+ disk = [ '/images/xen/busybox.img_vmnet,raw,xvda,rw' ]
  root = '/dev/xvda ro'
```
### 3. 创建网桥设备
```diff
[root@localhost ~]# cd /etc/sysconfig/network-scripts/
[root@localhost network-scripts]# cp ifcfg-xenbr0 ifcfg-xenbr1
[root@localhost network-scripts]# vim ifcfg-xenbr1


[root@localhost network-scripts]# service network restart
```
### 4. 创建2台虚拟机
```
[root@localhost ~]# xl -v create /etc/xen/busybox_conf
[root@localhost ~]# xl -v create /etc/xen/busybox_conf_vmnet 

[root@localhost ~]# xl li
Name                                        ID   Mem VCPUs      State   Time(s)
Domain-0                                     0  3271     4     r-----    3811.9
busybox-001                                  7   256     2     -b----       3.5
busybox-002                                  8   256     2     -b----       3.7

[root@localhost ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540088a23f       yes             virbr0-nic
xenbr0          8000.000c298084b5       no              eth0
xenbr1          8000.feffffffffff       no              vif7.0
                                                        vif8.0
```
### 5. 加载虚拟机网卡驱动并配ip地址
虚拟机配置10.0.0.0网段，互相可以ping通
```
[root@localhost ~]# xl console busybox-001

/ # insmod /lib/modules/xen-netfront.ko 
Initialising Xen virtual ethernet driver.

/ # ifconfig eth0 10.0.0.10 up
/ # ifconfig 
eth0      Link encap:Ethernet  HWaddr 00:16:3E:62:23:71  
          inet addr:10.0.0.10  Bcast:10.255.255.255  Mask:255.0.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:12 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:1344 (1.3 KiB)  TX bytes:0 (0.0 B)
          Interrupt:18 
```
```
[root@localhost ~]# xl console busybox-002

/ # insmod /lib/modules/xen-netfront.ko 
Initialising Xen virtual ethernet driver.

/ # ifconfig eth0 10.0.0.20 up

/ # ping 10.0.0.10
PING 10.0.0.10 (10.0.0.10): 56 data bytes
64 bytes from 10.0.0.10: seq=0 ttl=64 time=17.647 ms
64 bytes from 10.0.0.10: seq=1 ttl=64 time=1.232 ms
64 bytes from 10.0.0.10: seq=2 ttl=64 time=1.295 ms
```


-----

## Issue: 命令方式和配置文件的方式，创建桥接设备后，都会死机
kernel version, known bug，try to change to another kernel version
- ```yum list all kernel*``` ，查看所有可用kernel
下面2个都要安装，注意要安装新的kernel而不是升级当前使用的kernel，以免升级后无法开机
- ```yum -y install kernel-3.10.68```
- ```yum -y install kernel-3.10.68 kernel-firmware-3.10.68```
- ```vim /etc.grub.conf```
  ![](https://i.loli.net/2019/04/07/5ca9af2ec89f1.png)
  
这个版本的kernrl开机失败，尝试在该版本上面更新bridge-utils版本。实在不行，再次更换3.7.4版本kernel-xen-3.7.4
- ```rpm -q bridge-utils ```,查看版本bridge-utils-1.2-10.el6.x86_64，升级至1.5
- 命令行创建桥设备
  ![](https://i.loli.net/2019/04/07/5ca9b1761026b.png)


依然死机，去安装3.7.4 kernel和其fireware，再不行更改网卡类型，不使用r1000
- yum安装kernel，更改grub
  ![](https://i.loli.net/2019/04/07/5ca9b426edf91.jpg)

果然这个3.7.4版本ok。创建网桥后并没有死机
- 创建网桥xenbr0

-----





















