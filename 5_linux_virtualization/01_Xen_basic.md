
# 1. 在centos6安装xen

> 配置centos6的网卡eth0时，DNS1要指定8.8.8.8，否则下面yum安装，超级慢，或报错...

reference: [在Centos6.5上安装xen的两种方式](https://blog.51cto.com/luochen2015/1741411)

- 指定可以安装xen的yum源，否则yum install xen会报错找不到xen
  - 直接```yum install centos-release-xen```，此时或出现2个xen的repo文件
  - 移走其他的repo文件，yum.repos目录下只留上面新安装的2个xen的repo，否则安装yum install xen时会报错

- ```yum install -y xen```，默认安装最新版本xen，grub.conf文件第一个title的kernel和module也被自动修改正确
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
- 修改grub.conf文件，上面一步已经自动修改完成
  ![](https://i.loli.net/2019/04/07/5ca9bb50116f8.jpg)
- reboot
# 2. 创建PV格式的虚拟机  
当前使用 centos6 3.18.12version kernel

1. 准备磁盘镜像文件
  - ```mkdir -pv /images/xen```
  - ```qemu-img create -f raw -o size=2G /images/xen/busybox.img```
  - ```mke2fs -t ext /images/xen/bnusybox.img```
 
2. 提供根文件系统
  - 编译真正的busybox tar文件，并复制到busybox.img镜像中
  - ```mount -o loop /images/xen/busybox.img /mnt```
  - ```cp -a $BUSYBOX/_install/* /mnt```
  - ```mkdir /mnt{prox,sys,dev,var}```

3. 提供domU配置文件
  - dom0的内核文件，创建一个软连接，直接来使用
  - 根据/etc/xen下的xlexample.pvlinux模版文件来修改虚拟机配置文件
4. 启动实例
  - ```x; -v create <domU_cfg_file> -n```，检查一下
  - ```x; -v create <domU_cfg_file>```，真正创建


## 2.1 创建磁盘镜像文件qemu-img
- 创建
  - ```qemu-img create -f raw /images/xen/busybox.img 2G```
  - ```qemu-img create -f raw -o size=2G /images/xen/busybox.img```

  - 当使用```ll -h```查看时，2G大小，但是当使用```du -sh busybox.img```时，显示大小为0
    - ```df -h```查看系统中文件的使用情况
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
- 不分区，直接格式化，作为根文件系统
  - ```mke2fs -t ext2 /images/xen/busybox.img```
  - 挂载
    ```mount -o loop /images/xen/busybox.img /mnt```, -o loop??????????
    
    ```
    [root@localhost ~]# mount -o loop /images/xen/busybox.img  /mnt/
    [root@localhost ~]# cd /mnt/
    [root@localhost mnt]# ls
    lost+found
    ```
  - 若要将此busybox.img当作根文件目录，里面需要有etc，proc，usr等目录
    - 自己创建比较麻烦，复制自己当前宿主机等目录也比较慢
    - 所以直接去下载真正的busybox
      - ``` wget https://busybox.net/downloads/busybox-1.30.1.tar.bz2```，最新版本下面静态编译失败，报错```make: *** [busybox_unstripped] Error 1```
      - 使用旧版本``` wget https://busybox.net/downloads/busybox-1.23.0.tar.bz2```，或者busybox-1.22.1.tar.bz2
    - 编译安装busybox，需要安装编译工具```yum -y groupinstall "Development Tools" "Server Platform Development"```，需要移除掉之前下载的xen repo，然后只留下aliyun的repo，需要等超级久，重试好多次，最终才能安装成功
    - 下载busybox tar包，进到busybox解压目录下
      - 编译成静态链格式，即不让其再依赖于其他库。安装一个工具```yum -y install glibc-static```，如果yum安装报错，检查DNS是否是8.8.8.8
      - ```make menuconfig```去编译busybox
      - 图形画面下，Busybox Settings -> Build Options -> Build Busybox as a static binary(no shared libs)
        ![](https://i.loli.net/2019/04/07/5ca9860f20431.png)
      - ```make```,如果使用make install会直接安装在当前目录下
      - ```make install```，会安装在当前目录busybox的_install文件夹下
      - ```cp -a _install/* /mnt```
      - ```cd /mnt```
      - ```mkdir proc sys dev etc var boot home```，在mnt目录下，创建这几个文件夹。关键就是proc sys dev目录
    - 测试这个文件系统是否可用
      - ```chroot /mnt /bin/sh```
        ![](https://i.loli.net/2019/04/07/5ca988926ec1b.png)
**根文件系统准备完毕**

## 2.2 准备内核
宿主机自带的系统可以直接拿来用，虽然不能用作dom0，但是domU没有问题。创建软连接
- ```cd /boot```
- ```ln -sv vmlinuz-2.6.32-504...  vmlinuz```
- ```ln -sv initramfs-2.6.32.504...img initramfs.img```
## 2.3 创建虚拟机
进入到xen的目录 /etc/xen
- xl.conf， xl命令的通用全局配置文件，和domU的配置无关
- xlexample.hvm，xlwxample.pvlinux 为xl命令的domU创建模版
- ```cp xlexample.pvlinux busybox```,复制一份模版进行编辑
- ```vim busybox```，如下图
  ![](https://i.loli.net/2019/04/08/5cab10ca6e654.jpg)
- ```xl -v create busybox -n```,busybox为上面的配置文件，此命令并不会真正创建虚拟机，只会输出配置选项供检查使用 
- ```xl -v create busybox```，-v显示详细信息，真正创建虚拟机
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

创建虚拟机完成，但是刚才并没有配置网卡不能ssh进行远程操作，也没有图形界面，那么怎么进入虚拟机呢？
- ```xl console busybox-001```
- ```ctrl + ] ``` ctrl+右中括号，推出虚拟机终端，使用exit命令会删除当前虚拟机

## 2.4 虚拟机网络配置
命令方式，或者配置文件的方式，创建桥接设备后，会死机。kernel version, known bug，try to change to another kernel version
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

### 配置虚拟机网络接口
![](https://i.loli.net/2019/04/07/5ca9be3d0a713.jpg)

- ```vim /etc/xen/busybox```
  - 修改 ```vif = ['bridge=xenbr0']```

- ```xl -v create /etc/xen/busybox -c ```, -c创建虚拟机后直接进入控制台
  - 此时虚拟机内执行```ifconfig -a```，并不能看到eth0，因为并没有网卡驱动
  - 于是去宿主机上拷贝相应内核版本的网卡驱动程序
    - 在虚拟机上执行```uname -r```，查看虚拟机的内核版本为2.6.32-504

- 将之前创建的镜像文件挂载到/mnt，并复制网卡驱动文件进去
  - ```mount -o loop /images/xen/busybox.img /mnt```

 






















