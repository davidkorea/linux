
# 1.xl创建centos6虚拟机
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
