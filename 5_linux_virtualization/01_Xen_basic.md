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
