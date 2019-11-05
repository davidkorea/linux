# linux - centos7


- tty[1-6]就是你用ctr+alt+f[1-6]所看到的那个终端; 即虚拟控制台。其他的是外部终端和网络终端。
  - 有一个tty7是表示图形界面，我当前登录的是GNOME，当然就是图形界面了。还有tty1-tty6表示文字界面，可以用Ctrl+Alt+F1-F6切换，+F7就是切换回图形界面。

- pts/*为伪(虚拟)终端, 其中pts/0,1,2在桌面Linux中是标准输入，标准输出，标准出错
  - 当前打开了两个终端窗口，所以就有pts/0和pts/





3.4.4 终端设备/dev/tty/*、/dev/pts/*和/dev/tty
终端设备负责在用户进程和输入输出设备之间传送字符，通常是在终端显示屏上显示文字。终端设备接口由来已久，一直可以追溯到手动打字机时代。
伪终端设备模拟终端设备的功能，由内核为程序提供I/O接口，而不是真实的I/O设备，shell窗口就是伪终端。
常见的两个终端设备是/dev/tty1（第一虚拟控制台）和/dev/pts/0（第一虚拟终端），/dev/pts目录中有一个专门的文件系统。
/dev/tty代表当前进程正在使用的终端设备，虽然不是每个进程都连接到一个终端设备。
显示模式和虚拟控制台
Linux系统有两种显示模式：文本模式和X Windows系统服务器（即图形模式，通常是通过显示管理器）。通常系统是在文本模式下启动，但是很多Linux发行版通过内核参数和内置图形显示机制（如plymouth）将文本模式完全屏蔽起来，这样系统从始至终是在图形模式下启动。
Linux系统支持虚拟控制台来实现多个终端的显示，虚拟控制台可以在文本模式和图形模式下运行。在文本模式下，你可以使用ALT-Function在控制台之间进行切换，例如ALT-F1切换到/dev/tty1，ALT-F2切换到/dev/tty2等等。这些控制台通常会被getty进程占用以显示登录提示符，详见7.4节。
X server在图形模式下使用的虚拟控制台稍微有些不同，它不是从init配置中获得虚拟控制台，而是由X server来控制一个空闲的虚拟控制台，除非另外指定。例如，如果tty1和tty2上运行着getty进程，X server就会使用tty3。此外，X server将虚拟控制台设置为图形模式后，通常需要按CTRL-ALT-Function而不是ALT-Function来切换到其他虚拟控制台。
如果你想在系统启动后使用文本模式，可以按CTRL-ALT-F1。按ALT-F2、ALT-F3等返回X11会话。
如果在切换控制台的时候遇到问题，你可以尝试chvt命令强制系统切换工作台。例如：使用root运行以下命令切换到tty1：
# chvt 1



-----

由于笔记本电脑加入了非常多的省电机制或者是其他硬件的管理机制，包括显示适配器常常是整合型
的， 因此在笔记本电脑上面的硬件常常与一般桌面计算机不怎么相同。

使用 DVD 开机时，选择『』然后按下 [tab] 按键后，加入底下这些选项：
`nofb apm=off acpi=off pci=noacpi`

- apm(Advanced Power Management)是早期的电源管理模块
- acpi(Advanced Configuration and Power Interface)则是近期的电源管理模块。
这两者都是硬件本身就有支持的，但是笔记本电脑可能不是使用这些机制， 因此，当安装时启动这些机制将会造成一些错误，导致无法顺利安装。
- nofb 则是取消显示适配器上面的缓冲存储器侦测。因为笔记本电脑的显示适配器常常是整合型的，Linux 安装程序本身可能就不是很能够侦测到该显示适配器模块。此时加入 nofb 将可能使得你的安装过程顺利一些


















-----

# 1. linux basics

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

7. 
