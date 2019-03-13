# rsync+sersync

# 1. scp - Secure Copy Protocol 安全拷贝协议 

scp无法备份大量数据，类似windows的复制，大文件复制特别慢
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
# 2. rsync

采用C/S模式（客户端/服务器模式）[ 就是一个点到点的传输，直接使用rsync命令 ] 端口873

1. rsync 基于ssh协议的远程传输文件协议
2. rsync 第一次全部传输，之后增量传输，只传输新增和变化修改过的文件。边复制，边统计，边比较
3. rsync 但是如果文件损坏，一样会传输。之前好的文件也会被同步为损坏的文件
4. rsync 镜像保存整个目录树和文件系统， 保持原来文件的权限、时间、软硬。支持匿名传输，以方便进行网站镜象。选择性保持：符号连接，硬链接，文件属性，权限，时间等
5. 常用于定时备份，完整备份，差量备份，增量备份
6. 名词的解释：
    - 发起端：负责发起rsync同步操作的客户机叫做发起端，通知服务器我要备份你的数据
    - 备份源：负责相应来自客户机rsync同步操作的服务器脚在备份源，需要备份的服务器
    - 客户端：存放备份数据
    - 服务端：运行rsyncd服务，一般来说，需要备份的服务器
7. 数据同步方式，两种方案，rsync都有对应的命令来实现
    - 推push：一台主机负责把数据传送给其他主机，服务器开销很大，比较适合后端服务器少的情况。
      - 推：目的主机配置为rsync服务器，源主机周期性的使用rsync命令把要同步的目录推过去（需要备份的机器是客户端，存储备份的机器是服务端）
    - 拉pull：所有主机定时去找一主机拉数据，可能就会导致数据缓慢
      - 拉：源主机配置为rsync服务器，目的主机周期性的使用rsync命令把要同步的目录拉过来（需要备份的机器是服务端，存储备份的机器是客户端）
8. rsync小服务由超级互联网守护进程服务xinetd统一管理
9. rsync 常用参数 -avz， --delete参数使得在2个机器之间完全一致，生产环境下不建议使用
## 2.1 创建系统用户进行备份

本教程使用push的方式进行同步
- 你推送给我，我就是服务端服。需要备份的机器是客户端，存储备份的机器是服务端
- 传给谁，谁安装rsync服务端，但是两边都要安装rsync服务
    
### 1. 安装
```
[root@client163 ~]# yum install xinetd rsync -y

[root@client163 ~]# rsync --daemon                  # 后台运行
[root@client163 ~]# systemctl status rsync          # 并不能在systemctl查到该服务    
Unit rsync.service could not be found.

[root@client163 ~]# netstat -anutp | grep 873        # 使用netstat -nlutp | grep 873 搜索不到结果
tcp        0      0 0.0.0.0:873             0.0.0.0:*               LISTEN      25583/rsync         
tcp6       0      0 :::873                  :::*                    LISTEN      25583/rsync  
```
### 2. 将server162的/var/www/html目录 备份至 client163的/root/web-back。创建测试用户rget1用于下载
- 创建备份用户rget1，client163上面，可以把/web-back的主+组给到新创建的用户 chown
- 创建备份用户rget1，server162上面，将此新用户添加至/var/www/html, setfacl不改变目录原有的用户和属组。此处不能把目录的权限完全chown到rget1，是因为程序运行的时候，有时必须用到root权限
### 3. client162的设置
- 创建传输专用用户rget1
- 创建要备份的路径mkdir -p /var/www/html  
- 给该路径分配rget1用户的扩展权限getfacl setfacl [linux basic command](https://github.com/davidkorea/linux_study/tree/master/1_linux_baisc)
- 创建测试数据至/var/www/html 
```shell
[root@client162 ~]# useradd rget1;echo rget1:11111|chpasswd        # 创建用户

[root@server162 ~]# mkdir -p /var/www/html                         # 创建要备份的路径             
[root@server162 ~]# ll -d /var/www/html/
drwxr-xr-x. 2 root root 6 Nov  5 09:47 /var/www/html/

[root@server162 ~]# getfacl /var/www/html/                         # 查看该目录的文件扩展权限ACL(access control list)
getfacl: Removing leading '/' from absolute path names
# file: var/www/html/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

[root@server162 ~]# setfacl -R -m u:rget1:rwx /var/www/html/       # -R 递归应用子目录文件 -m modify -u user
[root@server162 ~]# getfacl /var/www/html/                         # -R 必须用在 -m 之前，否则没有效果
getfacl: Removing leading '/' from absolute path names
# file: var/www/html/
# owner: root
# group: root
user::rwx
user:rget1:rwx
group::r-x
mask::rwx
other::r-x

[root@server162 ~]# setfacl -R -m d:rget1:rwx /var/www/html/       # -d == -default 默认
[root@server162 ~]# getfacl /var/www/html/
getfacl: Removing leading '/' from absolute path names
# file: var/www/html/
# owner: root
# group: root
user::rwx
user:rget1:rwx
group::r-x
mask::rwx
other::r-x
default:user::rwx
default:user:rget1:rwx
default:group::r-x
default:mask::rwx
default:other::r-x

[root@server162 ~]# cp ./* /var/www/html/           # 创建测试数据
```

### 4. client163的设置
- 创建与server162相同的新用户rget1
- 创建备份数据保存目录
- 将此目录的主组chown变更给新用户rget1
```shell
[root@client163 ~]# useradd rget1;echo rget1:11111|chpasswd

[root@client163 ~]# chown rget1:rget1 -R /web-back/       # 创建于根/ 下的web-back， 不加/，则表示再root/web-back

[root@client163 ~]# ll -d /web-back/
drwxr-xr-x 2 rget1 rget1 6 Mar 13 11:50 /web-back/
```

### 5. rsync备份
server162 推送至 client163
```
##### /root/web-back 会失败#####
# 因为虽然给web-back来rget1 -R 的权限，但是外层文件夹/root 是不允许rget1访问的，因此会报错拒绝
[root@server162 ~]# rsync -avz --delete /var/www/html/ rget1@192.168.0.163:/root/web-back
rget1@192.168.0.163's password: 
sending incremental file list
rsync: failed to set times on "/root/web-back/.": Operation not permitted (1)

##### /web-back 根下面的文件夹 #####
# 再根/下面创建是可以的，因此此文件夹外层无其他文件夹，给到rget1权限后，-R 下面的目录都有访问权限
[root@server162 ~]# rsync -avz --delete /var/www/html/ rget1@192.168.0.163:/web-back/
rget1@192.168.0.163's password: 
sending incremental file list
......
sent 462,188 bytes  received 589 bytes  185,110.80 bytes/sec
total size is 463,820  speedup is 1.00
```

## 2.2 非系统用户备份数据











-----


11. 同步162的/var/www/html 到163的/web-back. 163是服务端。单独创建一个用户进行传输。可以更改用户和属组权限给web-back
12. 162，163两个机器都要创建同样的一个用户，将要备份的目录的权限给到这个用户
13. ```rsync -avz --delete /var/www/html rget1@192.168.0.163:/web-back```，传输html文件夹
    - ```rsync -avz --delete /var/www/html/ rget1@192.168.0.163:/web-back```，今川氏html文件夹下面的文件
    - 但是 生产环境中 不加delete，以免将备份数据清空，或者服务器被黑了，备份数据也跟着被删除

14. 用非系统用户，进行文件传输。对于安全级别比较高的服务器更适合一些。需要更改配置文件/etc/rsyncd.comf。centos7已存在模版，但是6上面没有
    - 163服务端
      ```shell
      uid = root    用root权限
      gid = root
      address = 192.168.0.163     本机ip地址
      port = 873
      host allow = 192.168.0.0/24     允许整个网段
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

      - 创建/etc/rsync.passwd, ```rsyncuser:password123 ``` 前面的用户名用要和上面定义的auth user保持一致
        - 权限一定是600或者是700 否则读取不到```chmod 600 /etc/rsync.passwd```

      - 重启rsync服务，使其读取刚才的配置文件 
        ```
        systemctl start xinetd
        systemctl enable xinetd  开机自启动
        kill -9 pid 干掉之前没有读取配置文件的rsync --daemon进程
        rsync --daemon --config=/etc/rsync.conf  加载配置文件后启动
        ps aux | gerp rsync
        netstat -nlutp | grep 873  可以监听打破局域网内所有在线ip的873端口，因为之前配置文件中配置了192.168.0.0/24host allow
        ```
    - 162 需要备份的服务器
      - ```rsync -avz --delete /var/www/html rsyncuser@192.168.0.163::wwwroot```
        - 用户名是配置文件中的用户，wwwroot为配置文件中的模块名，会自动查找该模块下的path路径
  
      - 自动化命令备份时，密码怎么传输？？
        - 创建与刚才同样的文件/etc/rsync.passwd 输入password123 但是不需要用户名。命令中指定密码文件路径即可
        - ```rsync -avz -delete /var/www/html rsyncuser@192.168.0.163::wwwroot --password-file=/etc/rsync.passwd```
  
-----

# rsync + sersync 实时同步
  
1. sersync 安装在数据源，监控源数据的变化，监控到变化后就可以触发rsync服务，将增删改后到数据传输到备份服务器
2. /var/www/html所在服务器162安装sersync， 163机器不需要更改 
3. sersync基于inotify开发，inotify只能监听到变化，但不知具体哪个文件变化，所以同步都是全量同步。 sersync知道具体哪个文件变化是增量同步
4. 解压后不需要安装，到源文件目录下找到confxml.xml配置文件，改之前 备份一份
    ```
    24 watch="/var/www/html"
    25 remote ip="192.168.0.163" name="wwwroot"  远程服务器到ip，以及rsync到模块名wwwroot
    31 auth start=“true” users=“rsyncuser” passowrdfile=“/etc/rsync.passwd”
    ```
5. 还是刚才源文件目录下到另一个文件sersync2这个文件。启动sersync服务
    ```shell
    ./sersync2 -d -r -o ./confxml.xml

    config xml parse success
    watch path is /var/www/hmtl
    ```
6. 启动服务之前，文件夹内已有到内容不会被监听和同步。只有在服务启动后 新创建到文件，在增删改，才会被坚挺到并同步到备份服务器
7. 只能用来实时同步，不能用来备份。如果备份源被删除了，那么备份源也将被同步删除

8. 应用场景，集群之间数据同步。负载均衡服务器上的内容需要完全一致，才能正常对外提供服务。同步的延迟在毫秒级
9. 同步多个目录，写多个xml文件 www_conxml.xml， bbs_confxml.xml
    - ```/usr/local/sersync/sersync2 -d -o /usr/local/sersync/bbs_confxml.xml```
    - ```/usr/local/sersync/seraync2 -d -o /usr/local/sersync/www_confxml.xml```






  
  
  
