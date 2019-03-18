# 搭建无人执守安装服务器 - 批量安装

- 安装三四十台机器，一般就需要无人值守安装了
- 从网卡启动
- 新购买服务器 - 上架 - RAID - 重启，快捷键，从网卡启动 - DHCP server 
<p align="center">
  <img src="https://i.loli.net/2019/03/18/5c8f1368ec875.png" width=300, height=300>
</p>
  将上面三个服务安装在一台服务器上

- 安装的时候，最开始，通过快捷键进入网络引导模式。不建议更改第一启动位置。因为安装好系统，重启还会再次通过网络引导安装系统
- ```lsof -i：69```， 查看某个端口被那个进程占用


- 更改vsftp设置
```
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
