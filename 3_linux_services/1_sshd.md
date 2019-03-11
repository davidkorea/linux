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

8. 克隆后发现克隆的机器网卡无法启动，需要如下操作：
  - 删除克隆机器的网卡MAC地址
  - 删除网卡信息文件```rm -rf /etc/udev/rules.d/70-persistent-net.rules```
  - 重启：reboot

# 2. sshd服务安装-ssh命令使用方法
1. SSHD服务
  - 介绍：SSH 协议：安全外壳协议。为 Secure Shell 的缩写。SSH 为建立在应用层和传输层基础上的安全协议。
  - 作用：sshd服务使用SSH协议可以用来进行远程控制， 或在计算机之间传送文件。相比较之前用telnet方式来传输文件要安全很多，因为telnet使用明文传输，是加密传输。
2. 服务安装：需要安装OpenSSH 四个安装包, OpenSSH软件包，提供了服务端后台程序和客户端工具，用来加密远程控件和文件传输过程中的数据，并由此来代替原来的类似服务。
  - 安装包：OpenSSH服务需要4 个软件包
    - openssh-5.3p1-114.el6_7.x86_64：包含OpenSSH服务器及客户端需要的核心文件
    - openssh-clients-5.3p1-114.el6_7.x86_64：OpenSSH客户端软件包
    - openssh-server-5.3p1-114.el6_7.x86_64：OpenSSH服务器软件包
    - openssh-askpass-5.3p1-114.el6_7.x86_64：支持对话框窗口的显示，是一个基于X 系统的密码诊断工具
3. 配置yum源，通过```yum install openssh openssh-clients openssh-server -y```来安装。前提：系统以及配置好yum源，（本地源or网络源） 推荐用yum来安装。 奇怪， 安装完成后并没有 openssh-askpass-5.3p1-114.el6_7.x86_64， 只有下面三个
    ```shell
    [root@localhost ~]# rpm -qa | grep openssh
    openssh-7.4p1-16.el7.x86_64
    openssh-server-7.4p1-16.el7.x86_64
    openssh-clients-7.4p1-16.el7.x86_64
    ```
# 3. sshd服务配置和管理    

介绍下配置文件/etc/ssh/sshd_config，以及需要安全调优的地方。注：参数前面有#，且#后没有空格，表示是默认缺省值。如要变更，需去除前面#，再该更保存，才能生效。

### port22

设置sshd监听端口号
- SSH 预设使用 22 这个port，也可以使用多个port，即重复使用 port 这个设定项目！
- 例如想要开放sshd端口为 22和222，则多加一行内容为： Port 222 即可
- 然后重新启动sshd这样就好了。 建议大家修改 port number 为其它端口。防止别人暴力破解。




```shell
[root@localhost ~]# vim /etc/ssh/sshd_config 

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/bin:/usr/bin

  9 # OpenSSH is to specify options with their default value where
 10 # possible, but leave them commented.  Uncommented options over    ride the
 11 # default value.
 12 
 13 # If you want to change the port on a SELinux system, you have     to tell
 14 # SELinux about this change.
 15 # semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
 16 #
 17 Port 222
```































      
