# FTP & NFS

# 1. FTP(vsftp)
## 1.1 VSFTP服务器概述
- FTP服务器（File Transfer Protocol Server）是在互联网上提供文件存储和访问服务的计算机，它们依照FTP协议提供服务。
- 常见FTP服务器：
  - windows：Serv-U FTP Server，filezilla_server
  - Linux：ProFTPD:（Professional FTP daemon）一个Unix平台上或是类Unix平台上（如Linux, FreeBSD等）的FTP服务器程序。

## 1.2 安装配置VSFTP
## 1.3 实战：匿名访问VSFTP
## 1.4 实战：用户名密码方式访问VSFTP


1. 支持匿名传输 在pub目录下 
  -  配置文件给匿名权限
  - /var/ftp/pub 改主组权限chown ftp.ftp /ver/ftp/pub 才可以上传
  - 如果不知道当前服务是哪个用户在哦运行，那么目录权限全部改了 755 或766，一般766就够了
    - 其实可以ps aux | grep vsftp 来查看当前服务是哪个用户在使用
  - anom_other_write_enable=YES 不建议这个开启
    - 一般情况下 匿名用户 只读，执行，写，删除权限不可有
    
2. 系统用户
  - ftp web一起使用，为了安全 
  - 创建一个系统用户，但是不允许登陆服务器
    - ```useradd -s /sbin/mologin team1```
    - ```echo "11111" | passwd --stdin team
    - 修改配置文件
      ```
      anonymous_enable=NO
      local_enable=YES    # 允许系统创建的本地用户
      101 local_root=/var/www/html      # 新添加这一样
      102 chroot_list_enable=YES
      104 chroot_list_file=/etc/vsftp/chroot_list   # 不存在，需要自己创建，要锁定用户的用户名
      105 allow_writeable_chroot=YES      # 允许锁定的用户有写的权限
      ```
    - 创建名单
      ```
      vim /etc/vsftp/chroot_list
      
        team1   # 一个用户名，一行
        team2
      ```
    - 创建限定的目录
      ```
      mkdir -p /var/www/html
      chmod -R o+w /var/www/html    # a+w 全新啊有点大 
      ```
- FTP证书，当ftp服务器放在公网的时候，为了安全 。sftp 相比ftp 大文件传输特别慢
  - openssl req -new x509 -node -out vsftp.pem -keyout vsftp.pem -days 3650
  - mkdir /etc/vsftp/.sslkey # 吧pem文件拷贝到这个路径
  - chmod 400 vsftp.pem
  ```
  ssl_enable=YES     #启用SSL支持
  allow_anon_ssl=NO 
   force_local_data_ssl=YES   
  force_local_logins_ssl=YES
  force_anon_logins_ssl=YES
  force_anon_data_ssl=YES
  #上面四行force 表示强制匿名用户使用加密登陆和数据传输
  ssl_tlsv1=YES   #指定vsftpd支持TLS v1[
  ssl_sslv2=YES   #指定vsftpd支持SSL v2
  ssl_sslv3=YES   #指定vsftpd支持SSL v3
  require_ssl_reuse=NO   #不重用SSL会话,安全配置项 
  ssl_ciphers=HIGH    #允许用于加密 SSL 连接的 SSL 算法。这可以极大地限制那些尝试发现使用存在缺陷的特定算法的攻击者
  rsa_cert_file=/etc/vsftpd/.sslkey/vsftpd.pem 
  rsa_private_key_file=/etc/vsftpd/.sslkey/vsftpd.pem
  #定义 SSL 证书和密钥文件的位置

  ```
  - 重启服务
  - 但是在ftp客户端链接到时候依旧可以选择使用无证书加密到方式链接。和客户端有关系
      
# 2. NFS network file system

rpcbind
nfc-utils


      
      
      
      
      
      
      
      
      
      
      
