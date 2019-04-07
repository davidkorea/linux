# Xen基本概念


## 创建磁盘镜像文件qemu-img
- 创建
  - ```qemu-img create -f raw /images/xen/busybox.img 2G```
  - ```qemu-img create -f raw -o size=2G /images/xen/busybox.img```

  - 当使用```ll -h```查看时，2G大小，但是当使用```du -sh busybox.img```时，显示大小为0
    - ```df -h```查看系统中文件的使用情况
    - ```du -sh *```查看当前目录下各个文件及目录占用空间大小

- 不分区，直接格式化
  - ```mke2fs -t ext2 /images/xen/busybox.img```
  - 挂载
    ```mount -o loop /images/xen/busybox.img /mnt```, -o loop??????????
