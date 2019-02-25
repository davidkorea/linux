# linux - centos7 baiscs

15. xfs文件系统的备份和恢复，xfsdump xfsrestore

XFS提供了 xfsdump 和 xfsrestore 工具协助备份XFS文件系统中的数据。xfsdump 按inode顺序备份一个XFS文件系统。centos7选择xfs格式作为默认文件系统，而且不再使用以前的ext，仍然支持ext4，xfs专为大数据产生，每个单个文件系统最大可以支持8eb，单个文件可以支持16tb，不仅数据量大，而且扩展性高。还可以通过xfsdump，xfsrestore来备份和恢复。

与传统的UNIX文件系统不同，XFS不需要在备份前被卸载；对使用中的XFS文件系统做备份就可以保证镜像的一致性。XFS的备份和恢复的过程是可以被中断然后继续，无须冻结文件系统。xfsdump 甚至提供了高性能的多线程备份操作——它把一次dump拆分成多个数据流，每个数据流可以被发往不同的目的地。

xfsdump的备份级别有以下两种，默认为0（即完全备份）

|级别|说明|
|-|-|
|0 级别| 完全备份，每次都把指定的备份目录完整的复制一遍，不管目录下的文件有没有变化|
|1 到9级别|增量备份，每次将之前（第一次、第二次、直到前一次）做过备份之后有变化的文件进行备份|

fdisk挂载新硬盘，vm中创建新硬盘。安装系统时已默认挂载sda，新硬盘识别为sdb

- 创建新分区partition
  - ```fdisk sdb```
  - ```n```, enter, enter
  - ```+1G```
  - ```w```,write for create a new partition. 
  - ```fdisk /dev/sdb``` -> ```p```, 查看此硬盘下已有分区partition
    ```
    [root@localhost b]# fdisk /dev/sdb
    Welcome to fdisk (util-linux 2.23.2).

    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.


    Command (m for help): n
    Partition type:
       p   primary (2 primary, 0 extended, 2 free)
       e   extended
    Select (default p): 
    Using default response p
    Partition number (3,4, default 3): 
    First sector (4196352-41943039, default 4196352): 
    Using default value 4196352
    Last sector, +sectors or +size{K,M,G} (4196352-41943039, default 41943039): +1G
    Partition 3 of type Linux and of size 1 GiB is set

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.

    ```
- 使用新分区前，先进行格式化
  - ```mkfs.xfs /dev/sdb3```
    ```
    [root@localhost ~]# mkfs.xfs /dev/sdb3
    meta-data=/dev/sdb3              isize=512    agcount=4, agsize=65536 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=1        finobt=0, sparse=0
    data     =                       bsize=4096   blocks=262144, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
    log      =internal log           bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0
    ```
- 挂载新分区至制定目录下，新分区可以正常使用
  - ```mkdir ./sdb3```
  - ```mount /dev/sdb3 /sdb3```

再新分区sdb2中创建一些文件，下面操作将sdb2进行备份。

- xfsdump -f 备份存放位置 要备份路径或设备文件  
  - ```xfsdump -f ```
    ```
    [root@localhost ~]# xfsdump -f /opt/sdb3_dump /sdb3
    xfsdump: using file dump (drive_simple) strategy
    xfsdump: version 3.1.7 (dump format 3.0) - type ^C for status and control

     ============================= dump label dialog ==============================

    please enter label for this dump session (timeout in 300 sec)
     -> dump_sdb3
    session label entered: "dump_sdb3"

     --------------------------------- end dialog ---------------------------------

    xfsdump: level 0 dump of localhost.localdomain:/sdb3
    xfsdump: dump date: Mon Feb 25 11:23:45 2019
    xfsdump: session id: 05b2aeb0-8426-4850-8078-e09cdbf50212
    xfsdump: session label: "dump_sdb3"
    xfsdump: ino map phase 1: constructing initial dump list
    xfsdump: ino map phase 2: skipping (no pruning necessary)
    xfsdump: ino map phase 3: skipping (only one dump stream)
    xfsdump: ino map construction complete
    xfsdump: estimated dump size: 20800 bytes
    xfsdump: /var/lib/xfsdump/inventory created

     ============================= media label dialog =============================

    please enter label for media in drive 0 (timeout in 300 sec)
     -> media
    media label entered: "media"

     --------------------------------- end dialog ---------------------------------

    xfsdump: creating dump session media file 0 (media 0, file 0)
    xfsdump: dumping ino map
    xfsdump: dumping directories
    xfsdump: dumping non-directory files
    xfsdump: ending media file
    xfsdump: media file size 21016 bytes
    xfsdump: dump size (non-dir files) : 0 bytes
    xfsdump: dump complete: 13 seconds elapsed
    xfsdump: Dump Summary:
    xfsdump:   stream 0 /opt/sdb3_dump OK (success)
    xfsdump: Dump Status: SUCCESS
    ```
  - ```[root@xuegod63 sdb1]# xfsdump -f /opt/dump_passwd /sdb1 -L dump_passwd -M media1```,指定备份时免交互操作，方便后期做定时备份, -L  ：xfsdump  纪录每次备份的 session 标头，这里可以填写针对此文件系统的简易说明。 -M  ：xfsdump 可以纪录储存媒体的标头，这里可以填写此媒体的简易说明

- 查看备份信息与内容
  - ```xfsdump -I```，大写字母i
    ```
    [root@localhost ~]# xfsdump -I
    file system 0:
      fs id:		f0071f64-9967-4848-a3af-08232e5ee4aa
      session 0:
        mount point:	localhost.localdomain:/sdb3
        device:		localhost.localdomain:/dev/sdb3
        time:		Mon Feb 25 11:23:45 2019
        session label:	"dump_sdb3"
        session id:	05b2aeb0-8426-4850-8078-e09cdbf50212
        level:		0
        resumed:	NO
        subtree:	NO
        streams:	1
        stream 0:
          pathname:	/opt/sdb3_dump
          start:		ino 0 offset 0
          end:		ino 1 offset 0
          interrupted:	NO
          media files:	1
          media file 0:
            mfile index:	0
            mfile type:	data
            mfile size:	21016
            mfile start:	ino 0 offset 0
            mfile end:	ino 1 offset 0
            media label:	"media"
            media id:	8297b6b5-d9f8-4c54-bc33-a1c97eb7c92d
    xfsdump: Dump Status: SUCCESS    
    ```
  - ```ls /var/lib/xfsdump/inventory/```，可以查到上述fs id f0071f64-9967-4848-a3af-08232e5ee4aa.InvIndex

- 恢复备份
  - 先删除原有文件
    - ```cd sdb3```
    - ```rm -rf ./*```
  - xfsrestore -f 指定备份文件的存放位置 指定存放恢复后的文件的路径
    - ```xfsrestore -f /opt/sdb3_dump /sdb3```
      ```
      [root@localhost ~]# xfsrestore -f /opt/sdb3_dump /sdb3
      xfsrestore: using file dump (drive_simple) strategy
      xfsrestore: version 3.1.7 (dump format 3.0) - type ^C for status and control
      xfsrestore: searching media for dump
      xfsrestore: examining media file 0
      xfsrestore: dump description: 
      xfsrestore: hostname: localhost.localdomain
      xfsrestore: mount point: /sdb3
      xfsrestore: volume: /dev/sdb3
      xfsrestore: session time: Mon Feb 25 11:23:45 2019
      xfsrestore: level: 0
      xfsrestore: session label: "dump_sdb3"
      xfsrestore: media label: "media"
      xfsrestore: file system id: f0071f64-9967-4848-a3af-08232e5ee4aa
      xfsrestore: session id: 05b2aeb0-8426-4850-8078-e09cdbf50212
      xfsrestore: media id: 8297b6b5-d9f8-4c54-bc33-a1c97eb7c92d
      xfsrestore: using online session inventory
      xfsrestore: searching media for directory dump
      xfsrestore: reading directories
      xfsrestore: 1 directories and 0 entries processed
      xfsrestore: directory post-processing
      xfsrestore: restore complete: 0 seconds elapsed
      xfsrestore: Restore Summary:
      xfsrestore:   stream 0 /opt/sdb3_dump OK (success)
      xfsrestore: Restore Status: SUCCESS

      
      ```

14. 查看文件
  - ```cat /etc/passwd```
  - ```more /etc/passwd```, 按下回车刷新一行，按下空格刷新一屏，输入q键退出。不支持后退
  - ```less /etc/passwd```, 支持前后翻滚，既可以向上翻页（pageup按键），也可以向下翻页（pagedown按键）
  - ```head -n 3 /etc/passwd```, 只看前三行 默认不加-n时显示前十行
  - ```tail -n 3 /var/log/secure```, 查看最后3行记录
  - ```tail -f /var/log/secure```, 动态查看文件内容, 不关闭
  
13. ```mv a.txt dir1/abc.txt```, 在移动文件的时候支持改名操作,a->abc

12. cp 源文件/目录  目录文件/目录
- ```cp -r /boot/grup /opt```, -R/r：递归处理，将指定目录下的所有文件与子目录一并处理

11. ```rm -rf```, -f 强制删除没有提示, -r 删除目录


10. mkdir
- ```mkdir -p /tmp/a/b/c```,在创建一个目录的时候，如果这个目录的上一级不存在的话，要加参数-p = parent父目录

9. touch
- ```touch file{1..8}.txt```, file1, file2, ...file8


8. 查看文件状态
- ```ll /etc/passwd```, -rw-r--r--. 1 root root 2307 Feb 22 13:21 /etc/passwd
- ```stat /etc/passwd```

  ```
      File: '/etc/passwd'
    Size: 2307      	Blocks: 8          IO Block: 4096   regular file
  Device: 802h/2050d	Inode: 8443595     Links: 1
  Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
  Context: system_u:object_r:passwd_file_t:s0
  Access: 2019-02-24 13:30:01.853163925 +0800 # 访问时间：atime  查看内容  cat  a.txt
  Modify: 2019-02-22 13:21:28.562334692 +0800 # 修改时间：mtime  修改内容  vim a.txt
  Change: 2019-02-22 13:21:28.563334692 +0800 # 改变时间：ctime  文件属性，比如权限change time, chmod +x a.sh
   Birth: -
  ```







1. ```systemctl status NetworkManager```, #查看networkmanager服务是是否启动

2. RHEL/CENTOS Linux网络相关的配置文件/etc/sysconfig/network-scripts

  - ```ls /etc/sysconfig/network-scripts/ifcfg-ens33```, #IP地址，子网掩码等配置文件

  - ```ls /etc/sysconfig/network-scripts/ifcfg-lo```, #网卡回环地址

  - ```cat /etc/resolv.conf```, #DNS配置文件

  - ```cat /etc/hosts```, #设置主机和IP绑定信息

  - ```cat /etc/hostname```, #设置主机名

3. 修改ip地址
  - 方法1: ```nmtui```, 使用nmtui文本框方式修改IP，重启网卡服务生效：```systemctl restart network``` ---重启服务
  - 方法2: ```vim /etc/sysconfig/network-scripts/ifcfg-ens33```

4. 关闭防火墙并设置开机开不启动
  - ```systemctl status firewalld.service```    #查看firewalld状态
  - ```systemctl stop firewalld```       #关闭
  - ```systemctl start firewalld```       #开启
  - ```systemctl disable firewalld```     #开机自动关闭   //RHLE7
  - ```chkconfig --list|grep network```    #查看开机是否启动   //RHLE6
  - ```systemctl enable firewalld```     #开机自动启动

5. 设置系统光盘开机自动挂载
  - ```vim  /etc/fstab```  #在文档最后，添加以以下内容：
/dev/cdrom 			      /mnt			  iso9660 defaults        0 0
  - ```mount -a```
mount: /dev/sr0 写保护，将以只读方式挂载
  - ```ls /mnt/```   #可以查看到此目录下有内容，说明挂载成功
CentOS_BuildTag  GPL       LiveOS    RPM-GPG-KEY-CentOS-7

6. 配置本地YUM源
yum的一切配置信息都储存在一个叫yum.repos.d的配置文件中，通常位于/etc/yum.repos.d目录下 
  - ```rm -rf  /etc/yum.repos.d/*```, 删除原有的文件, 也可以不删除 
  - ```vim  CentOS7.repo```  #创建一个新的yum源配置文件，yum源配置文件的结尾必须是.repo, 写入以下内容
        
        [CentOS7]        # --->yum的ID，必须唯一 
        name=CentOS-server    #  ----->描述信息
        baseurl=file:///mnt    # -------> /mnt表示的是光盘的挂载点  . file:后面有3个///
        enabled=1   # ------>启用
        gpgcheck=0   # ---->取消验证

  - ```yum clean all```			#清空并生成缓存列表,清空yum缓存
  - ```yum list```						#生成缓存列表
  - ```yum -y install httpd``` # 验证一下

## 2019-02-25

7. install tree package
  - ```mount /dev/sr0 /media/```, # 挂在磁盘与media目录，mount: /dev/sr0 写保护，将以只读方式挂载， 也可以挂在于/mnt，此目录默认为空
  - ``` rpm -ivh /media/Packages/tree-1.6.0-10.el7.x86_64.rpm```
  
8. centos 各目录介绍
   
|目录|说明|
|-|-|
|/|root，处于linux系统树形结构的最顶端，它是linux文件系统的入口，所有的目录、文件、设备都在 / 之下|
|/bin|bin是Binary的缩写。常用的二进制命令目录。比如 ls、cp、mkdir、cut等；和/usr/bin类似，一些用户级gnu工具|
|/boot|存放的系统启动相关的文件，例如：kernel.grub(引导装载程序)|
|/dev | dev是Device的缩写。设备文件目录，比如声卡、磁盘...在Linux中一切都被看做文件。终端设备、磁盘等等都被看做文件。设备文件:/dev/sda,/dev/sda1,/dev/tty1,/dev/tty2,/dev/pts/1, /dev/zero, /dev/null, /dev/cdrom |
|/etc|常用系统及二进制安装包配置文件默认路径和服务器启动命令目录。passwd 用户信息文件，shadow  用户密码文件，group 存储用户组信息，fstab 系统开机启动自动挂载分区列表，hosts 设定用户自己的IP与主机名对应的信息。|
|/home|普通用户的家目录默认存放目录 |
|/lib|库文件存放目录,函数库目录|
|/mnt, /media|一般用来临时挂载存储设备的挂载目录，比如有cdrom、U盘等目录,在CENTOS7中会挂载到/run下面,/run目录默认不为空|
|/opt|option表示的是可选择的意思，有些软件包也会被安装在这里 |
|/proc|操作系统运行时，进程（正在运行中的程序）信息及内核信息（比如cpu、硬盘分区、内存信息等）存放在这里。/proc目录是伪装的文件系统proc的挂载目录，proc并不是真正的文件系统。因此，这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。也就是说，这个目录的内容不在硬盘上而是在内存里.查看CPU信息 ```cat /proc/cpuinfo``` |
|/sys|系统目录，存放硬件信息的相关文件|
|/run|运行目录，存放的是系统运行时的数据，比如进程的PID文件|
|/srv|服务目录，存放的是我们本地服务的相关文件|
|/sbin|大多数涉及系统管理的命令都存放在该目录中，它是超级权限用户root的可执行命令存放地，普通用户无权限执行这个目录下的命令，凡是目录sbin中包含的命令都是root权限才能执行的  |
|/tmp|该目录用于存放临时文件，有时用户运行程序的时候，会产生一些临时文件。/tmp就是用来存放临时文件的。/var/tmp目录和该目录的作用是相似的。不能存放重要数据，它的权限比较特殊 ```ls –ld /tmp # -d只看目录，不看下面发文件``` -> ```drwxrwxrwt 10 root root 12288 Oct 3 20:45 /tmp/``` →粘滞位（sticky bit）目录的sticky位表示这个目录里的文件只能被owner和root删除|
|/var|系统运行和软件运行时产生的日志信息，该目录的内容是经常变动的，存放的是一些变化的文件。比如/var下有/var/log目录用来存放系统日志的目录，还有mail、/var/spool/cron   |
|/usr|存放应用程序和文件，/usr/bin 普通用户使用的应用程序，/usr/sbin 管理员使用的应用程序，/usr/lib 库文件Glibc(32位)，/usr/lib64 库文件Glibc|
|/lib, /lib64 都在/usr目录下| 这个目录里存放着系统最基本的动态链接共享库，包含许多被/bin/和/sbin/中的程序使用的库文件，目录/usr/lib/中含有更多用于用户程序的库文件。作用类似于windows里的DLL文件，几乎所有的应用程序都需要用到这些共享库。注：lib***.a是静态库，lib***.so是动态库。静态库在编译时被加载到二进制文件中，动态库在运行时加载到进程的内存空间中，简单的说：这些库是为了让你的程序能够正常编译运行的，其实类似于WIN中.dll文件，几乎所有的应用程序都需要用到这些共享库|
|/lost+found 只在centos6中有|默认为空，被FSCK（file system check用来检查和维护不一致的文件系统。若系统掉电或磁盘发生问题，可利用fsck命令对文件系统进行检查）用来放置零散文件（没有名称的文件） 当系统非法关机后，这里就会存放一些文件。在centos6版本下，每个分区的挂载点下会有些目录|

