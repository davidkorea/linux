# linux - centos7

# 1. linux baiscs

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
|/usr|
存放应用程序和文件，
/usr/bin 普通用户使用的应用程序
/usr/sbin 管理员使用的应用程序
/usr/lib 库文件Glibc(32位)
/usr/lib64 库文件Glibc|
|/lib, /lib64 都在/usr/目录下| 这个目录里存放着系统最基本的动态链接共享库，包含许多被/bin/和/sbin/中的程序使用的库文件，目录/usr/lib/中含有更多用于用户程序的库文件。作用类似于windows里的DLL文件，几乎所有的应用程序都需要用到这些共享库
注：lib***.a是静态库
     lib***.so是动态库
静态库在编译时被加载到二进制文件中
动态库在运行时加载到进程的内存空间中
简单的说：这些库是为了让你的程序能够正常编译运行的
其实类似于WIN中.dll文件，几乎所有的应用程序都需要用到这些共享库|


 
