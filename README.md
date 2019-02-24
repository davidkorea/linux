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
  - ```vim  /etc/fstab```  #在文档最后，添加以一下红色内容：
/dev/cdrom 			      /mnt			  iso9660 defaults        0 0
  - ```mount -a```
mount: /dev/sr0 写保护，将以只读方式挂载
  - ```ls /mnt/```   #可以查看到此目录下有内容，说明挂载成功
CentOS_BuildTag  GPL       LiveOS    RPM-GPG-KEY-CentOS-7
