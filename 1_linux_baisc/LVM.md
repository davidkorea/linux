# LVM管理和ssm存储管理器使用

1. LVM的工作原理
2. 创建LVM的基本步骤
3. 实战-使用SSM工具为公司的邮件服务器创建可动态扩容的存储池

> 实战场景：对于生产环境下的服务器来说,如果存储数据的分区磁盘空间不够了怎么办?
> ![](https://i.loli.net/2019/02/28/5c7778440fe30.png)
> 
>答：只能换一个更大的磁盘。 如果用了一段时间后， 空间又不够了，怎么办？再加一块更大的？换磁盘的过程中，还需要把数据从一个硬盘复制到另一个硬盘，过程太慢了。
> 
> 解决方案：使用LVM在线动态扩容


# 1. LVM的工作原理
  LVM（ Logical Volume Manager）逻辑卷管理，是在磁盘分区和文件系统之间添加的一个逻辑层，为文件系统屏蔽下层磁盘分区布局，提供一个抽象的盘卷，在盘卷上建立文件系统。
  
  管理员利用LVM可以在磁盘不用重新分区的情况下动态调整文件系统的大小，并且利用LVM管理的文件系统可以跨越磁盘，当服务器添加了新的磁盘后，管理员不必将原有的文件移动到新的磁盘上，而是通过LVM可以直接扩展文件系统跨越磁盘。
  
  它就是通过将底层的物理硬盘封装起来，然后以逻辑卷的方式呈现给上层应用。在LVM中，其通过对底层的硬盘进行封装，当我们对底层的物理硬盘进行操作时，其不再是针对于分区进行操作，而是通过一个叫做逻辑卷的东西来对其进行底层的磁盘管理操作。

## 1.1 LVM常用的术语
- 物理存储介质（The physical media）:LVM存储介质可以是整个磁盘，磁盘分区，RAID阵列或SAN磁盘，设备必须初始化为LVM物理卷，才能与LVM结合使用 
- 物理卷PV（physical volume）：物理卷就是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数,创建物理卷它可以用硬盘分区，也可以用硬盘本身
  - PE（physical extents）：PV物理卷中可以分配的最小存储单元，PE的大小是可以指定的，默认为4MB
- 卷组VG（Volume Group）：一个LVM卷组由一个或多个物理卷组成
- 逻辑卷LV（logical volume）：LV建立在VG之上，可以在LV之上建立文件系统
  - LE（logical extent）：LV逻辑卷中可以分配的最小存储单元，在同一个卷组中，LE的大小和PE是相同的，并且一一对应
  
最小存储单位总结：

|名称|     	最小存储单位|命令|
|-|-|-|
|硬盘    |扇区（512字节）|    
|文件系统 |block（1K或4K）| mkfs.ext4  -b 2048  /dev/sdb1  ，最大支持到4096 |
|raid    | 	chunk（512K）| mdadm -C -v /dev/md5 -l 5 -n 3 -c 512 -x 1 /dev/sde{1,2,3,5}|
|LVM     |	PE (4M）| vgcreate -s 4M  vg1 /dev/sdb{1,2}|

- LVM主要元素构成
![](https://i.loli.net/2019/02/28/5c777a64596e2.png)

## 1.2 LVM优点
- 使用卷组，使多个硬盘空间看起来像是一个大的硬盘
- 使用逻辑卷，可以跨多个硬盘空间的分区  sdb1 sdb2  sdc1  sdd2  sdf
- 在使用逻辑卷时，它可以在空间不足时动态调整它的大小
- 在调整逻辑卷大小时，不需要考虑逻辑卷在硬盘上的位置，不用担心没有可用的连续空间
- 可以在线对LV,VG 进行创建，删除，调整大小等操作。LVM上的文件系统也需要重新调整大小。
- 允许创建快照，可以用来保存文件系统的备份。

**RAID+LVM一起用：LVM是软件的卷管理方式，而RAID是磁盘管理的方法。对于重要的数据，使用RAID来保护物理的磁盘不会因为故障而中断业务，再用LVM用来实现对卷的良性的管理，更好的利用磁盘资源。**

# 2. 创建LVM的基本步骤

1. 物理磁盘被格式化为PV，(空间被划分为一个个的PE) #PV包含PE
2. 不同的PV加入到同一个VG中，(不同PV的PE全部进入到了VG的PE池内) #VG包含PV
3. 在VG中创建LV逻辑卷，基于PE创建，(组成LV的PE可能来自不同的物理磁盘) #LV基于PE创建
4. LV直接可以格式化后挂载使用  #格式化挂载使用
5. LV的扩充缩减实际上就是增加或减少组成该LV的PE数量，其过程不会丢失原始数据

## 2.1 lvm常用的命令

- 创建命令

|功能|PV管理命令|VG管理命令|LV管理命令|
|-|-|-|-|
|scan 扫描|pvscan|vgscan|lvscan|
|create 创建|pvcreate|vgcreate|lvcreate|
|display显示|pvdisplay|vgdisplay|lvdisplay|
|remove 移除|pvremove|vgremove|lvremove|
|extend 扩展| |vgextend|lvextend|
|reduce减少| |vgreduce|lvreduce|

- 查看命令

|查看卷名|简单对应卷信息的查看|扫描相关的所有的对应卷|详细对应卷信息的查看|
|-|-|-|-|
|物理卷|pvs|pvscan|pvdisplay|
|卷组|vgs|vgscan|vgdisplay|
|逻辑卷|lvs|lvscan|lvdisplay|

## 2.2 创建并使用LVM逻辑卷

### 1. 创建PV
```
[root@localhost ~]# fdisk /dev/sdb    # 划分分区sdb1~sdb4，主分区，扩展分区皆可

[root@localhost ~]# pvcreate /dev/sdb[1,2,3,4]    # sdb{1,2,3,4}
WARNING: xfs signature detected on /dev/sdb1 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/sdb1.
WARNING: xfs signature detected on /dev/sdb2 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/sdb2.
WARNING: xfs signature detected on /dev/sdb3 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/sdb3.
WARNING: dos signature detected on /dev/sdb4 at offset 510. Wipe it? [y/n]: y
  Wiping dos signature on /dev/sdb4.
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdb2" successfully created.
  Physical volume "/dev/sdb3" successfully created.
  Physical volume "/dev/sdb4" successfully created.

[root@localhost ~]# pvs   # 简单查看
  PV         VG Fmt  Attr PSize PFree
  /dev/sdb1     lvm2 ---  1.00g 1.00g
  /dev/sdb2     lvm2 ---  1.00g 1.00g
  /dev/sdb3     lvm2 ---  1.00g 1.00g
  /dev/sdb4     lvm2 ---  1.00g 1.00g

[root@localhost ~]# pvdisplay     # 详细查看
  "/dev/sdb2" is a new physical volume of "1.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb2
  VG Name               
  PV Size               1.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               4IkMNT-qLFY-79Pf-0vVM-sFEb-2E2g-i3dAZp

... ...
```
### 2. 创建VG

- ```vgcreate  vg名字  pv的名字   (可以跟多个pv)```

```
[root@localhost ~]# vgcreate vg01 /dev/sdb1     # 只将sdb1一个物理卷pv，加入vg01群组中
  Volume group "vg01" successfully created
  
[root@localhost ~]# vgs
  VG   #PV #LV #SN Attr   VSize    VFree   
  vg01   1   0   0 wz--n- 1020.00m 1020.00m
  
[root@localhost ~]# vgdisplay 
  --- Volume group ---
  VG Name               vg01
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1020.00 MiB
  PE Size               4.00 MiB
  Total PE              255
  Alloc PE / Size       0 / 0   
  Free  PE / Size       255 / 1020.00 MiB
  VG UUID               kKFi1y-EvPW-gw6N-ONJg-FcPl-hJeP-11OcMW
```
### 3. 创建LV
- 创建```lvcreate -n 指定新逻辑卷的名称  -L指定lv大小的SIZE(M,G) （-l：小l 指定LE的数量） vgname```
```
[root@localhost ~]# lvcreate -n lv01 -L 16M vg01 
  Logical volume "lv01" created.
```
- 查看lv01详情
```
[root@localhost ~]# vgdisplay 
  --- Volume group ---
  VG Name               vg01
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1020.00 MiB
  PE Size               4.00 MiB
  Total PE              255                       # 一共1G，255个PE
  Alloc PE / Size       4 / 16.00 MiB             # 已使用4个PE，共16M
  Free  PE / Size       251 / 1004.00 MiB
  VG UUID               kKFi1y-EvPW-gw6N-ONJg-FcPl-hJeP-11OcMW
  
  [root@localhost ~]# pvdisplay                   # 创建的16M的逻辑卷实际存放在sdb1分区，或者叫sdb1物理卷PV
  --- Physical volume ---
  PV Name               /dev/sdb1
  VG Name               vg01
  PV Size               1.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              255
  Free PE               251
  Allocated PE          4
  PV UUID               cAr4cb-kB11-HiqQ-W6oG-QK52-kqCc-xSaUXN
```






























































