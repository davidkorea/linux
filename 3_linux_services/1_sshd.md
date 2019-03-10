# sshd服务搭建管理和防止暴力破解
1. Linux服务前期环境准备、搭建一个RHEL7环境
2. sshd服务安装-ssh命令使用方法
3. sshd服务配置和管理
4. 防止SSHD服务暴力破解的几种方式

# 1.  准备环境

1. 清空关闭防火墙
```
iptables -F
systemctl stop firewalld
systemctl disable firewalld
```
2. 永久关闭Selinux
```
# 临时关闭
[root@localhost ~]# getenforce 
Enforcing
[root@localhost ~]# setenforce 0
setenforce: SELinux is disabled
```
```
# 永久关闭
[root@localhost ~]# vim /etc/selinux/config  
改：7 SELINUX=enforcing     #前面的7，表示文档中第7行。方便你查找
为：7 SELINUX=disabled
[root@localhost ~]# getenforce 
Disabled
```
