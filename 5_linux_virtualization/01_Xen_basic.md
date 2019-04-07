# Xen基本概念


## 创建磁盘镜像文件qemu-img
- 创建
  - ```qemu-img create -f raw /images/xen/busybox.img 2G```
  - ```qemu-img create -f raw -o size=2G /images/xen/busybox.img```

  - 当使用```ll -h```查看时，2G大小，但是当使用```du -sh busybox.img```时，显示大小为0
    - ```df -h```查看系统中文件的使用情况
    - ```du -sh *```查看当前目录下各个文件及目录占用空间大小

- 不分区，直接格式化，作为根文件系统
  - ```mke2fs -t ext2 /images/xen/busybox.img```
  - 挂载
    ```mount -o loop /images/xen/busybox.img /mnt```, -o loop??????????
  - 若要将此busybox.img当作根文件目录，里面需要有etc，proc，usr等目录
    - 自己创建比较麻烦，复制自己当前宿主机等目录也比较慢
    - 所以直接去下载真正的busybox
    - 编译安装busybox，需要安装编译工具```yum -y groupinstall "Development Tools" "Server Platform Development"```
    - 下载busybox tar包，进到busybox解压目录下
      - 编译成静态链格式，即不让其再依赖于其他库。安装一个工具```yum -y install glibc-static```
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

## 准备内核
宿主机自带的系统可以直接拿来用，虽然不能用作dom0，但是domU没有问题。创建软连接
- ```cd /boot```
- ```ln -sv vmlinuz-2.6.32-504...  vmlinuz```
- ```ln -sv initramfs-2.6.32.504...img initramfs.img```
## 创建虚拟机
进入到xen的目录 /etc/xen
- xl.conf， xl命令的通用全局配置文件，和domU的配置无关
- xlexample.hvm，xlwxample.pvlinux 为xl命令的domU创建模版
- ```cp xlexample.pvlinux busybox```,复制一份模版进行编辑
- ```vim busybox```，如下图
  ![](https://i.loli.net/2019/04/07/5ca991fe76a2b.jpg)
- ```xl -v create busybox -n```,busybox为上面的配置文件，此命令并不会真正创建虚拟机，只会输出配置选项供检查使用 
- ```xl -v create busybox```，-v显示详细信息，真正创建虚拟机
- ```xl list``` ，会列出Domain-0 和 刚刚创建的busybox-001虚拟机

创建虚拟机完成，但是刚才并没有配置网卡不能ssh进行远程操作，也没有图形界面，那么怎么进入虚拟机呢？
- ```xl console busybox-001```



