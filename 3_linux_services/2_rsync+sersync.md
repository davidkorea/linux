# rsync+sersync

1. rsync基于ssh协议的远程传输文件协议
2. scp远程传输文件协议
  ```
  ##### 单个文件 #####
  [root@server162 ~]# scp print.sh root@192.168.0.163:/root
  The authenticity of host '192.168.0.163 (192.168.0.163)' can't be established.
  ECDSA key fingerprint is SHA256:TDObcVvc4d/BfgGvlfoUD1dd5a9+t9B2nSq6gdbSEoY.
  ECDSA key fingerprint is MD5:28:3d:cc:60:dc:71:2d:b3:c2:37:1d:3a:32:c2:00:1b.
  Are you sure you want to continue connecting (yes/no)? yes
  Warning: Permanently added '192.168.0.163' (ECDSA) to the list of known hosts.
  root@192.168.0.163's password: 
  print.sh                                             100%  108    10.3KB/s   00:00  
  
  ##### 整个目录 -r #####
  [root@server162 ~]# scp -r a/ root@192.168.0.163:/root
  root@192.168.0.163's password: 
  1.txt                                                100%    0     0.0KB/s   00:00 
  ```
3. rsync 第一次全部传输，之后增量传输，只传输新增和变化修改过的文件。边复制，边统计，边比较
4. rsync 但是如果文件损坏，一样会传输。之前好的文件也会被同步为损坏的文件
5. rsync 镜像保存整个目录树和文件系统， 保持原来文件的权限、时间、软硬。支持匿名传输，以方便进行网站镜象。选择性保持：符号连接，硬链接，文件属性，权限，时间等
5. 常用于定时备份，完整备份，差量备份，增量备份
6. 名词的解释：
    - 发起端：负责发起rsync同步操作的客户机叫做发起端，通知服务器我要备份你的数据 / 客户端：存放备份数据
    - 备份源：负责相应来自客户机rsync同步操作的服务器脚在备份源，需要备份的服务器 / 服务端：运行rsyncd服务，一般来说，需要备份的服务器
  
7. 数据同步方式
    - 推push：一台主机负责把数据传送给其他主机，服务器开销很大，比较适合后端服务器少的情况。
      - 推：目的主机配置为rsync服务器，源主机周期性的使用rsync命令把要同步的目录推过去（需要备份的机器是客户端，存储备份的机器是服务端）
    - 拉pull：所有主机定时去找一主机拉数据，可能就会导致数据缓慢
      - 拉：源主机配置为rsync服务器，目的主机周期性的使用rsync命令把要同步的目录拉过来（需要备份的机器是服务端，存储备份的机器是客户端）
两种方案，rsync都有对应的命令来实现
8. 本教程使用push的方式进行同步
    - 你推送给我，我就是服务端服。需要备份的机器是客户端，存储备份的机器是服务端
    - 传给谁，谁安装rsync服务端，但是两边都要安装rsync服务
9. rsync小服务由超级互联网守护进程服务xinetd统一管理
10. rsync 常用参数 -avz， --delete 完全一致 在2个机器之间
11. 同步162的/var/www/html 到163的/web-back. 163是服务端。单独创建一个用户进行传输。可以更改用户和属组权限给web-back
12. 162，163两个机器都要创建同样的一个用户，将要备份的目录的权限给到这个用户
13. ```rsync -avz --delete /var/www/html rget1@192.168.0.163:/web-back```，传输html文件夹
    - ```rsync -avz --delete /var/www/html/ rget1@192.168.0.163:/web-back```，今川氏html文件夹下面的文件
    - 但是 生产环境中 不加delete，以免将备份数据清空，或者服务器被黑了，备份数据也跟着被删除

14.用非系统用户，进行文件传输。对于安全级别比较高的服务器更适合一些。需要更改配置文件/etc/rsyncd.comf。centos7已存在模版，但是6上面没有
    - 163服务端
    ```
      uid = root    用root权限
      gid = root
      address = 192.168.0.163     本机ip地址
      port = 873
      host allow = 192.168.0.0/24     允许整个望断
      use chroot = yes
      max connections = 4    最大链接数
      pid file = /var/run/rsyncd.pid
      lock file = /var/run/rsync/lock
      log file = /var/log/rsync.log
      motd file = /etc/rsync.motd     没有此文件，需要自行创建
      
      [wwwroot]       模块名，自己定义一个模块，名称自己定
      path = /web-back/
      comment = web back rsync server
      read only = false
      list = yes 是否可以产看此模块信息yes
      auth user = rsyncuser     用户名自己指定，自己定义的非系统用户，即不需要useradd命令来创建的真是centos系统用户
      secrets file = /etc/rsync。passwd      没有此文件，需要自行创建
    ```
    
    - 创建motd
    - 创建/etc/rsync.passwd
      ```
      rsyncuser:password123       前面的用户名用要和上面定义的auth user保持一致
      ```
      - 权限一定是600或者是700 否则读取不到```chmod 600 /etc/rsync.passwd```















