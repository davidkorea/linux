# linux - centos7

# 1. linux baiscs

1. ```systemctl status NetworkManager```, #查看networkmanager服务是是否启动

2. /etc/sysconfig/network-scripts

```ls /etc/sysconfig/network-scripts/ifcfg-ens33```, #IP地址，子网掩码等配置文件

```ls /etc/sysconfig/network-scripts/ifcfg-lo```, #网卡回环地址

```cat /etc/resolv.conf```, #DNS配置文件

```cat /etc/hosts```, #设置主机和IP绑定信息

```cat /etc/hostname```, #设置主机名
