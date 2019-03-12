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
9. rsync小服务由超级互联网守护进程服务xinetd统一管理
10. rsync 常用参数 -avz， --delete完全一致 在2个机器之间
11. 同步162的/var/www/html 到163的/web-back. 163是服务端。单独创建一个用户进行传输。可以更改用户和属组权限给web-back
12. 162，163两个机器都要创建同样的一个用户，将要备份的目录的权限给到这个用户





















