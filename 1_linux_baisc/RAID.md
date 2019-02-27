# RAID磁盘阵列的原理与搭建

1. RAID概念-企业级RAID 0, 1,5,10的工作原理
2. RAID-0-1-5-10搭建及使用-删除RAID及注意事项
3. 实战：企业中硬件raid5的配置

# 1 RAID概念
- 磁盘阵列（Redundant Arrays of Independent Disks，RAID），有“独立磁盘构成的具有冗余能力的阵列”之意。 磁盘阵列是由很多价格较便宜的磁盘，以硬件（RAID卡）或软件（MDADM）形式组合成一个容量巨大的磁盘组，利用多个磁盘组合在一起，提升整个磁盘系统效能。利用这项技术，将数据切割成许多区段，分别存放在各个硬盘上。 磁盘阵列还能利用同位检查（Parity Check）的观念，在数组中任意一个硬盘故障时，仍可读出数据，在数据重构时，将数据经计算后重新置入新硬盘中
- 注：RAID可以预防数据丢失，但是它并不能完全保证你的数据不会丢失，所以大家使用RAID的同时还是注意备份重要的数据
- RAID的创建有两种方式：软RAID（通过操作系统软件来实现）和硬RAID（使用硬件阵列卡）；在企业中用的最多的是：raid1、raid5和raid10。不过随着云的高速发展，供应商一般可以把硬件问题解决掉。

## 1.1 RAID几种常见的类型

  |级 别|说 明|最低磁盘个数|空间利用率|各自的优缺点|
  |-|-|-|-|-|
  |RAID0|条带卷|2+|100%|读写速度快，不容错|
  |RAID1|镜像卷|2|50%|读写速度一般，容错|
  |RAID5|带奇偶校验的条带卷|3+|(n-1)/n|读写速度快，容错，允许坏一块盘|
  |RAID6|带奇偶校验的条带集，双校验|4+|(n-2)/n|读写快，容错，允许坏两块盘|
  |RAID10|RAID1的安全+RAID0的高速|4|50%|读写速度快，容错|
  |RAID50|RAID5的安全+RAID0的高速|6|(n-2)/n|读写速度快，容错|

- RAID基本思想：把好几块硬盘通过一定组合方式把它组合起来，成为一个新的硬盘阵列组，从而使它能够达到高性能硬盘的要求
- RAID有三个关键技术：
  - 镜像：提供了数据的安全性；
  - chunk条带（块大小也可以说是条带的粒度），它的存在的就是为了提高I/O，提供了数据并发性
  - 数据的校验：提供了数据的安全

## 1.2 RAID-0的工作原理
- 条带 （strping），也是我们最早出现的RAID模式
- 需磁盘数量:2块以上(大小最好相同)，是组建磁盘阵列中最简单的一种形式，只需要2块以上的硬盘即可.
- 特点:成本低，可以提高整个磁盘的性能和吞吐量。RAID 0没有提供冗余或错误修复能力，速度快.
- 任何一个磁盘的损坏将损坏全部数据；磁盘利用率为100%。

![]()

## .1.3  RAID-1
- mirroring（镜像卷），需要磁盘两块以上
- 原理:是把一个磁盘的数据镜像到另一个磁盘上，也就是说数据在写入一块磁盘的同时，会在另一块闲置的磁盘上生成镜像文件，(同步)
- RAID 1 mirroring（镜像卷），至少需要两块硬盘，raid大小等于两个raid分区中最小的容量（最好将分区大小分为一样），数据有冗余，在存储时同时写入两块硬盘，实现了数据备份；
- 磁盘利用率为50%，即2块100G的磁盘构成RAID1只能提供100G的可用空间。

![]()

## 1.4  RAID-5
- 需要三块或以上硬盘，可以提供热备盘实现故障的恢复；只损坏一块，没有问题。但如果同时损坏两块磁盘，则数据将都会损坏。 
- 空间利用率： (n-1)/n   2/3  如下图所示

![]()

- 奇偶校验信息的作用:当RAID5的一个磁盘数据发生损坏后，利用剩下的数据和相应的奇偶校验信息去恢复被损坏的数据。异或运算,是用相对简单的异或逻辑运算（相同为0，相异为1）

## 1.5  嵌套RAID级别
- RAID-10镜像+条带
- RAID 10是将镜像和条带进行两级组合的RAID级别，第一级是RAID1镜像对，第二级为RAID 0。比如我们有8块盘，它是先两两做镜像，形成了新的4块盘，然后对这4块盘做RAID0；当RAID10有一个硬盘受损其余硬盘会继续工作，这个时候受影响的硬盘只有2块

![]()

## 1.6  RAID硬盘失效处理
一般两种处理方法：热备和热插拔
  - 热备：HotSpare
  - 定义：当冗余的RAID组中某个硬盘失效时，在不干扰当前RAID系统的正常使用的情况下，用RAID系统中另外一个正常的备用硬盘自动顶替失效硬盘，及时保证RAID系统的冗余性
- 全局式：备用硬盘为系统中所有的冗余RAID组共享
- 专用式：备用硬盘为系统中某一组冗余RAID组专用
如下图所示：是一个全局热备的示例，该热备盘由系统中两个RAID组共享，可自动顶替任何一个RAID中的一个失效硬盘

![]()

- 热插拔：HotSwap, 定义：在不影响系统正常运转的情况下，用正常的物理硬盘替换RAID系统中失效硬盘。


# 2. RAID-0-1-5-10搭建及使用-删除RAID及注意事项

## 2.1 RAID的实现方式
>我们做硬件RAID，是在装系统前还是之后？  答：先做阵列才装系统 ，一般服务器启动时，有显示进入配置Riad的提示，比如：按下CTRL+L/H/M进入配置raid界面
- 硬RAID：需要RAID卡，我们的磁盘是接在RAID卡的，由它统一管理和控制。数据也由它来进行分配和维护；它有自己的cpu，处理速度快
- 软RAID：通过操作系统实现。	Linux内核中有一个md(multiple devices)模块在底层管理RAID设备，它会在应用层给我们提供一个应用程序的工具mdadm ，mdadm是linux下用于创建和管理软件RAID的命令。
  - mdadm命令常见参数解释

  |命令|说明|命令|说明|
  |-|-|-|-|
  |-C或--creat|建立一个新阵列|-r|移除设备|
  |-A|激活磁盘阵列|-l 或--level=|设定磁盘阵列的级别|
  |-D或--detail|打印阵列设备的详细信息|-n或--raid-devices=|指定阵列成员（分区/磁盘）的数量|
  |-s或--scan|扫描配置文件或/proc/mdstat得到阵列缺失信息|-x或--spare-devicds=|指定阵列中备用盘的数量|
  |-f|将设备状态定为故障|-c或--chunk=|设定阵列的块chunk块大小 ，单位为KB|
  |-a或--add|添加设备到阵列|-G或--grow|改变阵列大小或形态|
  |-v|--verbose 显示详细信息|-S|停止阵列|

  - 实验环境

  |raid种类|磁盘|热备盘|
  |-|-|-|
  |raid0|sdb、sdc|
  |raid1|sdd、sde|sdf|
  |raid5|sdg、sdh、sdi|sdj|
  |raid10|分区：sdk1,sdk2,sdk3.sdk4|
  
  工作中正常做raid全部是使用独立的磁盘来做的。为了节约资源，raid10以一块磁盘上多个分区来代替多个独立的磁盘做raid，但是这样做出来的raid没有备份数据的作用，因为一块磁盘坏了，这个磁盘上所做的raid也就都坏了。
    

## 2.2 创建RAID0
实验环境：

|raid种类|磁盘|热备盘|
|-|-|-|
|raid0|sdb、sdc|

### 1. 创建md0
  ```
  [root@localhost ~]# mdadm -C -v /dev/md0 -l 0 -n 2 /dev/sdb /dev/sdc
  mdadm: chunk size defaults to 512K
  mdadm: partition table exists on /dev/sdb
  mdadm: partition table exists on /dev/sdb but will be lost or
         meaningless after creating array
  Continue creating array? y
  mdadm: Defaulting to version 1.2 metadata
  mdadm: array /dev/md0 started.
  ```
### 2. 查看并生成配置文件
  ```
  [root@localhost ~]# mdadm -Ds
  ARRAY /dev/md0 metadata=1.2 name=localhost.localdomain:0 UUID=2d41722a:69ed39b5:59a1ef60:40635d1b
  ```
  
  ```
  [root@localhost ~]# mdadm -D /dev/md0 
  /dev/md0:
             Version : 1.2
       Creation Time : Wed Feb 27 13:19:26 2019
          Raid Level : raid0
          Array Size : 41908224 (39.97 GiB 42.91 GB)
        Raid Devices : 2
       Total Devices : 2
         Persistence : Superblock is persistent

         Update Time : Wed Feb 27 13:19:26 2019
               State : clean 
      Active Devices : 2
     Working Devices : 2
      Failed Devices : 0
       Spare Devices : 0

          Chunk Size : 512K

  Consistency Policy : none

                Name : localhost.localdomain:0  (local to host localhost.localdomain)
                UUID : 2d41722a:69ed39b5:59a1ef60:40635d1b
              Events : 0

      Number   Major   Minor   RaidDevice State
         0       8       16        0      active sync   /dev/sdb
         1       8       32        1      active sync   /dev/sdc
  ```
  ```
  [root@localhost ~]# mdadm -Ds > /etc/mdadm.conf   # > 插入并表示覆盖文件原有内容， >>最后追加，不覆盖原有内容
  ```

### 3. 对创建的RAID0进行文件系统创建并挂载
1. 手动挂载

  ```
  [root@localhost ~]# mkfs.xfs /dev/md0 
  meta-data=/dev/md0               isize=512    agcount=16, agsize=654720 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=0, sparse=0
  data     =                       bsize=4096   blocks=10475520, imaxpct=25
           =                       sunit=128    swidth=256 blks
  naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
  log      =internal log           bsize=4096   blocks=5120, version=2
           =                       sectsz=512   sunit=8 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0
  [root@localhost ~]# mkdir /raid0
  [root@localhost ~]# mount /dev/md0 /raid0/

  [root@localhost ~]# df -Th /raid0/    # -T 显示Type
  Filesystem     Type  Size  Used Avail Use% Mounted on
  /dev/md0       xfs    40G   33M   40G   1% /raid0
  ```
2. 开机自动挂载
  ```
  [root@localhost ~]# umount /dev/md0     
  [root@localhost ~]# df -h   # 确认已解除挂载

  [root@localhost ~]# blkid /dev/md0    # 查询blockid
  /dev/md0: UUID="198a4ecc-5d38-4469-bbe2-1aa1fef9f63d" TYPE="xfs" 

  [root@localhost ~]# echo "UUID=198a4ecc-5d38-4469-bbe2-1aa1fef9f63d /raid0 xfs defaults 0 0" >> /etc/fstab 

  [root@localhost ~]# mount -a

  [root@localhost ~]# df -Th /raid0/    # -T 显示Type
  Filesystem      Size  Used Avail Use% Mounted on
  /dev/md0         40G   33M   40G   1% /raid0
  ```
  
## 2.3 RAID1
实验内容：

  |raid种类|磁盘|热备盘|
  |-|-|-|
  |raid1|sdd、sde|sdf|
  
- 创建RAID1
- 添加1个热备盘
- 模拟磁盘故障，自动顶替故障盘
- 从raid1中移出故障盘
- 添加新硬盘至raid1


### 1. 创建
```
[root@localhost ~]# mdadm -C -v /dev/md1 -l 1 -n 2 -x 1 /dev/sd[d,e,f]
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 20954112K
Continue creating array? 
Continue creating array? (y/n) y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```
### 2. 查看并生成配置文件
```
[root@localhost ~]# mdadm -D /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Wed Feb 27 14:08:23 2019
        Raid Level : raid1
        Array Size : 20954112 (19.98 GiB 21.46 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 2
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Wed Feb 27 14:08:33 2019
             State : clean, resyncing 
    Active Devices : 2
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 1

Consistency Policy : resync

     Resync Status : 8% complete    # 完成初始化后，此行会消失

              Name : localhost.localdomain:1  (local to host localhost.localdomain)
              UUID : 58c81516:761836f4:4c1d6f59:602b393c
            Events : 1

    Number   Major   Minor   RaidDevice State
       0       8       48        0      active sync   /dev/sdd
       1       8       64        1      active sync   /dev/sde

       2       8       80        -      spare   /dev/sdf
```

```
[root@localhost ~]# mdadm -Dsv
ARRAY /dev/md0 level=raid0 num-devices=2 metadata=1.2 name=localhost.localdomain:0 UUID=2d41722a:69ed39b5:59a1ef60:40635d1b
   devices=/dev/sdb,/dev/sdc
ARRAY /dev/md1 level=raid1 num-devices=2 metadata=1.2 spares=1 name=localhost.localdomain:1 UUID=58c81516:761836f4:4c1d6f59:602b393c
   devices=/dev/sdd,/dev/sde,/dev/sdf

[root@localhost ~]# mdadm -Dsv > /etc/mdadm.conf    # mdadm -Dsv已经包含来之前创建的md0，所以可以使用>覆盖原有内容
```
### 3. 挂载  
```
[root@localhost ~]# mkfs.xfs /dev/md1
meta-data=/dev/md1               isize=512    agcount=4, agsize=1309632 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=5238528, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost ~]# mkdir /raid1
[root@localhost ~]# mount /dev/md1 /raid1/

[root@localhost ~]# df -Th /raid1
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/md1       xfs    20G   33M   20G   1% /raid1
```
### 4. 故障
1. 模拟故障
```
[root@localhost ~]# touch /raid1/1.txt
[root@localhost ~]# ls /raid1
1.txt         # 提前创建好文件，测试硬盘故障后是否有数据丢失

[root@localhost ~]# mdadm /dev/md1 -f /dev/sde
mdadm: set /dev/sde faulty in /dev/md1
[root@localhost ~]# mdadm -D /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Wed Feb 27 14:08:23 2019
        Raid Level : raid1
        Array Size : 20954112 (19.98 GiB 21.46 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 2
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Wed Feb 27 14:20:25 2019
             State : clean, degraded, recovering 
    Active Devices : 1
   Working Devices : 2
    Failed Devices : 1
     Spare Devices : 1

Consistency Policy : resync

    Rebuild Status : 17% complete

              Name : localhost.localdomain:1  (local to host localhost.localdomain)
              UUID : 58c81516:761836f4:4c1d6f59:602b393c
            Events : 21

    Number   Major   Minor   RaidDevice State
       0       8       48        0      active sync   /dev/sdd
       2       8       80        1      spare rebuilding   /dev/sdf

       1       8       64        -      faulty   /dev/sde
       
"""初始化完成后显示如下
    Number   Major   Minor   RaidDevice State
       0       8       48        0      active sync   /dev/sdd
       2       8       80        1      active sync   /dev/sdf

       1       8       64        -      faulty   /dev/sde
"""

[root@localhost ~]# ls /raid1
1.txt     # 数据未丢失
```
2. 移除故障硬盘
```
[root@localhost ~]# mdadm -r /dev/md1 /dev/sde
mdadm: hot removed /dev/sde from /dev/md1
[root@localhost ~]# mdadm -D /dev/md1
...
    Number   Major   Minor   RaidDevice State
       0       8       48        0      active sync   /dev/sdd
       2       8       80        1      active sync   /dev/sdf

```
3. 添加新硬盘至raid1
```
[root@localhost ~]# mdadm -a /dev/md1 /dev/sde
mdadm: added /dev/sde
[root@localhost ~]# mdadm -D /dev/md1
...
    Number   Major   Minor   RaidDevice State
       0       8       48        0      active sync   /dev/sdd
       2       8       80        1      active sync   /dev/sdf

       3       8       64        -      spare   /dev/sde
```

重要数据如：数据库，系统盘（把系统安装到raid1的md1设备上，可以对md1做分区）存储在RAID1

## 2.4 RAID5

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
