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
3. 系统光盘开机自动挂载
```
[root@localhost ~]# echo "/dev/sr0 /mnt iso9660 defaults 0 0" >> /etc/fstab
[root@localhost ~]# mount -a
mount: /dev/sr0 写保护，将以只读方式挂载
[root@localhost ~]# ls /mnt/   #可以查看到此目录下有内容，说明挂载成功
CentOS_BuildTag  GPL       LiveOS    RPM-GPG-KEY-CentOS-7
```
4. 本地yum源
```
[root@localhost yum.repos.d]#rm -rf  /etc/yum.repos.d/*  # 删除原有的文件
[root@localhost yum.repos.d]# vim  CentOS7.repo  #写入以下红色内容
  [CentOS7]   
  name=CentOS-server     
  baseurl=file:///mnt  
  enabled=1  
  gpgcheck=0
```


5. 配置网络yum源
  - 阿里云镜像源站点（http://mirrors.aliyun.com/）。
  - centos镜像参考：http://mirrors.aliyun.com/help/centos
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup   # 1、备份
wget -O /etc/yum.repos.d/CentOS-Base.repohttp://mirrors.aliyun.com/repo/Centos-7.repo   #下载新的CentOS-Base.repo 到/etc/yum.repos.d/
```
6. 之后运行```yum makecache```生成缓存

7. 安装epel源 ```yum install epel-release –y```


