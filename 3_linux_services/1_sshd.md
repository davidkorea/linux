# sshd服务搭建管理和防止暴力破解
1. Linux服务前期环境准备、搭建一个RHEL7环境
2. sshd服务安装-ssh命令使用方法
3. sshd服务配置和管理
4. 防止SSHD服务暴力破解的几种方式

# 1.  准备环境

1. 清空关闭防火墙
```
iptables -F     # 可以通过/sbin/iptables -F清除所有规则来暂时停止防火墙： (警告：这只适合在没有配置防火墙的环境中，如果已经配置过默认规则为deny的环境，此步骤将使系统的所有网络访问中断)
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
我想先备份一下系统自带的yum配置文件，再执行上面的操作
```
[root@client163 ~]# tar zcvf yumreposd.tar.gz /etc/yum.repos.d/     # tar.gz包被创建在/root下
[root@client163 ~]# mkdir /media/yum.repos.d/
[root@client163 ~]# cp yumreposd.tar.gz /media/yum.repos.d/   # 将压缩包复制到/media/yum.repos.d/文件夹下保存
[root@client163 yum.repos.d]# tar tvf yumreposd.tar.gz        # 不解压查看压缩包的文件
drwxr-xr-x root/root         0 2018-11-05 09:53 etc/yum.repos.d/
-rw-r--r-- root/root      1664 2018-11-23 21:16 etc/yum.repos.d/CentOS-Base.repo
-rw-r--r-- root/root      1309 2018-11-23 21:16 etc/yum.repos.d/CentOS-CR.repo
-rw-r--r-- root/root       649 2018-11-23 21:16 etc/yum.repos.d/CentOS-Debuginfo.repo
-rw-r--r-- root/root       630 2018-11-23 21:16 etc/yum.repos.d/CentOS-Media.repo
-rw-r--r-- root/root      1331 2018-11-23 21:16 etc/yum.repos.d/CentOS-Sources.repo
-rw-r--r-- root/root      5701 2018-11-23 21:16 etc/yum.repos.d/CentOS-Vault.repo
-rw-r--r-- root/root       314 2018-11-23 21:16 etc/yum.repos.d/CentOS-fasttrack.repo
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
  - OpenSSH配置文件，OpenSSH常用配置文件有两个/etc/ssh/ssh_config和/etc/sshd_config。
    - ssh_config为客户端配置文件
    - sshd_config为服务器端配置文件

# 3. sshd服务配置和管理    

介绍下配置文件/etc/ssh/sshd_config，以及需要安全调优的地方。注：参数前面有#，且#后没有空格，表示是默认缺省值。如要变更，需去除前面#，再该更保存，才能生效。

### 3.1 port22

设置sshd监听端口号
- SSH 预设使用 22 这个port，也可以使用多个port，即重复使用 port 这个设定项目！
- 例如想要开放sshd端口为 22和222，则多加一行内容为： Port 222 即可
- 然后重新启动sshd这样就好了。 建议大家修改 port number 为其它端口。防止别人暴力破解。

1. 修改配置文件
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
 17 Port 22             # 先不要注释掉端口22，否则下面配置失败后就再也不能远程了，测试222成功后再注释掉
 18 Port 222
```
2. 本来重启就成功来，没想到报错
```
[root@localhost ~]# systemctl restart sshd
Job for sshd.service failed because the control process exited with error code. 
See "systemctl status sshd.service" and "journalctl -xe" for details.
```
3. 按照上面的报错提示设置一下，查看到时防火墙禁止来222端口
```
[root@localhost ~]# journalctl -xe
*****  Plugin bind_ports (99.5 confidence) suggests   ************************
                                                     
 If you want to allow /usr/sbin/sshd to bind to network port 222
 Then you need to modify the port type.
 Do
 # semanage port -a -t PORT_TYPE -p tcp 222
     where PORT_TYPE is one of the following: ssh_port_t, vnc_port_t, xserver_port_t.
```
4. 按照报错提示的命令进行操作，竟然成功了
```
[root@localhost ~]# semanage port -a -t ssh_port_t -p tcp 222
[root@localhost ~]# systemctl restart sshd
[root@localhost ~]# systemctl status sshd
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2019-03-11 10:19:33 CST; 2min 41s ago
```
5. 查看防火墙ssh的端口,222和22都可以使用，但是远程登录依旧提示错误
```
[root@localhost ~]# semanage port -l |grep ssh
ssh_port_t                     tcp      222, 22
```

6. 抓紧看来下上面的步骤firewalld, selinux都没有关闭，因为换了台服务器做测试。
```
[root@localhost ~]# systemctl stop firewalld      # 先停止掉
[root@localhost ~]# systemctl disable firewalld   # 再禁止开机启动
[root@localhost ~]# vim /etc/selinux/config  
改：7 SELINUX=enforcing     #前面的7，表示文档中第7行。方便你查找
为：7 SELINUX=disabled
[root@localhost ~]# reboot 
```
7. 果然还是要关闭firewalld之后才可以ssh
防火墙设置参考[Centos7 修改SSH 端口](https://www.baidu.com/link?url=60dVRJChAmvON7vivW06OJZUPzJSnEAOQs3AA8_xaAyb-nC4JT30D9_nsxlERyn3Ik-zZccoyWeKH-9TY6H9oq&wd=&eqid=b67809ed0006069b000000065c85cbe3)

8. 查看端口状态
```
[root@localhost ~]# netstat -nlutp | grep sshd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      11599/sshd          
tcp        0      0 0.0.0.0:222             0.0.0.0:*               LISTEN      11599/sshd      
tcp6       0      0 :::22                   :::*                    LISTEN      11599/sshd          
tcp6       0      0 :::222                  :::*                    LISTEN      11599/sshd          
```
9. 更改端口后登陆 ```ssh -p 222 192.168.0.162```


### 3.2 ListenAddress 0.0.0.0
```
[root@localhost ~]# vim /etc/ssh/sshd_config 
 17 Port 22
 18 Port 222
 19 #AddressFamily any
 20 #ListenAddress 0.0.0.0
 21 #ListenAddress ::
 22 
```
设置sshd服务器绑定的IP 地址，0.0.0.0 表示侦听所有地址。这个值可以写成本地IP地址也可以写成所有地址

### 3.3 HostKey /etc/ssh/ssh_host_key
设置包含计算机私人密匙的文件
```
[root@localhost ~]# vim /etc/ssh/sshd_config 
 23 HostKey /etc/ssh/ssh_host_rsa_key
 24 #HostKey /etc/ssh/ssh_host_dsa_key
 25 HostKey /etc/ssh/ssh_host_ecdsa_key
 26 HostKey /etc/ssh/ssh_host_ed25519_key
```

### 3.4  SyslogFacility AUTHPRIV 
当有人使用 SSH 登入系统的时候，SSH 会记录信息，这个信息要记录的类型为AUTHPRIV。sshd服务日志存放在： /var/log/secure 
因为LogLevel INFO，登录记录的等级！INFO级别以上。
[定义ssh服务的日志类别为local0，编辑sshd服务的主配置文件](https://github.com/davidkorea/linux_study/blob/master/1_linux_baisc/Scheduled_Activity_and_Log_Management.md#2-%E5%AE%9A%E4%B9%89ssh%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%97%A5%E5%BF%97%E7%B1%BB%E5%88%AB%E4%B8%BAlocal0%E7%BC%96%E8%BE%91sshd%E6%9C%8D%E5%8A%A1%E7%9A%84%E4%B8%BB%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
```
[root@localhost ~]# vim /etc/ssh/sshd_config 
 31 # Logging
 32 #SyslogFacility AUTH
 33 #SyslogFacility AUTHPRIV
 34 #LogLevel INFO
```
关于日志相关信息 参考[Linux计划任务与日志管理
](https://github.com/davidkorea/linux_study/blob/master/1_linux_baisc/Scheduled_Activity_and_Log_Management.md#21-%E5%B8%B8%E8%A7%81%E6%97%A5%E5%BF%97%E6%96%87%E4%BB%B6%E7%9A%84%E4%BD%9C%E7%94%A8)
```
[root@localhost ~]# vim /etc/rsyslog.conf 
 56 # The authpriv file has restricted access.
 57 authpriv.*                                              /var/log/secure
```

### 3.5 安全调优的重点
- ```39 #LoginGraceTime 2m``` 优雅登录时间
  - 当使用者连上 SSH server 之后，会出现输入密码的画面，在该画面中，在多久时间内没有成功连上 SSH server 就强迫断线！若无单位则默认时间为秒

- ``` 40 #PermitRootLogin yes```
  - 是否允许 root 登入！预设是允许的，但是建议设定成 no ！真实的生产环境服务器，是不允许root账号直接登陆的！！！

- ``` 67 PasswordAuthentication yes```
  -  密码验证当然是需要的！所以这里写 yes，也可以设置为no. 在真实的生产服务器上，根据不同安全级别要求，有的是设置不需要密码登陆的，通过认证的秘钥来登陆

- ``` 66 #PermitEmptyPasswords no```
  -  若上面那一项如果设定为 yes 的话，这一项就最好设定为 no，这个项目在是否允许以空的密码登入！当然不许！

- ```107 #PrintMotd yes```
  - 登入后是否显示出一些信息呢？例如上次登入的时间、地点等等，预设是 yes,亦即是打印出 /etc/motd这个文档的内容，默认是个空文档。
  - 添加警告信息
    ```
    [root@localhost ~]# echo "Warning! From now on, all of your operations have been recorded." > /etc/motd 
    [root@localhost ~]# cat /etc/motd 
    Warning! From now on, all of your operations have been recorded.
    ```
- ```108 #PrintLastLog yes```, 显示上次登入的信息！预设也是 yes  
  - ssh登录成功后 会有如下显示
    ```
    Last login: Mon Mar 11 11:03:32 2019 from 192.168.0.219
    Warning! From now on, all of your operations have been recorded.
    ```

- ```117 #UseDNS yes```
  - 一般来说，为了要判断客户端来源是正常合法的，因此会使用 DNS 去反查客户端的主机名。不过如果是在内网互连，这项目设定为 no 会让联机速度比较快。

# 4. sshd服务防止暴力破解

1. 密码足够的复杂，密码的长度要大于8位最好大于20位。密码的复杂度是密码要尽可能有数字、大小写字母和特殊符号混合组成，
2. 修改默认端口号
3. 不允许root账号直接登陆，添加普通账号，授予root的权限。是否可以禁止root身份登录？ 不行，因为有些程序需要使用root身份登录并运行。另外判断一个用户是不是超级管理员，看的是用户的ID是否为0。
4. 不允许密码登陆，只能通过认证的秘钥来登陆系统

### 4.1 通过密钥认证实现sshd认证

#### 1. client
- 客户端生成密钥对，然后把公钥传输到服务端
  ```
  [root@client163 ~]# ssh-keygen
  Generating public/private rsa key pair.
  Enter file in which to save the key (/root/.ssh/id_rsa): 
  Enter passphrase (empty for no passphrase): 
  Enter same passphrase again: 
  Your identification has been saved in /root/.ssh/id_rsa.
  Your public key has been saved in /root/.ssh/id_rsa.pub.
  The key fingerprint is:
  SHA256:U5H4ITsNZKM2eQqrMLjKsZAsuYrFn/dc7lAOHrtzZ8U root@client163
  The key's randomart image is:
  +---[RSA 2048]----+
  |       .+...     |
  |       ++.o.     |
  |    . = .*..     |
  |.    + +o.o      |
  |+   . . S..  .   |
  |.B .   . B    E  |
  |*o+     + o  .   |
  |*oo. ....=. o    |
  |*o  o. .++oo     |
  +----[SHA256]-----+
  ```
- 发布公钥到服务端 ```ssh-copy-id -i 192.168.0.162```使用默认22端口进行ssh通信
  ```shell
  [root@client163 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub -p 222 root@192.168.0.162      # 指定ssh端口222
  /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
  The authenticity of host '[192.168.0.162]:222 ([192.168.0.162]:222)' can't be established.
  ECDSA key fingerprint is SHA256:2jRM2HatzcmG7TN+NPtKjv9LTQkNSuhDKGby2x+JrRI.
  ECDSA key fingerprint is MD5:62:23:4c:dd:4d:43:07:bd:8c:c2:29:07:f5:42:ab:d0.
  Are you sure you want to continue connecting (yes/no)? yes
  /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
  /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
  root@192.168.0.162's password: 

  Number of key(s) added: 1

  Now try logging into the machine, with:   "ssh -p '222' 'root@192.168.0.162'"
  and check to make sure that only the key(s) you wanted were added.
  ```



### 4.2 通过开源的防护软件来防护安全      
简单、灵活、功能强大

实战背景：
最近公网网站一直被别人暴力破解sshd服务密码。虽然没有成功，但会导致系统负载很高，原因是在暴力破解的时候，系统会不断地认证用户，从而增加了系统资源额外开销，导致访问公司网站速度很慢。

#### 1. fail2ban

fail2ban可以监视你的系统日志，然后匹配日志的错误信息（正则式匹配）执行相应的屏蔽动作（一般情况下是防火墙），而且可以发送e-mail通知系统管理员，很好、很实用、很强大！简单来说其功能就是防止暴力破解。工作的原理是通过分析一定时间内的相关服务日志，将满足动作的相关IP利用iptables加入到dorp列表一定时间。 
注：重启iptables服务的话，所有drop将重置。

http://www.fail2ban.org

- steps (refer to README.md in package folder)
1. ```tar zxvf fail2ban-0.9.4.tar.gz```
2. ```cd fail2ban-0.9.4```
3. ```python setup.py install```
4. ```[root@localhost ail2ban-0.9.4]# cp files/redhat-initd /etc/rc.d/init.d/fail2ban```
  the system init/service script is not automatically installed.To enable fail2ban as an automatic service, simply copy the script for your distro from the `files` directory to `/etc/rc.d/init.d/`
5. ```[root@localhost ail2ban-0.9.4]# chkconfig --add fail2ban```  #开机自动启动

你怎么知道要复制这个文件？ 一个新的软件包，后期怎么可以知道哪个文件是启动脚本文件？这就要找服务器启动脚本文件中有什么特点，然后过滤出来对应的文件名。
```shell
[root@localhost fail2ban-0.9.4]# grep chkconfig ./* -R --color
./files/redhat-initd:# chkconfig: - 92 08             # 启动脚本里都包含chkconfig 字段
```
- 相关主要文件说明：
  - ```/etc/fail2ban/action.d``` #动作文件夹，内含默认文件。iptables以及mail等动作配置
  - ```/etc/fail2ban/fail2ban.conf```    #定义了fai2ban日志级别、日志位置及sock文件位置
  - ```/etc/fail2ban/filter.d```                    #条件文件夹，内含默认文件。过滤日志关键内容设置
  - ```/etc/fail2ban/jail.conf```     #主要配置文件，模块化。主要设置启用ban动作的服务及动作阀值


- 应用实例
设置条件：ssh远程登录5分钟内3次密码验证失败，禁止用户IP访问主机1小时，1小时该限制自动解除，用户可重新登录。因为动作文件（action.d/iptables.conf）以及日志匹配条件文件（filter.d/sshd.conf ）安装后是默认存在的。基本不用做任何修改。所有主要需要设置的就只有jail.conf文件。启用sshd服务的日志分析，指定动作阀值即可。
  - 实例文件/etc/fail2ban/jail.conf及说明如下：
    ```shell
      [DEFAULT]                   #全局设置
      ignoreip = 127.0.0.1/8      #忽略的IP列表,不受设置限制
      bantime  = 600              #屏蔽时间，单位：秒
      findtime  = 600             #这个时间段内超过规定次数会被ban掉
      maxretry = 3                #最大尝试次数
      backend = auto              #日志修改检测机制（gamin、polling和auto这三种）
    ```

