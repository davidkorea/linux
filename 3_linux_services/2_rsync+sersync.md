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
3. rsync 增量传输，只传输新增和变化修改过的文件。边复制，边统计，边比较
