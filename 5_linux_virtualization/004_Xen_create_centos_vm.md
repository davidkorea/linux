
# 1.xl全手动创建centos6虚拟机
## 1.1 准备内核文件
- yum/cdrom中的文件(iso文件内也都有)
  - isolinux
    - vmlinuz
    - initrd
    
- https://mirrors.aliyun.com/centos/6.10/os/x86_64/isolinux/
  ```
  Index of /centos/6.10/os/x86_64/isolinux/
  ../
  boot.msg                     29-Jun-2018 16:11                  84
  grub.conf                    29-Jun-2018 16:11                 321
  initrd.img                   29-Jun-2018 16:11            40991898
  isolinux.bin                 29-Jun-2018 16:20               24576
  isolinux.cfg                 29-Jun-2018 16:11                 924
  memtest                      29-Jun-2018 16:11              183012
  splash.jpg                   29-Jun-2018 16:11              151230
  vesamenu.c32                 29-Jun-2018 16:11              163728
  vmlinuz                      29-Jun-2018 16:11             4315504
  ```
  - ```wget https://mirrors.aliyun.com/centos/6.10/os/x86_64/isolinux/initrd.img ```
  - ```wget https://mirrors.aliyun.com/centos/6.10/os/x86_64/isolinux/vmlinuz  ```
- ```mkdir /images/kernel```
- ```mv initrd.img vmlinuz /images/kernel```
## 1.2 创建磁盘镜像qemu-img
-  ```qemu-img create -f qcow2 -o size=20G,preallocation=metedata /images/xen/centos6.10.img```
-  ```df -lh```查看物理磁盘空间够不够

## 1.3 准备虚拟机配置文件
- ```cp /etc/xen/busybox_conf centos_conf```
- 修改配置文件
  ```
  vim /etc/xen/centos_ conf
  name = 'centos-001'
  kernel = '/images/kernel/vmlinuz'
  ramdisk = '/images/kernelinitrd.img'
  extra = ''
  memory = 512
  vcpus = 2
  vif = ['bridge=xenbr0']
  disk = ['/images/xen/centos6.10.img,qcow2,xvda,rw']
  #root = '/dev/xvda ro '
  ```
## 1.4 创建虚拟机
- ```xl create /etc/xen/centos_conf```
- ```xl console centos-001```
#### 1. English
```
Welcome to CentOS for x86_64

                    ┌────────┤ Choose a Language ├────────┐
                    │                                     │
                    │ What language would you like to use │
                    │ during the installation process?    │
                    │                                     │
                    │      Catalan                ↑       │
                    │      Chinese(Simplified)    ▒       │
                    │      Chinese(Traditional)   ▮       │
                    │      Croatian               ▒       │
                    │      Czech                  ▒       │
                    │      Danish                 ▒       │
                    │      Dutch                  ▒       │
                    │      English                ↓       │
                    │                                     │
                    │               ┌────┐                │
                    │               │ OK │                │
                    │               └────┘                │
                    │                                     │
                    │                                     │
                    └─────────────────────────────────────┘

  <Tab>/<Alt-Tab> between elements  | <Space> selects | <F12> next screen
```
-----
#### 2. URL
```
Welcome to CentOS for x86_64



                      ┌───┤ Installation Method ├───┐
                      │                             │
                      │ What type of media contains │
                      │ the installation image?     │
                      │                             │
                      │        Local CD/DVD         │
                      │        Hard drive           │
                      │        NFS directory        │
                      │        URL                  │
                      │                             │
                      │   ┌────┐       ┌──────┐     │
                      │   │ OK │       │ Back │     │
                      │   └────┘       └──────┘     │
                      │                             │
                      │                             │
                      └─────────────────────────────┘



<Tab>/<Alt-Tab> between elements  | <Space> selects | <F12> next screen
```
-----
#### 3. uncheck IPv6
```
欢迎使用Secure Shell Extension（版本：0.17）。
常见问题解答：https://goo.gl/muppJj（按住 Ctrl 键的同时点击链接即可打开）
                    
                 ┌────────────┤ Configure TCP/IP ├────────────┐
                 │                                            │
                 │ [*] Enable IPv4 support                    │
                 │        (*) Dynamic IP configuration (DHCP) │
                 │        ( ) Manual configuration            │
                 │                                            │
                 │ [ ] Enable IPv6 support                    │
                 │        (*) Automatic                       │
                 │        ( ) Automatic, DHCP only            │
                 │        ( ) Manual configuration            │
                 │                                            │
                 │        ┌────┐              ┌──────┐        │
                 │        │ OK │              │ Back │        │
                 │        └────┘              └──────┘        │
                 │                                            │
                 │                                            │
                 └────────────────────────────────────────────┘
                    
```
-----

#### 4. https://mirrors.aliyun.com/centos/6.10/os/x86_64/
```
欢迎使用Secure Shell Extension（版本：0.17）。
常见问题解答：https://goo.gl/muppJj（按住 Ctrl 键的同时点击链接即可打开）
        ┌────────────────────────┤ URL Setup ├─────────────────────────┐
        │                                                              │
        │          Please enter the URL containing the CentOS          │
        │          installation image on your server.                  │
        │                                                              │
        │ https://mirrors.aliyun.com/centos/6.10/os/x86_64/___________ │
        │                                                              │
        │ [ ] Enable HTTP proxy                                        │
        │                                                              │
        │ Proxy URL        ___________________________________         │
        │ Username         _______________                             │
        │                                                              │
        │ Password         _______________                             │
        │                                                              │
        │            ┌────┐                       ┌──────┐             │
        │            │ OK │                       │ Back │             │
        │            └────┘                       └──────┘             │
        │                                                              │
        │                                                              │
        └──────────────────────────────────────────────────────────────┘

```
-----

#### 5. Re-initialize all
```
Welcome to CentOS for x86_64
 ┌────────────────────────────────┤ Warning ├─────────────────────────────────┐
 │                                                                            │
 │         Error processing drive:                                 ↑          │
 │                                                                 ▮          │
 │         xen-vbd-51712                                           ▒          │
 │         122880MB                                                ▒          │
 │         Xen Virtual Block Device                                ▒          │
 │                                                                 ▒          │
 │         This device may need to be reinitialized.               ▒          │
 │                                                                 ▒          │
 │         REINITIALIZING WILL CAUSE ALL DATA TO BE LOST!          ▒          │
 │                                                                 ▒          │
 │         This action may also be applied to all other disks      ▒          │
 │         needing reinitialization.                               ↓          │
 │                                                                            │
 │  ┌────────┐   ┌────────────┐   ┌───────────────┐   ┌───────────────────┐   │
 │  │ Ignore │   │ Ignore all │   │ Re-initialize │   │ Re-initialize all │   │
 │  └────────┘   └────────────┘   └───────────────┘   └───────────────────┘   │
 │                                                                            │
 │                                                                            │
 └────────────────────────────────────────────────────────────────────────────┘

  <Tab>/<Alt-Tab> between elements   |  <Space> selects   |  <F12> next screen

```
-----

#### 6. Asia/Shanghai   
```
Welcome to CentOS for x86_64
 
 
                    ┌───────┤ Time Zone Selection ├───────┐
                    │                                     │
                    │ In which time zone are you located? │
                    │                                     │
                    │ [*] System clock uses UTC           │
                    │                                     │
                    │  Asia/Sakhalin                   ↑  │
                    │  Asia/Samarkand                  ▒  │
                    │  Asia/Seoul                      ▮  │
                    │  Asia/Shanghai                   ▒  │
                    │  Asia/Singapore                  ↓  │
                    │                                     │
                    │      ┌────┐          ┌──────┐       │
                    │      │ OK │          │ Back │       │
                    │      └────┘          └──────┘       │
                    │                                     │
                    │                                     │
                    └─────────────────────────────────────┘
 

  <Tab>/<Alt-Tab> between elements   |  <Space> selects   |  <F12> next screen
```
-----

#### 7. 111111
```
Welcome to CentOS for x86_64
 
 
                    
                ┌──────────────┤ Root Password ├───────────────┐
                │                                              │
                │    Pick a root password. You must type it    │
                │    twice to ensure you know it and do not    │
                │    make a typing mistake.                    │
                │                                              │
                │ Password:           ******__________________ │
                │ Password (confirm): ******__________________ │
                │                                              │
                │        ┌────┐               ┌──────┐         │
                │        │ OK │               │ Back │         │
                │        └────┘               └──────┘         │
                │                                              │
                │                                              │
                └──────────────────────────────────────────────┘
                    
                    
 

  <Tab>/<Alt-Tab> between elements   |  <Space> selects   |  <F12> next screen
```
-----

#### 8. Replace existing Linux system 
```
Welcome to CentOS for x86_64
 
       ┌─────────────────────┤ Partitioning Type ├─────────────────────┐
       │                                                               │
       │ Installation requires partitioning of your hard drive.  The   │
       │ default layout is suitable for most users.  Select what space │
       │ to use and which drives to use as the install target.         │
       │                                                               │
       │                 Use entire drive                              │
       │                 Replace existing Linux system                 │
       │                 Use free space                                │
       │                                                               │
       │   Which drive(s) do you want to use for this installation?    │
       │      [*]   xvda   122880 MB (Xen Virtual Block Devic) ↑       │
       │                                                       ▮       │
       │                                                               │
       │                      ┌────┐   ┌──────┐                        │
       │                      │ OK │   │ Back │                        │
       │                      └────┘   └──────┘                        │
       │                                                               │
       │                                                               │
       └───────────────────────────────────────────────────────────────┘

<Space>,<+>,<-> selection   |   <F2> Add drive   |   <F12> next screen
```
-----

#### 9. 安装后，重启前，修改配置文件
- bootloader = 'pygrub' ，否则重启后会再次进行安装系统
```diff
[root@localhost ~]# grep -v ^# !$
grep -v ^# /etc/xen/centos_conf

  name = "centos-001"
- kernel = "/images/kernel/vmlinuz"
+ #kernel = "/images/kernel/vmlinuz"
- ramdisk = "/images/kernel/initrd.img"
+ #ramdisk = "/images/kernel/initrd.img"
  extra = ""
  memory = 512
  vcpus = 2
  vif = [ 'bridge=xenbr0' ]
  disk = [ '/images/xen/centos6.10.img,qcow2,xvda,rw' ]
+ bootloader = 'pygrub'            # /usr/bin/pygrub
```

#### 10. 重启后进入虚拟机
- ```xl console centos-001```
- 配置网卡
  ```
  [root@localhost ~]# ifconfig eth0 192.168.0.20 up
  [root@localhost ~]# ping 192.168.0.18
  PING 192.168.0.18 (192.168.0.18) 56(84) bytes of data.
  64 bytes from 192.168.0.18: icmp_seq=1 ttl=64 time=0.776 ms
  64 bytes from 192.168.0.18: icmp_seq=2 ttl=64 time=0.529 ms
  ```
# 2. 基于ks文件的无人值守安装centos6

> centos6.10宿主机上，搭建vsfrpd，tftp，system-config-kickstart创建ks.cfg文件

## 2.1 磁盘文件
- ```qemu-img create -f qcow2 -o size=120G,preallocation=metedata /images/xen/ks-centos.img```

## 2.2 配置文件
```diff
[root@localhost ~]# cd /etc/xen/
[root@localhost xen]# vim kscentos_conf 

  name = "centos-ks-001"
  kernel = "/images/kernel/vmlinuz"
  ramdisk = "/images/kernel/initrd.img"
  extra = "ks=ftp://192.168.0.15/ks.cfg"
- memory = 512
+ memory = 2048
  vcpus = 2
  vif = [ 'bridge=xenbr0' ]
  disk = [ '/images/xen/ks-centos.img,qcow2,xvda,rw' ]
+ on_reboot = 'destroy'
```
- PXE安装时。报错anaconda 磁盘空间不足。因此扩大内存至2018M
- on_reboot = 'destroy'，以免安装完重启后，自动再次自行系统安装，反复循环。安装完全结束后destroy会自动删除虚拟机，但是安装好的完整磁盘文件还在
## 2.3 创建虚拟机

- ```xl create /etc/xen/kscentos_conf -c```
  - 首次创建虚拟机，只是为了创建一个可以运行centos6的完整磁盘镜像
  
## 2.4 安装完，虚拟机会被自动删除，更改配置文件
```diff
[root@localhost ~]# vim /etc/xen/kscentos_conf 
  name = "centos-ks-001"
- kernel = "/images/kernel/vmlinuz"
- ramdisk = "/images/kernel/initrd.img"
- extra = "ks=ftp://192.168.0.15/ks.cfg"
  memory = 2048
  vcpus = 2
  vif = [ 'bridge=xenbr0' ]
  disk = [ '/images/xen/ks-centos.img,qcow2,xvda,rw' ]
- on_reboot = 'destroy'
+ bootloader = 'pygrub'
```
- 使用镜像内部的文件，自行进行系统引导

## 2.5 再次创建虚拟机

- ```xl create /etc/xen/kscentos_conf -c```
  - 此时可以从刚才创建好的完整系统镜像中启动centos6
  - ```ifconfig eth0 192.168.0.165```，配置网络，可以ping通宿主机




































<p align='center'> | </p>
<p align='center'> | </p>
<p align='center'> | </p>
<p align='center'> | </p>
<p align='center'> | </p>

-----
# 附录：创建PXE安装kickstart ks.cfg文件
## 1. 安装启动ftp，tftp
#### 1. vsftpd
- ```yum install vsftpd -y```
- ```service vsftpd start```
#### 2. tftp
- ```yum install tftp tftp-server xinetd -y```
- 修改xinetd服务配置文件
  ```diff
  [root@localhost ~]# vim /etc/xinetd.d/tftp 

     1 # default: off
     2 # description: The tftp server serves files using the trivial file transfer \
     3 #       protocol.  The tftp protocol is often used to boot diskless \
     4 #       workstations, download configuration files to network-aware printers, \
     5 #       and to start the installation process for some operating systems.
     6 service tftp
     7 {
     8         socket_type             = dgram
     9         protocol                = udp
    10         wait                    = yes
    11         user                    = root
    12         server                  = /usr/sbin/in.tftpd
  - 13         server_args             = -s /var/lib/tftpboot
  + 13         server_args             = -s /tftpboot        # 并不存在，需要自行创建
  - 14         disable                 = yes
  + 14         disable                 = no
    15         per_source              = 11
    16         cps                     = 100 2
    17         flags                   = IPv4

  [root@localhost ~]# systemctl start xinetd          # 启动服务
  [root@localhost ~]# lsof -i:69
  COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
  xinetd  9102 root    5u  IPv4  51445      0t0  UDP *:tftp 
  ```
#### 3. 配置PXE启动所需的相关文件
- 安装服务
[root@localhost ~]# yum -y install system-config-kickstart && syslinux
##### 如果syslinux安装不成功，需要单独在yum install -y syslinux安装一下
- 将tftp需要共享出去的文件，存放点tftp根目录/tftpboot
  - ```wget https://mirrors.aliyun.com/centos/6.10/os/x86_64/isolinux/initrd.img ```，不挂在光盘，直接下载也可以
  - ```wget https://mirrors.aliyun.com/centos/6.10/os/x86_64/isolinux/vmlinuz  ```
  ```
  [root@localhost ~]# mkdir /tftpboot
  [root@localhost ~]# mkdir /tftpboot/pxelinux.cfg
  [root@localhost ~]# cp /usr/share/syslinux/pxelinux.0 /tftpboot/    # 只有安装了syslinux软件包，才会有/usr/share/syslinux/目录及
  [root@localhost ~]# cp /mnt/images/pxeboot/initrd.img  /tftpboot/
  [root@localhost ~]# cp /mnt/images/pxeboot/vmlinuz  /tftpboot/
  [root@localhost ~]# cp /mnt/isolinux/isolinux.cfg  /tftpboot/pxelinux.cfg/default
  [root@localhost ~]# chmod 644  /tftpboot/pxelinux.cfg/default
  [root@localhost ~]# tree /tftpboot/
  /tftpboot/
  |-- initrd.img
  |-- pxelinux.0    
  |-- vmlinuz
  |-- pxelinux.cfg
      `-- default
  ```   

## 2. 制作ks.cfg注意事项
#### 1. no VMNET,
- use pysical network 192.168.0.15，此处不能参考教程设置为虚拟网络，需要配置在物理网络中
#### 2. ext4 
- ```Bootable partitions cannot be on an xfs filesystem. ```
- 在centos7中制作kickstart文件时，默认磁盘分区为xfs，需要改成ext4，因为centos6不支持xfs
- 或者直接在centos6下制作kickstart文件，默认就是ext4文件系统
#### 3. 创建分区时，根分区 / = 1，不可以
- 会报错anaconda 没有足够磁盘空间，需要手动指定大小，比如15G，或者15500，确保不超过qcow2总容量
- 安装方式也不要安装开发恐惧development，只安装core，节省磁盘空间，以免报错

![](https://i.loli.net/2019/04/11/5caee4e279592.png)

## 3. CentOS6 ks.cfg文件源代码
```diff
  #platform=x86, AMD64, or Intel EM64T
  #version=DEVEL
  # Firewall configuration
  firewall --disabled
  # Install OS instead of upgrade
  install
  # Use network installation
  url --url="ftp://192.168.0.160/pub"
  # Root password
  rootpw --plaintext 111111
  # System authorization information
  auth  --useshadow  --passalgo=sha512
  # Use graphical install
  graphical
  firstboot --disable
  # System keyboard
  keyboard us
  # System language
  lang en_US
  # SELinux configuration
  selinux --disabled
  # Installation logging level
  logging --level=info
  # Reboot after installation
  reboot
  # System timezone
  timezone  Asia/Shanghai
  # System bootloader configuration
  bootloader --location=mbr
  # Clear the Master Boot Record
  zerombr
  # Partition clearing information
  clearpart --all --initlabel 
  # Disk partitioning information
- part /boot --fstype="xfs" --size=300
+ part /boot --fstype="ext4" --size=500
  part swap --fstype="swap" --size=2000
- part / --fstype="xfs" --size=1
+ part / --fstype="ext4" --size=15500

  %packages
- @development
+ @core

  %end
```
