
# 1.xl全手动创建centos6虚拟机
## 1.1 准备内核文件
- yum/cd中的文件
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
  - wget https://mirrors.aliyun.com/centos/6.10/os/x86_64/isolinux/initrd.img 
  - wget https://mirrors.aliyun.com/centos/6.10/os/x86_64/isolinux/vmlinuz  
- mkdir /images/kernel
- mv initrd.img vmlinuz /images/kernel
## 1.3 创建磁盘镜像qemu-img
-  qemu-img create -f qcow2 -o size=120G,preallocation=metedata /images/xen/centos6.10.img
-  df -lh查看物理磁盘空间够不够

## 1.3 准备虚拟机配置文件
- cp /etc/xen/busybox_conf centos_conf
- vim /etc/xen/centos_ conf
  ```
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
- xl create /etc/xen/centos_conf
- xl console centos-001
-----
1. English
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
  2. URL
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
3. uncheck IPv6
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
4. https://mirrors.aliyun.com/centos/6.10/os/x86_64/
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
```
Welcome to CentOS for x86_64





               ┌──────────────────┤ CentOS ├───────────────────┐
               │                                               │
               │ Welcome to CentOS!                            │
               │                                               │
               │                                               │
               │                    ┌────┐                     │
               │                    │ OK │                     │
               │                    └────┘                     │
               │                                               │
               │                                               │
               └───────────────────────────────────────────────┘






  <Tab>/<Alt-Tab> between elements   |  <Space> selects   |  <F12> next screen
```
-----
5. Re-initialize all
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
6. Asia/Shanghai   
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
7. 111111
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
8. Replace existing Linux system 
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
9. 安装后，重启前，修改配置文件
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
+ bootloader = 'pygrub'
```
10. 重启后进入虚拟机
- xl console centos-001
- 配置网卡
  ```
  [root@localhost ~]# ifconfig eth0 192.168.0.20 up
  [root@localhost ~]# ping 192.168.0.18
  PING 192.168.0.18 (192.168.0.18) 56(84) bytes of data.
  64 bytes from 192.168.0.18: icmp_seq=1 ttl=64 time=0.776 ms
  64 bytes from 192.168.0.18: icmp_seq=2 ttl=64 time=0.529 ms
  ```
# 2. 基于ks文件的无人值守安装centos








