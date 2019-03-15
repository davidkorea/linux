# FTP & NFS

# 1. FTP(vsftp)
## 1.1 FTP服务器概述
- FTP服务器（File Transfer Protocol Server）是在互联网上提供文件存储和访问服务的计算机，它们依照FTP协议提供服务。
- 常见FTP服务器：
  - windows：Serv-U FTP Server，filezilla_server
  - Linux：ProFTPD:（Professional FTP daemon）一个Unix平台上或是类Unix平台上（如Linux, FreeBSD等）的FTP服务器程序。
  - VSFTP是一个基于GPL发布的类Unix系统上使用的FTP服务器软件，它的全称是Very Secure FTP 从此名称可以看出来，编制者的初衷是代码的安全。它是一个安全、高速、稳定的FTP服务器。port 20 (传数据) , port 21 (传指令)
- FTP只通过TCP连接,没有用于FTP的UDP组件.FTP不同于其他服务的是它使用了两个端口, 一个数据端口和一个命令端口(或称为控制端口)。通常21端口是命令端口，20端口是数据端口。当混入主动/被动模式的概念时，数据端口就有可能不是20了。
- 工作模式： 主动模式，被动模式
  - **主动模式**下FTP客户端从任意的非特殊的端口（N > 1023）连入到FTP服务器的命令端口--21端口。然后客户端在N+1（N+1 >= 1024）端口监听，并且通过N+1（N+1 >= 1024）端口发送命令给FTP服务器。服务器会反过来连接用户本地指定的数据端口，比如20端口。以服务器端防火墙为立足点，要支持主动模式FTP需要打开如下交互中使用到的端口：
    - FTP服务器命令（21）端口接受客户端任意端口（客户端初始连接）
    - FTP服务器命令（21）端口到客户端端口（>1023）（服务器响应客户端命令）
    - FTP服务器数据（20）端口到客户端端口（>1023）（服务器初始化数据连接到客户端数据端口）
    - FTP服务器数据（20）端口接受客户端端口（>1023）（客户端发送ACK包到服务器的数据端口）
    ![](https://images2017.cnblogs.com/blog/564326/201710/564326-20171012115944777-931494355.jpg)
    
    在第1步中，客户端的命令端口与FTP服务器的命令端口建立连接，并发送命令“PORT 1027”。然后在第2步中，FTP服务器给客户端的命令端口返回一个"ACK"。在第3步中，FTP服务器发起一个从它自己的数据端口（20）到客户端先前指定的数据端口（1027）的连接，最后客户端在第4步中给服务器端返回一个"ACK"。
    
    - 主动方式FTP的主要问题实际上在于客户端。FTP的客户端并没有实际建立一个到服务器数据端口的连接，它只是简单的告诉服务器自己监听的端口号，服务器再回来连接客户端这个指定的端口。对于客户端的防火墙来说，这是从外部系统建立到内部客户端的连接，这是通常会被阻塞的。
    
  - **被动模式**为了解决服务器发起到客户的连接的问题，人们开发了一种不同的FTP连接方式。这就是所谓的被动方式，或者叫做PASV，当客户端通知服务器它处于被动模式时才启用。在被动方式FTP中，命令连接和数据连接都由客户端，这样就可以解决从服务器到客户端的数据端口的入方向连接被防火墙过滤掉的问题。当开启一个FTP连接时，客户端打开两个任意的非特权本地端口（N >; 1024和N+1）。第一个端口连接服务器的21端口，但与主动方式的FTP不同，客户端不会提交PORT命令并允许服务器来回连它的数据端口，而是提交PASV命令。这样做的结果是服务器会开启一个任意的非特权端口（P >; 1024），并发送PORT P命令给客户端。然后客户端发起从本地端口N+1到服务器的端口P的连接用来传送数据。对于服务器端的防火墙来说，必须允许下面的通讯才能支持被动方式的FTP:

    - FTP服务器命令（21）端口接受客户端任意端口（客户端初始连接）
    - FTP服务器命令（21）端口到客户端端口（>1023）（服务器响应客户端命令）
    - FTP服务器数据端口（>1023）接受客户端端口（>1023）（客户端初始化数据连接到服务器指定的任意端口）
    - FTP服务器数据端口（>1023）到客户端端口（>1023）（服务器发送ACK响应和数据到客户端的数据端口）
    ![](https://images2017.cnblogs.com/blog/564326/201710/564326-20171012120001793-1278247588.jpg)
    
    在第1步中，客户端的命令端口与服务器的命令端口建立连接，并发送命令“PASV”。然后在第2步中，服务器返回命令"PORT 2024"，告诉客户端（服务器）用哪个端口侦听数据连接。在第3步中，客户端初始化一个从自己的数据端口到服务器端指定的数据端口的数据连接。最后服务器在第4 步中给客户端的数据端口返回一个"ACK"响应。

    - 被动方式的FTP解决了客户端的许多问题，但同时给服务器端带来了更多的问题。最大的问题是需要允许从任意远程终端到服务器高位端口的连接。幸运的是，许多FTP守护程序，包括流行的WU-FTPD允许管理员指定FTP服务器使用的端口范围。详细内容参看附录1。 
    - 第二个问题是客户端有的支持被动模式，有的不支持被动模式，必须考虑如何能支持这些客户端，以及为他们提供解决办法。例如，Solaris提供的FTP命令行工具就不支持被动模式，需要第三方的FTP客户端，比如ncftp。
    - 随着WWW的广泛流行，许多人习惯用web浏览器作为FTP客户端。大多数浏览器只在访问ftp:// 这样的URL时才支持被动模式。这到底是好还是坏取决于服务器和防火墙的配置。

## 1.2 安装配置VSFTP
#### 1. 安装
1. 服务端
```
[root@server162 ~]# yum install vsftpd lftp -y
```
2. 客户端
```
[root@client163 ~]# yum install -y lftp
```
  从RHEL6开始，系统镜像中默认没有ftp客户端命令。取而代之的是lftp命令。lftp 是一个功能强大的下载工具，它支持访问文件的协议: ftp, ftps, http, https, hftp, fish.(其中ftps和https需要在编译的时候包含openssl库)。llftp的界面非常好一个shell: 有命令补全，历史记录，允许多个后台任务执行等功能，使用起来非常方便。它还有书签、排队、镜像、断点续传、多进程下载等功能。
#### 2. 配置文件
```
[root@server162 ~]# ls /etc/vsftpd/
ftpusers                vsftpd.conf             
user_list               vsftpd_conf_migrate.sh  
```
1. /etc/vsftpd/ftpusers：用于指定哪些用户不能访问FTP 服务器。  黑名单
```
[root@server162 ~]# cat /etc/vsftpd/ftpusers 
# Users that are not allowed to login via ftp
root
bin ... ...
```
2. /etc/vsftpd/user_list：可以用来允许使用vsftpd 的用户列表文件的白名单。但userlist_deny= YES（默认）不允许在这个文件中的用户登录ftp，甚至不提示输入密码prompt 提示
```
[root@server162 ~]# cat /etc/vsftpd/user_list 
# vsftpd userlist
# If userlist_deny=NO, only allow users in this file
# If userlist_deny=YES (default), never allow users in this file, and # 其实默认也是黑名单
# do not even prompt for a password.
# Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
# for users that are denied.
root
bin ... ...
```
3. /etc/vsftpd/vsftpd.conf：vsftpd 的核心配置文件
4. /etc/vsftpd/vsftpd_conf_migrate.sh：是vsftpd 操作的一些变量和设置脚本。/var/ftp/：默认情况下匿名用户的根目录
#### 3. 启动vsftp服务
ftp 默认允许匿名登录，所以启动服务号可以直接登录，但是只读权限/var/ftp/pub/
1. 服务端启动服务
```
[root@server162 ~]# systemctl start vsftpd
[root@server162 ~]# systemctl enable vsftpd
[root@server162 ~]# netstat -anutp | grep ftp   # 没有20号端口，是因为没有传输数据
tcp6       0      0 :::21       :::*      LISTEN      38886/vsftpd   
```
2. 客户端连接
```
[root@client163 ~]# lftp 192.168.0.162
lftp 192.168.0.162:~> pwd
ftp://192.168.0.162
lftp 192.168.0.162:~> ls
drwxr-xr-x    2 0      0       6 Oct 30 19:45 pub
```
> ![](https://i.loli.net/2019/03/15/5c8b188f8512b.png)
> ![](https://i.loli.net/2019/03/15/5c8b190e9bcfe.png)

## 1.3 实战：匿名访问VSFTP
> 公司技术部准备搭建一台功能简单的FTP 服务器，允许所有员工上传和下载文件，并允许创建用户自己的目录。
> 
> 分析：允许所有员工上传和下载文件需要设置成允许匿名用户登录并且需要将允许匿名用户上传功能, anon_mkdir_write_enable可以控制是否允许匿名用户创建目录。

#### 1. 更改配置文件
```shell
[root@server162 ~]# vim /etc/vsftpd/vsftpd.conf 

 29 anon_upload_enable=YES        # 取消掉此行注释#
 30 #
 31 # Uncomment this if you want the anonymous FTP user to be able to create
 32 # new directories.
 33 anon_mkdir_write_enable=YES   # 取消掉此行注释#
```
#### 2. 更改ftp文件目录读写权限
- 虽然ftp的配置文件已经允许匿名用户上传和写入文件，但是文件目录的权限不允许拥有者之外有写入权限
```
[root@server162 ~]# ll -d /var/ftp/pub/
drwxr-xr-x 2 root root 6 Oct 31 03:45 /var/ftp/pub/
```
- 所以需要加入ftp服务使用用户的写入权限。首先查看哪些用户在使用ftp服务
  - root拥有文件目录，不用管
  - nobody是匿名用户
  - ftp是ftp服务使用的账户
```
[root@server162 ~]# ps aux | grep vsftpd
root     16334  0.0  0.0  53272   724 ?        Ss   11:39   0:00 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf
nobody   17004  0.0  0.0  55396  1484 ?        Ss   11:40   0:00 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf
ftp      17006  0.0  0.0  57504  1480 ?        S    11:40   0:00 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf
root     21032  0.0  0.0 112680   700 pts/2    S+   11:44   0:00 grep --color=auto vsftpd
```
- 将目录/var/ftp/pub/的主组完全转给用户ftp
```
[root@server162 ~]# chown ftp.ftp /var/ftp/pub/     # chown ftp:ftp /var/ftp/pub
[root@server162 ~]# ll -d /var/ftp/pub/
drwxr-xr-x 2 ftp ftp 6 Oct 31 03:45 /var/ftp/pub/
```
- 此时可以上传，创建文件。但是删除和重命名并不允许
![](https://i.loli.net/2019/03/15/5c8b23079b53a.png)

#### 3. 开启匿名用户全部权限 anon_other_write_enable=YES
> **Issue**
> ```
> [root@server162 ~]# systemctl start vsftpd
> Mar 15 13:15:07 server162 vsftpd[32918]: 500 OOPS: bad bool value in config file for: anon_other_write_enable
> ```
> 
> 配置文件中的有效行，每行命令的任何地方不许有空格，=前后，命令结尾YES后，都不可以。参考： [vsftpd 500 OOPS: bad bool value in config file for: anon_world_readable_only](https://blog.csdn.net/zhaoyangjian724/article/details/45841655)

```
[root@server162 ~]# vim /etc/vsftpd/vsftpd.conf 

 29 anon_upload_enable=YES
 30 #
 31 # Uncomment this if you want the anonymous FTP user to be able to create
 32 # new directories.
 33 anon_mkdir_write_enable=YES
 34 anon_other_write_enable=YES       # 默认没有这一行命令，手动添加
 
[root@server162 ~]# systemctl start vsftpd
```
此时可以删除，重命名文件。但是这个参数对匿名用户来说权限太大,不安全，均衡使用这个参数

注意，默认匿名用户家目录的权限是755（rwx r-x r-x），这个权限是不能改变的。切记！
    ![](https://i.loli.net/2019/03/15/5c8b390455f9e.png)

#### 4. 创建一个公司上传用的目录
下面我们来一步一步的实现,先修改目录权限,创建一个公司上传用的目录xeroxdata,设置拥有者为ftp用户所有，目录权限是755。不允许删除，取消配置文件中的参数anon_other_write_enable=YES
```
[root@server162 ~]# mkdir /var/ftp/xeroxdata
[root@server162 ~]# chown ftp:ftp /var/ftp/xeroxdata/
[root@server162 ~]# ll -d /var/ftp/xeroxdata/
drwxr-xr-x 2 ftp ftp 6 Mar 15 13:37 /var/ftp/xeroxdata/
```
**强烈部件是使用匿名用户**

## 1.4 实战：用户名密码方式访问VSFTP

> 公司内部现在有一台FTP和WEB服务器，FTP 的功能主要用于维护公司的网站内容，包括上传文件、创建目录、更新网页等等。公司现有两个部门负责维护任务，他们分别适用team1 和team2帐号进行管理。先要求仅允许team1 和team2 帐号登录FTP 服务器，但不能登录本地系统，并将这两个帐号的根目录限制为/var/www/html，不能进入该目录以外的任何目录。
> 
> 分析：将FTP 和WEB 服务器做在一起是企业经常采用的方法，这样方便实现对网站的维护，为了增强安全性，首先需要使用仅允许本地用户访问，并禁止匿名用户登录。其次使用chroot 功能将team1和team2 锁定在/var/www/html 目录下。如果需要删除文件则还需要注意本地权限

#### 1. 创建一个系统用户，但是不允许登录服务器
```
[root@server162 ~]# useradd -s /sbin/nologin team1
[root@server162 ~]# echo "11111" | passwd --stdin team1
Changing password for user team1.
passwd: all authentication tokens updated successfully.

[root@server162 ~]# tail -2 /etc/passwd       # 确认权限，的确服务使用的账号都是nologin，不熏晕登录服务器的
dhcpd:x:177:177:DHCP server:/:/sbin/nologin
team1:x:1006:1006::/home/team1:/sbin/nologin
```
#### 2. 配置vsftpd.conf
```
[root@server162 ~]# vim /etc/vsftpd/vsftpd.conf 

 12 anonymous_enable=NO             # 禁止匿名登录
 16 local_enable=YES                # 允许系统创建的本地用户，默认已开启


 33 #anon_mkdir_write_enable=YES    # 注释掉之前的匿名权限，不注释也可以，上面已经全部禁止匿名来
 34 #anon_other_write_enable=YES

102 #chroot_local_user=YES
103 local_root=/var/www/html        # 新增这一行，指定ftp可以访问的本地目录
104 chroot_list_enable=YES          # 激活chroot 功能
105 # (default follows)
106 chroot_list_file=/etc/vsftpd/chroot_list     # 此文件存放要锁定的用户名，不存在，需要自己创建
107 allow_writeable_chroot=YES      # 新增这一行，允许锁定的用户有写的权限
108 #
```
#### 3. 建立/etc/vsftpd/chroot_list文件，添加team1 和team2 帐号
```
[root@server162 ~]# vim /etc/vsftpd/chroot_list
  team1   # 一个用户名，一行
```

#### 4. 修改本地目录权限， 重启vsftpd服务使配置生效
```
[root@server162 ~]# ll -d /var/www/html/
drwxrwxr-x+ 2 root root 4096 Mar 14 09:41 /var/www/html/

[root@server162 ~]# chmod -R o+w /var/www/html/
[root@server162 ~]# ll -d /var/www/html/
drwxrwxrwx+ 2 root root 4096 Mar 14 09:41 /var/www/html/

[root@server162 ~]# systemctl restart vsftpd
```
测试客户端登录
1. lftp
```shell
[root@client163 ~]# lftp 192.168.0.162 -u team1,11111   # -u用户，用户名和密码用,隔开，不能有空格
lftp team1@192.168.0.162:~> ls
ls: Login failed: 530 Login incorrect.                  # 报错！！
lftp team1@192.168.0.162:~> 
```
2. windows 同样一致显示登录界面

  ![](https://i.loli.net/2019/03/15/5c8b582663319.png)

#### Issue：Login failed: 530 Login incorrect.  
【fixed】参考：[ftp vsftpd 530 login incorrect 解决办法汇总](https://blog.csdn.net/wlchn/article/details/50855447)
- 查看vsftp配置文件中pam服务名称=vsftp，正确
```
[root@server162 ~]# vim /etc/vsftpd/vsftpd.conf 
  129 pam_service_name=vsftpd
```
- 去到/etc/pam.d/vsftpd，注释掉第四行#auth  required    pam_shells.so
```
[root@server162 ~]# vim /etc/pam.d/vsftpd 

  1 #%PAM-1.0
  2 session    optional     pam_keyinit.so    force revoke
  3 auth       required     pam_listfile.so item=user sense=deny file=/etc/vsftpd/f    tpusers onerr=succeed
  4 #auth       required    pam_shells.so
  5 auth       include      password-auth
  6 account    include      password-auth
  7 session    required     pam_loginuid.so
  8 session    include      password-auth
```
- 重启vsftpd服务，客户端登录成功

  ![](https://i.loli.net/2019/03/15/5c8b5a2a482bb.png)
  
  
  
  
## 1.5 使用SSL证书加密数据传输
FTP与HTTP一样缺省状态都是基于明文传输，希望FTP服务器端与客户端传输保证安全，可以为FTP配置SSL

#### 1. 使用OpenSSL生成自签证书
- OpenSSL 简单参数解释:
  - req - 是 X.509 Certificate Signing Request （CSR，证书签名请求）管理的一个命令。
  - x509 - X.509 证书数据管理。
  - days - 定义证书的有效日期。
  - newkey - 指定证书密钥处理器。
  - keyout - 设置密钥存储文件。
  - out - 设置证书存储文件，注意证书和密钥都保存在一个相同的文件  
  
```shell
[root@server162 ~]# openssl req -new -x509 -nodes -out vsftpd.pem -keyout vsftpd.pem -days 3650
Generating a 2048 bit RSA private key
.....+++
..........................................................................................................+++
writing new private key to 'vsftpd.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:Kr
State or Province Name (full name) []:Seoul
Locality Name (eg, city) [Default City]:Seoul
Organization Name (eg, company) [Default Company Ltd]:Xerox
Organizational Unit Name (eg, section) []:SBG
Common Name (eg, your name or your server's hostname) []:server162
Email Address []:ftp@server162.com
```
#### 2. 创建证书文件存放目录
```
[root@server162 ~]# mkdir /etc/vsftpd/.sslkey                   # 创建隐藏路径

[root@server162 ~]# cp vsftpd.pem /etc/vsftpd/.sslkey/          # 复制pem文件至隐藏路径

[root@server162 ~]# chmod 400 /etc/vsftpd/.sslkey/vsftpd.pem    # 更改pem文件权限为400
[root@server162 ~]# ll !$
ll /etc/vsftpd/.sslkey/vsftpd.pem
-r-------- 1 root root 3095 Mar 15 17:47 /etc/vsftpd/.sslkey/vsftpd.pem
```
### 3. 修改配置文件,支持SSL
```shell
[root@server162 ~]# vim /etc/vsftpd/vsftpd.conf     # 手动添加下面命令

  119 # config ssl, add below commands
  120 ssl_enable=YES
  121 allow_anon_ssl=NO
  122 force_local_data_ssl=YES
  123 force_local_logins_ssl=YES
  124 force_anon_logins_ssl=YES
  125 force_anon_data_ssl=YES
  126 ssl_tlsv1=YES
  127 ssl_sslv2=YES
  128 ssl_sslv3=YES
  129 require_ssl_reuse=NO
  130 ssl_ciphers=HIGH
  131 rsa_cert_file=/etc/vsftpd/.sslkey/vsftpd.pem
  132 rsa_private_key_file=/etc/vsftpd/.sslkey/vsftpd.pem
```
- 参数解释： 
  - 表示强制匿名用户使用加密登陆和数据传输
    - ssl_enable=YES     #启用SSL支持
    - allow_anon_ssl=NO 
    - force_local_data_ssl=YES   
    - force_local_logins_ssl=YES
    - force_anon_logins_ssl=YES
    - force_anon_data_ssl=YES
  - ssl_tlsv1=YES   #指定vsftpd支持TLS v1[
  - ssl_sslv2=YES   #指定vsftpd支持SSL v2
  - ssl_sslv3=YES   #指定vsftpd支持SSL v3
  - require_ssl_reuse=NO   #不重用SSL会话,安全配置项 
  - ssl_ciphers=HIGH    #允许用于加密 SSL 连接的 SSL 算法。这可以极大地限制那些尝试发现使用存在缺陷的特定算法的攻击者
  - rsa_cert_file=/etc/vsftpd/.sslkey/vsftpd.pem  #定义 SSL 证书和密钥文件的位置
  - rsa_private_key_file=/etc/vsftpd/.sslkey/vsftpd.pem

**注意:上面的配置项不要添加到vsftpd.conf 文件最后,否则启动报错**
  
#### 4. 配置FileZilla客户端验证

![](https://i.loli.net/2019/03/15/5c8bc12d96622.png)
![](https://i.loli.net/2019/03/15/5c8bc12d9a04b.png)

- [X]但是在ftp客户端链接到时候依旧可以选择使用无证书加密到方式链接。是否使用ssl认证和客户端配置有关系，如上图配置
  - NO,NO,NO 不是这样的，ftp服务器开启了使用ssl，所以浏览器，windows资源管理器无法访问。只能用专门的客户端来ssl链接ftp
  - linux lftp 需要做如下更改，才可以正常访问。参考：[lftp提示Fatal Error: Certificate Verification: Not Trusted错误的解决方法-linux](https://blog.csdn.net/nnaiwa/article/details/81044046)
    ```shell
    [root@client12 ~]# vim /etc/lftp.conf 

      8 # add this command
      9 set ssl:verify-certificate no
    ```

    
- 注意: 在工作中,内网FTP传输,可以不用证书加密传输。**如果FTP服务器在公网,为了数据的安全性,就一定要配置证书加密传输**
 
      
            
# 2. NFS network file system

rpcbind
nfc-utils


      
      
      
      
      
      
      
      
      
      
      
