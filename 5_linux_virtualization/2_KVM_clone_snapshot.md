# 第二章 KVM 虚拟机克隆和快照

1. KVM 虚拟机克隆方法
2. 虚拟机常用镜像格式对比
3. KVM 虚拟机快照功能使用方法
4. virsh 命令常见用法
5. KVM 常用镜像格式转换

# 1. KVM 虚拟机克隆

### 1.1 KVM虚拟机镜像

虚拟机镜像: 就是整个虚拟机文件。 不是操作系统光盘镜像 rhel6.5.iso
```
[root@localhost ~]# cd /var/lib/libvirt/images/
[root@localhost images]# ls
CentOS-7-x86_64-DVD-1810.iso  kvm_centos7.qcow2

[root@localhost images]# ll -h
总用量 15G
-rw-r--r-- 1 qemu qemu 4.3G 2月  20 00:04 CentOS-7-x86_64-DVD-1810.iso
-rw------- 1 qemu qemu  11G 3月  17 22:19 kvm_centos7.qcow2
```

### 1.2 基于 centos7.0 克隆一台虚拟机

1. 克隆前，centos7.0 需要提前关机
2. 语法:virt-clone -o 原虚拟机 -n 新虚拟机 -f 新虚拟机镜像存放路径 选项:-o old -n new
```
[root@localhost ~]# virt-clone -o kvm_centos7 -n clone_kvm_centos7 -f /var/lib/libvirt/images/clone_kvm_centos7.img
正在分配 'clone_kvm_ce 18% [===              ]  43 MB/s | 1.9 GB  03:14 ETA 
```
3. 查看克隆后到镜像文件大小
```
[root@localhost images]# ll -h
总用量 12G
-rw------- 1 root root 1.3G 3月  17 22:42 clone_kvm_centos7.img    # 只有1.3G
-rwx------ 1 root root  11G 3月  17 22:39 kvm_centos7.qcow2
```
