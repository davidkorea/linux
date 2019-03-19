# 搭建无人执守安装服务器 - 批量安装
1. 无人执守安装服务器常见概念
2. 搭建无人执守安装服务器及相关配置文件
3. 实战：为公司内网搭建一个搭建无人执守安装服务器

# 1. 无人执守安装服务器常见概念
- 需要使用到的服务：PXE + DHCP + TFTP + Kickstart+ FTP/HTTP

- 安装三四十台机器，一般就需要无人值守安装了
- 新购买服务器 - 上架 - RAID - 重启，快捷键，从网卡启动 - DHCP server, 三个服务安装在一台服务器上
<p align="center">
  <img src="https://i.loli.net/2019/03/18/5c8f1368ec875.png" width=300, height=300>
</p>

- 安装的时候，最开始，通过快捷键进入网络引导模式。不建议更改第一启动位置。因为安装好系统，重启还会再次通过网络引导安装系统

#### 1. PXE原理和概念：  
- 严格来说，PXE 并不是一种安装方式，而是一种引导的方式。
- 进行 PXE 安装的必要条件是要安装的计算机中包含一个 PXE 支持的网卡（NIC），即网卡中必须要有 PXE Client。PXE （Pre-boot Execution Environment）协议使计算机可以通过网络启动。协议分为 client 和 server 端，PXE client 在网卡的 ROM 中，当计算机引导时，BIOS 把 PXE client 调入内存执行，由 PXE client 将放置在远端的文件通过网络下载到本地运行。现在网卡都支持。
- 运行 PXE 协议需要设置 DHCP 服务器 和 TFTP 服务器。DHCP 服务器用来给 PXE client（将要安装系统的主机）分配一个 IP 地址，由于是给 PXE client 分配 IP 地址，所以在配置 DHCP 服务器时需要增加相应的 PXE 设置```filename "pxelinux.0"```。此外，在 PXE client 的 ROM 中，已经存在了 TFTP Client。PXE Client 通过 TFTP 协议到 TFTP Server 上下载所需的文件。

#### 2. 什么是KickStart 
- KickStart是一种无人职守安装方式。KickStart的工作原理是通过记录典型的安装过程中所需人工干预填写的各种参数，并生成一个名为 ks.cfg的文件
- 在其后的安装过程中（不只局限于生成KickStart安装文件的机器）当出现要求填写参数的情况时，安装程序会首先去查找 KickStart生成的文件，当找到合适的参数时，就采用找到的参数，当没有找到合适的参数时，才需要安装者手工干预。
- 如果KickStart文件涵盖了安装过程中出现的所有需要填写的参数时，安装者完全可以只告诉安装程序从何处取ks.cfg文件，然后去忙自己的事情。等安装完毕，安装程序会根据ks.cfg中设置的重启选项来重启系统，并结束安装。


#### 3. 执行 PXE + KickStart安装需要准备内容：
- DHCP 服务器用来给客户机分配IP
- TFTP 服务器用来存放PXE的相关文件，比如：系统引导文件
- FTP 服务器用来存放系统安装文件，需要挂载光盘镜像
- KickStart所生成的ks.cfg配置文件；
- 带有一个 PXE 支持网卡的将安装的主机

# 2. 搭建无人执守安装服务器及相关配置文件

## 2.1 搭建yum ftp tftp dhcp服务
#### 1. 配置yum基本环境
```
[root@server162~]# mount  /dev/cdrom  /mnt
[root@server162~]# vi /etc/yum.repos.d/serverl.repo #在/etc/yum.repos.d目录下创建以.repo结尾的文件
```
#### 2. 安装ftp服务以及开启服务，设置为开机自动启动
- 纯净系统
```
[root@server162 ~]# yum install vsftpd -y
[root@server162 ~]# systemctl start vsftpd
[root@server162 ~]# systemctl enable vsftpd
```
- 之前配置国ftp的清空，更改vsftp设置
  - 实现匿名可以直接访问pub
  - team1 11111 依然可以向以前一样通过filezilla进行有账户访问/var/www/html
```
[root@server162 ~]# vim /etc/vsftpd/vsftpd.conf 

 12 anonymous_enable=YES
 29 anon_upload_enable=YES
 33 anon_mkdir_write_enable=YES
""" 并没有改动
103 local_root=/var/www/html
104 chroot_list_enable=YES
105 # (default follows)
106 chroot_list_file=/etc/vsftpd/chroot_list
107 allow_writeable_chroot=YES
"""
"""
120 ssl_enable=NO         # 只是禁用来这一行
121 #ssl_enable=YES
122 allow_anon_ssl=NO
123 force_local_data_ssl=YES
124 force_local_logins_ssl=YES
125 force_anon_logins_ssl=YES
126 force_anon_data_ssl=YES
127 ssl_tlsv1=YES
128 ssl_sslv2=YES
129 ssl_sslv3=YES
130 require_ssl_reuse=NO
131 ssl_ciphers=HIGH
132 rsa_cert_file=/etc/vsftpd/.sslkey/vsftpd.pem
133 rsa_private_key_file=/etc/vsftpd/.sslkey/vsftpd.pem
"""
```
#### 3. 安装TFTP,修改tftp配置文件及开启服务
在 PXE client 的ROM中，已经存在了TFTP Client。 PXE Client 通过TFTP协议到TFTP Server下载所需的文件

```
[root@server162 ~]# yum install tftp tftp-server xinetd -y
```
  修改第13，14行
```
[root@server162 ~]# vim /etc/xinetd.d/tftp 

  1 # default: off
  2 # description: The tftp server serves files using the trivial file transfer \
  3 #       protocol.  The tftp protocol is often used to boot diskless \
  4 #       workstations, download configuration files to network-aware printers, \
  5 #       and to start the installation process for some operating systems.
  6 service tftp
  7 {
  8         socket_type             = dgram
  9         protocol                = udp
 10         wait                    = yes
 11         user                    = root
 12         server                  = /usr/sbin/in.tftpd
 13         server_args             = -s /tftpboot        # 并不存在，需要自行创建
 14         disable                 = no
 15         per_source              = 11
 16         cps                     = 100 2
 17         flags                   = IPv4
 
[root@server162 ~]# systemctl start xinetd          # 启动服务
[root@server162 ~]# lsof -i:69
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
xinetd  9102 root    5u  IPv4  51445      0t0  UDP *:tftp 
```

- ```server_args = -s /tftpboot```: 是tftp服务器运行时的参数。-s /tftpboot表示服务器默认的目录是 /tftpboot,当执行put a.txt时，文件会被放到服务器的/tftpboot/a.txt，省去你敲put a /tftpboot/的麻烦。你也可以加其它服务器运行参数到这，具体可以执行man tftpd命令查阅。

- 参数-c: 上传文件时，服务器上没有。就自动创建这个文件。默认tftp客户端，只能上传tftp服务器已经有的文件。也就是只能传上去并覆盖服务器上的原文件。如果想上传原来目录中没有的文件，需要修改tftp服务器的配置文件并重起服务。需要修改如下：```server_args = -s /tftpboot -c```

TFTP (Trivial File Transfer Protocol)，中译简单文件传输协议或小型文件传输协议. 大家一定记得在2003年8月12日全球爆发冲击波（Worm.Blaster）病毒，这种病毒会监听端口69,模拟出一个TFTP服务器，并启动一个攻 击传播线程,不断地随机生成攻击地址，进行入侵。另外tftp被认为是一种不安全的协议而将其关闭，同时也是防火墙打击的对象，这也是有道理的。tftp 在嵌入式linux还是有用武之地的。需要打开防火墙，允许tftp访问网络。


#### 4. 安装DHCP，修改配置文件及开启服务

参考： [搭建DHCP服务器](https://github.com/davidkorea/linux_study/blob/master/3_linux_services/3_DHCP.md#21-%E9%85%8D%E7%BD%AEdhcp-server)
1. 安装服务
```
[root@server162~]# yum install dhcp -y
```
2. 添加一块信网卡vmnet4，并设置固定IP 192.168.1.10（写入ifcfg-ens39配置文件），不要仅仅临时``` ifconfig ens39 192.168.1.10/24```
3. 配置DHCP
```shell
[root@server162~]# cp /usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample /etc/dhcp/dhcpd.conf   # 生成配置文件
cp: overwrite `/etc/dhcp/dhcpd.conf'? y

subnet 192.168.1.0 netmask 255.255.255.0 {
        range 192.168.1.100 192.168.1.200;
        option domain-name-servers 192.168.1.1;
        option domain-name "xerox.kr";
        option routers 192.168.1.1;
        option broadcast-address 192.168.1.255;
        default-lease-time 600;
        max-lease-time 7200;
        next-server 192.168.1.10;       # 类似下一跳路由的意思
        filename "pxelinux.0";          # 读取pxelinux.0文件，PXE引导启动
}

##### 设置完成后，先不启动dhcpd服务，最后全部设置完再次启动 #####
```
- ```filename "pxelinux.0"```， 运行 PXE 协议需要设置 DHCP 服务器 和 TFTP 服务器。DHCP 服务器用来给 PXE client（将要安装系统的主机）分配一个 IP 地址，由于是给 PXE client 分配 IP 地址，所以在配置 DHCP 服务器时需要增加相应的 PXE 设置

## 2.2 配置PXE启动所需的相关文件
#### 1. 安装服务
```
[root@server162~]# yum -y install system-config-kickstart && syslinux
##### 如果syslinux安装不成功，需要单独在yum install -y syslinux安装一下
```
#### 2. 将tftp需要共享出去的文件，存放点tftp根目录/tftpboot
```
[root@server162~]# mkdir /tftpboot
[root@server162~]# mkdir /tftpboot/pxelinux.cfg
[root@server162~]# cp /usr/share/syslinux/pxelinux.0 /tftpboot/    # 只有安装了syslinux软件包，才会有/usr/share/syslinux/目录及
[root@server162~]# cp /mnt/images/pxeboot/initrd.img  /tftpboot/
[root@server162~]# cp /mnt/images/pxeboot/vmlinuz  /tftpboot/
[root@server162~]# cp /mnt/isolinux/isolinux.cfg  /tftpboot/pxelinux.cfg/default
[root@server162~]# chmod 644  /tftpboot/pxelinux.cfg/default
```
```
[root@server162 ~]# tree /tftpboot/
/tftpboot/
|-- initrd.img
|-- pxelinux.0    
|-- vmlinuz
|-- pxelinux.cfg
    `-- default
```

#### 3. 修改安装选项default文件
default配置文件的修改就是通过ftp服务器方式来访问kickstart文件
```shell
[root@server162 ~]# vim /tftpboot/pxelinux.cfg/default 

  1 default linux       # 此处的linux就是下面61行的label linux入口

 61 label linux
 62   menu label ^Install CentOS 7
 63   kernel vmlinuz
 64   append initrd=initrd.img inst.repo=ftp://192.168.1.10/pub inst.ks=ftp://192.168.1.10/ks.cfg
                            # 添加inst.rep 和 inst.ks ks文件路径，使安装程序通过FTP服务器访问kickstart文件
```

有多种方法可访问kickstart文件
- 其中最常用的一种方法是通过网络服务器进行，例如：ftp服务器、http WEB服务器或NFS服务器，这种方法非常易于部署，并且也使管理更改变得十分简单。
- 也可以通过USB磁盘、CD－ROM或本地硬盘。如果USB或CD－ROM中的kickstart文件非常便于访问，只需将kickstart文件放置在用来开始安装的引导介质中。而使用DHCP服务器和TFTP及PXE配置起来更为复杂。
- 使安装程序指向kickstart文件的书写格式如下：
  ```  
  ks=ftp://server/dir/file 如:ks=ftp://ftp服务器IP/ks.cfg
  ks=http://server/dir/file 如:ks=http://http服务器IP/ks.cfg
  ks=nfs:server:/dir/file 如:ks=nfs:nfs服务器IP:/var/ftp/pub/ks.cfg
  ks=hd:device:/dir/file 如:ks=hd:sdb1:/kickstar-files/ks.cfg
  ks=cdrom:/dir/file 如:ks=cdrom:/kickstart-files/ks.cfg
  ```
#### 4. 制作kickstart的无人值守安装文件
1. 配置ftp形式的yum源，将光盘挂载到ftp pub目录下

```
[root@server162 ~]# cd /etc/yum.repos.d/
[root@server162 yum.repos.d]# ls
CentOS-7-aliyun.repo  CentOS-CR.repo         CentOS-Media.repo    CentOS-Vault.repo
CentOS-Base.repo      CentOS-Debuginfo.repo  CentOS-Sources.repo  CentOS-fasttrack.repo

[root@server162 yum.repos.d]# mkdir bak
[root@server162 yum.repos.d]# mv *.repo  bak/         # 这样做是避免其他yum文件的影响
[root@server162 yum.repos.d]# ls
bak  
[root@server162 yum.repos.d]# vim my.repo
  [development]        
  name=my-centos7-dvd
  baseurl=file:///var/ftp/pub                           # 需要将光盘挂载到此路径下
  enabled=1
  gpgcheck=0

[root@server162 ~]# mount /dev/cdrom /var/ftp/pub     # 缺少这一步，下面kickstart中无法选择安装包
                                                      # 并且 makecache 也会报错

[root@server162 ~]# cd /var/ftp/pub/
[root@server162 pub]# ls
CentOS_BuildTag  EULA  LiveOS    RPM-GPG-KEY-CentOS-7          TRANS.TBL  isolinux
EFI              GPL   Packages  RPM-GPG-KEY-CentOS-Testing-7  images     repodata

[root@server162 yum.repos.d]# yum makecache
```

#### 5. 通过system-config-kickstart制作ks.cfg文件
```
[root@server162 ~]# system-config-kickstart 

(process:27304): Gtk-WARNING **: 10:41:31.785: Locale not supported by C library.
	Using the fallback 'C' locale.
```
> 用xshell远程连接，执行上面的命令可能无法弹出选择框
> - 需要安装gdm ```yum install -y gdm```
> - 在Xstart 里执行
>     ![](https://i.loli.net/2019/03/19/5c9057ead644b.png)
    
![1.png](https://i.loli.net/2019/03/19/5c905d6f47b7a.png)
![2.png](https://i.loli.net/2019/03/19/5c905d7975ade.png)
![3.png](https://i.loli.net/2019/03/19/5c905d88be508.png)
![4.png](https://i.loli.net/2019/03/19/5c905d90cab4a.png)
![5.png](https://i.loli.net/2019/03/19/5c905d987975c.png)
网络和认证保持默认
![6.png](https://i.loli.net/2019/03/19/5c905d9ddf576.png)
![7.png](https://i.loli.net/2019/03/19/5c905da457486.png)
![8.png](https://i.loli.net/2019/03/19/5c905db8c3d77.png)    
![9.png](https://i.loli.net/2019/03/19/5c905da9889d5.png)
![10.png](https://i.loli.net/2019/03/19/5c905db477989.png)
    

```
[root@server162 ~]#cp ks.cfg /var/ftp
```
