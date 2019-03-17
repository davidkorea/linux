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
### 1.3 KVM 虚拟机组成
一台 KVM 虚拟机由两部分组成:虚拟机配置文件和镜像img
1. 查看配置文件
```
[root@localhost ~]# ll /etc/libvirt/qemu
总用量 16
drwxr-xr-x  2 root root   29 3月  17 17:33 autostart
-rw-------  1 root root 4465 3月  17 22:42 clone_kvm_centos7.xml
-rw-------  1 root root 4449 3月  17 17:15 kvm_centos7.xml
drwx------. 3 root root   42 1月  30 02:34 networks
```
2. 查看远虚拟机和克隆虚拟机配置文件到差别
```
[root@localhost ~]# cd /etc/libvirt/qemu/
[root@localhost qemu]# ls
autostart  clone_kvm_centos7.xml  kvm_centos7.xml  networks
[root@localhost qemu]# vimdiff kvm_centos7.xml clone_kvm_centos7.xml 
还有 2 个文件等待编辑
```
![](https://i.loli.net/2019/03/17/5c8e50c73b6c1.png)

### 1.4 测试克隆机器
打开后，可以正常联网，不用做额外操作
![](https://i.loli.net/2019/03/17/5c8e52eb5f097.png)

之前到版本，开启后查看到到MAC地址和xml配置文件到MAC并不一致，需要修改
- 在 rhel6 下 kvm 克隆后的操作
  - 登录新克隆的虚拟机删除原来的 mac 和 IP 地址，让新克隆的机器可以上网
    ```
    [root@xuegod63 ~]# rm -rf /etc/udev/rules.d/70-persistent-*
    [root@xuegod63 ~]#vim /etc/sysconfig/network-scripts/ifcfg-eth0 #写入以下内容
    ```
    <img src="https://i.loli.net/2019/03/17/5c8e53eb90c5e.png" height="150" width="300">
    
    注: 记得把 ONBOOT="no" 改为: ONBOOT="yes"
    注: 把原配置文件中的 MAC 和 UUID 地址删除，然后修改一个和原虚拟机不一样的 IP，reboot #重启生效
  - 方法 2
    ```
    [root@xuegod63 ~]# start_udev # 重新启劢 udev 服务，自劢生成刚删除的/etc/udev/rules.d/70-persistent-*文件
                                  # 新生成的 udev 文件，会使用新系统的 MAC 地址。 
    [root@xuegod63 ~]# service network restart
    ```
# 2. 虚拟机常用镜像格式对比

目前主要虚拟机的镜像格式:raw，cow， qcow，qcow2，vmdk 

### 2.1 raw 格式镜像
- raw:老牌的镜像格式，用一个字来说就是裸，也就是赤裸裸，你随便 dd 一个 file 就模拟了一个 raw 格式的镜像。由于裸的彻底，性能上来说的话还是丌错的。
- centos6 上 KVM 和 XEN 默认的格式还是这 个格式。centos7 以上默认是 qcow2 。
- 裸的好处还有就是简单，支持转换成其它格式的虚拟机镜像对裸露的它来说还是很简单的(如果其它 格式需要转换，有时候还是需要它做为中间格式)，空间使用来看，这个很像磁盘，使用多少就是多少(du -h 看到的大小就是使用大小)。
- 之前 qcow2 转为 vmdk 方法是: qcow2 转为 raw ，然后把 raw 转为 vmdk 。也可以直接 qcow2 转为 vmdk
- 扩展: 佳能相机上的高保真用的就是这种 raw 格式。RAW 的原意就是“未经加工”。可以理解为:RAW 图像就是 CMOS 或者 CCD 图像感应器将捕捉到的光源信号转化为数字信号的原始数据。RAW 理解为“数 字底片”
- 缺点:不支持 snapshot 快照。

### 2.2 cow、qcow、qcow2 格式
1. cow 格式:还没有成熟，就被放弃了。后来被 qcow 格式所取代。
2. qcow 格式:刚刚出现的时候有比较好的特性，但其性能和 raw 格式对比还是有很大的差距，目 前已经被新版本的 qcow2 取代。
3. qcow2 格式:现在比较主流的一种虚拟化镜像格式，经过一代的优化，目前 qcow2 的性能上接近 raw 裸格式的性能

- qcow2 格式支持 snapshot，可以在镜像上做 N 多个快照，具有以下优点:
  - 更小的存储空间
  - 支持创建 image 镜像
  - 支持多个 snapshot，对历史snapshot 进行管理， 支持 zlib 的磁盘压缩
  - 支持 AES 的加密

### 2.3 vmdk 格式:
VMware 的格式，整体性能最好，因为原本 VMware 就是做虚拟化起家。从性能和功能上来说，vmdk应该算最出色的，由于 vmdk 结合了 VMware 的很多能力，目前来看，KVM 和 XEN 使用这种格式的情况不是太多。但就 VMware 的企业级虚拟化 Esxi 来看，它的稳定性和各方面的能力都很好


# 3. KVM 虚拟机快照功能使用方法
- 快照的作用:1、热备 2、灾难恢复 3、回滚到历中的某个状态
- 原理
  1. 创建快照之前到磁盘空间，不再进行任何改动
  2. 之后到任何操作都在磁盘分区到另一块空间进行
  3. 恢复快照，就是恢复到创建快照时到那一块空间
- kvm 快照，分两种:
  - 方法 1:使用 lvm 快照，如果分区是 lvm，可以利用 lvm 迚行 kvm 的快照备份 
  - 方法 2:使用 qcow2 格式的镜像创建快照
## 3.1 创建 KVM 快照
**要使用快照功能，磁盘格式必须为 qcow2。** 在 centos6 下，kvm 虚拟机默认使用 raw 格式的镜像格式，性能最好，速度最快，它的缺点就是不支持一些新的功能，如支持镜像,zlib 磁盘压缩,AES 加密等。
#### 1. 查看磁盘格式
虽然clone 到时候，镜像命名为img，但实际上还是qcow2到格式
```
[root@localhost ~]# qemu-img info /var/lib/libvirt/images/clone_kvm_centos7.img 
image: /var/lib/libvirt/images/clone_kvm_centos7.img
file format: qcow2
virtual size: 10G (10737418240 bytes)
disk size: 1.2G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: true
```
#### 2. 创建快照
创建快照时丌需要关闭虚拟机，关机创建快照比较快，开机创建快照需要把内存中的内容写到磁盘上，记录虚拟机这一时刻的状态。

- 语法: virsh snapshot-create 虚拟机的名字
```
[root@localhost ~]# virsh snapshot-create clone_kvm_centos7 
已生成域快照 1552832632
```
- 创建快照时启个名字, 语法:virsh snapshot-create-as KVM 虚拟机名 快照名
```
[root@localhost ~]# virsh snapshot-create-as clone_kvm_centos7 snapshot_1
已生成域快照 snapshot_1
```
- 比较开关机状态下，快照文件到大小。关机状态下，快照文件几乎不占用空间
```
[root@localhost ~]# virsh snapshot-create-as clone_kvm_centos7 snapshow_close
已生成域快照 snapshow_close
[root@localhost ~]# qemu-img info /var/lib/libvirt/images/clone_kvm_centos7.img 
image: /var/lib/libvirt/images/clone_kvm_centos7.img
file format: qcow2
virtual size: 10G (10737418240 bytes)
disk size: 1.6G
cluster_size: 65536
Snapshot list:
ID        TAG                 VM SIZE                DATE       VM CLOCK
2         snapshot_1             311M 2019-03-17 23:26:13   00:32:19.200
3         snapshow_close            0 2019-03-17 23:39:45   00:00:00.000
Format specific information:
    compat: 1.1
    lazy refcounts: true
```

#### 3. 查看虚拟机镜像快照列表
- 语法: virsh snapshot-list 虚拟机的名字
```
[root@localhost ~]# virsh snapshot-list clone_kvm_centos7 
 名称               生成时间              状态
------------------------------------------------------------
 1552832632           2019-03-17 23:23:52 +0900 running
 snapshot_1           2019-03-17 23:26:13 +0900 running
```
- 查看最近一次使用的快照版本
```
[root@localhost ~]# virsh snapshot-current clone_kvm_centos7 
<domainsnapshot>
  <name>snapshot_1</name>
  <state>running</state>
  <parent>
    <name>1552832632</name>
  </parent>
... ...
```

#### 4. 快照配置文件在/var/lib/libvirt/qemu/snapshot/虚拟机名称下
```
[root@localhost ~]# ll -h /var/lib/libvirt/qemu/snapshot/clone_kvm_centos7/
总用量 16K
-rw------- 1 root root 5.4K 3月  17 23:26 1552832632.xml
-rw------- 1 root root 5.5K 3月  17 23:26 snapshot_1.xml
```

## 3.2 恢复虚拟机快照

恢复虚拟机快照必须关闭虚拟机。注:阿里云也需要关闭后再恢复快照

```
[root@localhost ~]# virsh domstate clone_kvm_centos7      # 确认已经关闭
关闭

[root@localhost ~]# virsh snapshot-revert clone_kvm_centos7 snapshot_1 
```

## 3.3 删除快照
```
[root@localhost ~]# virsh snapshot-list clone_kvm_centos7               # 查看有哪些快照
 名称               生成时间              状态
------------------------------------------------------------
 1552832632           2019-03-17 23:23:52 +0900 running
 snapshot_1           2019-03-17 23:26:13 +0900 running

[root@localhost ~]# virsh snapshot-delete clone_kvm_centos7 1552832632  # 删除快照
已删除域快照 1552832632

[root@localhost ~]# qemu-img info /var/lib/libvirt/images/clone_kvm_centos7.img 
image: /var/lib/libvirt/images/clone_kvm_centos7.img
file format: qcow2
virtual size: 10G (10737418240 bytes)
disk size: 1.5G
cluster_size: 65536
Snapshot list:
ID        TAG                 VM SIZE                DATE       VM CLOCK
2         snapshot_1             311M 2019-03-17 23:26:13   00:32:19.200
Format specific information:
    compat: 1.1
    lazy refcounts: true
```
# 4. virsh 常用命令
- virsh list
- virsh list --all
- virsh version
- virsh start xuegod63-kvm2
- virsh shutdown xuegod63-kvm2 //关机 xuegod63-kvm2 虚拟机
- virsh dumpxml xuegod63-kvm2 > xuegod63-kvm2.xml //导出 xuegod63-kvm2 虚拟机配置文件
- virsh undefine centos7.0 //取消 centos7.0 定义 域 centos7.0 已经被取消定义
- virsh define /opt/centos7.0   
- virsh destroy xuegod63-kvm2 //强制关闭 xuegod63-kvm2 虚拟机。正常关不了机时，用这个
- virsh autostart centos7.0 //设置开机自启劢 node1。 
- virsh autostart --disable vm1 #取消虚拟机随宿主机开机自启
- virsh suspend vm1 #挂起虚拟机
- virsh resume vm1  #恢复虚拟机
- virsh console vm1   #控制台管理虚拟机

# 5. 实战: 虚拟机镜像文件格式转换
## 5.1 qcow2 格式转换成 raw
qemu-img -f qcow2 -O raw 源文件 转换后文件名字
- f 源镜像的格式
- O 目标镜像的格式
```
[root@localhost ~]# qemu-img convert -f qcow2 -O raw /var/lib/libvirt/images/clone_kvm_centos7.img /var/lib/libvirt/images/clone_kvm_centos7.raw

[root@localhost ~]# qemu-img info /var/lib/libvirt/images/clone_kvm_centos7.raw 
image: /var/lib/libvirt/images/clone_kvm_centos7.raw
file format: raw
virtual size: 10G (10737418240 bytes)
disk size: 1.2G
```

## 5.2 其他镜像格式转换方法:
- 将 vmdk 转换为 qcow2
```
qemu-img convert -f vmdk -O qcow2 source-name.vmdk target-name.qcow2
```
- 将 qcow2 转换为 vmdk
```
qemu-img convert -f qcow2 -O vmdk source-name.qcow2 target-name.vmdk
```

## 5.3 修改虚拟机配置文件，使用 raw 格式镜像文件，来启劢虚拟机
- 方法一
  ```
  [root@localhost ~]# virsh edit clone_kvm_centos7 
  """
    38       <driver name='qemu' type='qcow2'/>
    39       <source file='/var/lib/libvirt/images/clone_kvm_centos7.img'/>
  """
    38       <driver name='qemu' type='raw'/>                                   # 把上面到改成下面的
    39       <source file='/var/lib/libvirt/images/clone_kvm_centos7.raw'/>

  ```
- 方法二
  - vim 直接编辑配置文件/etc/libvirt/qemu/xuegod63-kvm2.xml 不生效 修改后，需要重启服务
    ```
    [root@localhost ~]# /etc/init.d/libvirtd restart
    ```
