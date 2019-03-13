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
- ```rsync -avz --delete /var/www/html rget1@192.168.0.163:/web-back```，传输html文件夹
- ```rsync -avz --delete /var/www/html/ rget1@192.168.0.163:/web-back```，传输html文件夹下面的文件
但是 生产环境中 不加delete，以免将备份数据清空，或者服务器被黑了，备份数据也跟着被删除

## 2.2 非系统用户备份数据

用非系统用户，进行文件传输。对于安全级别比较高的服务器更适合一些。需要更改配置文件/etc/rsyncd.comf，centos7已存在模版，但是6上面没有

### 1. client163
因为server162推送给client163，所以client163为服务端
- 更改配置文件
```shell
[root@client163 ~]# vim /etc/rsyncd.conf 

uid = root                          # 用root权限
gid = root
address = 192.168.0.163             # 本机ip地址
port = 873
hosts allow = 192.168.0.0/24         # 允许整个网段
use chroot = yes
max connections = 4                 # 最大链接数
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsyncd.lock
log file = /var/log/rsyncd.log
motd file = /etc/rsync.motd         # 没有此文件，需要自行创建

[wwwroot]                           # 模块名，自己定义一个模块，名称自己定
path = /web-back/
comment = web back rsync server
read only = false
list = yes                          # 是否可以产看此模块信息yes
auth users = rsyncuser               # 用户名自己指定，自己定义的非系统用户，
                                    # 即不需要useradd命令来创建的真是centos系统用户
secrets file = /etc/rsync.passwd    # 没有此文件，需要自行创建
```    

配置文件分为两部分：全局参数，模块参数

- 全局参数：对rsync服务器生效，如果模块参数和全局参数冲突，冲突的地方模块参数生效
- 模块参数：定义需要通过rsync输出的目录定义的参数

> 常见的全局参数：
> - ```port```          #→指定后台程序使用的端口号，默认为873。
> - ```uid```            #→该选项指定当该模块传输文件时守护进程应该具有的uid，配合gid选项使用可以确定哪些可以访问怎么样的文件权限，默认值是" nobody"。
> - ```gid```            #→该选项指定当该模块传输文件时守护进程应该具有的gid。默认值为" nobody"。
> - ```max connections```        #→指定该模块的最大并发连接数量以保护服务器，超过限制的连接请求将被告知随后再试。默认值是0，也就是没有限制。
> - ```lock file```                  #→指定支持max connections参数的锁文件，默认值是/var/run/rsyncd.lock。
> - ```motd file```         #→" motd file"参数用来指定一个消息文件，当客户连接服务器时该文件的内容显示给客户，默认
> 是没有motd文件的。
> - ```log file```             #→" log file"指定rsync的日志文件，而不将日志发送给syslog。
> - ```pid file```              #→指定rsync的pid文件，通常指定为“/var/run/rsyncd.pid”，存放进程ID的文件位置。
> - ```hosts allow``` =    #→单个IP地址或网络地址   //允许访问的客户机地址
> 
> 常见的模块参数：主要是定义服务器哪个要被同步输出，其格式必须为“ [ 共享模块名 ]” 形式，这个名字就是在 rsync 客户端看到的名字，其实很像 Samba 服务器提供的共享名。而服务器真正同步的数据是通过 path 来指定的。
> - ```Comment```         #→给模块指定一个描述，该描述连同模块名在客户连接得到模块列表时显示给客户。默认没有描述定义。
> - ```Path```                  #→指定该模块的供备份的目录树路径，该参数是必须指定的。
> - ```read only```         #→yes为只允许下载，no为可以下载和上传文件到服务器
> - ```exclude```             #→用来指定多个由空格隔开的多个文件或目录(相对路径)，将其添加到exclude列表中。这等同于在客户端命令中使用―exclude或----filter来指定某些文件或目录不下载或上传(既不可访问)
> - ```exclude from```   #→指定一个包含exclude模式的定义的文件名，服务器从该文件中读取exclude列表定义，每个文件或目录需要占用一行
> - ```include```             #→用来指定不排除符合要求的文件或目录。这等同于在客户端命令中使用--include来指定模式，结合include和exclude可以定义复杂的exclude/include规则。
> - ```include from```   #→指定一个包含include模式的定义的文件名，服务器从该文件中读取include列表定义。
> - ```auth users```       #→该选项指定由空格或逗号分隔的用户名列表，只有这些用户才允许连接该模块。这里的用户和系统用户没有任何关系。如果" auth users"被设置，那么客户端发出对该模块的连接请求以后会被rsync请求challenged进行验证身份这里使用的challenge/response认证协议。用户的名和密码以明文方式存放在" secrets file"选项指定的文件中。默认情况下无需密码就可以连接模块(也就是匿名方式)。
> - ```secrets file```      #→该选项指定一个包含定义用户名:密码对的文件。只有在" auth users"被定义时，该文件才有作用。文件每行包含一个username:passwd对。一般来说密码最好不要超过8个字符。没有默认的secures file名，注意：该文件的权限一定要是600，否则客户端将不能连接服务器。
> - ```hosts allow ```     #→指定哪些IP的客户允许连接该模块。定义可以是以下形式：
>     单个IP地址，例如：192.167.0.1，多个IP或网段需要用空格隔开，
>     整个网段，例如：192.168.0.0/24，也可以是192.168.0.0/255.255.255.0
> “*”则表示所有，默认是允许所有主机连接。
> - ```hosts deny```      #→指定不允许连接rsync服务器的机器，可以使用hosts allow的定义方式来进行定义。默认是没有hosts deny定义。
> - ```list```              #→该选项设定当客户请求可以使用的模块列表时，该模块是否应该被列出。如果设置该选项为false，
> 可以创建隐藏的模块。默认值是true。
> - ```timeout```   #→通过该选项可以覆盖客户指定的IP超时时间。通过该选项可以确保rsync服务器不会永远等待一个崩溃的客户端。超时单位为秒钟，0表示没有超时定义，这也是默认值。对于匿名rsync服务器来说，一个理想的数字是600。

- 创建motd文件和rsync.passwd文件
```shell
[root@localhost ~]# echo "welcome to back server" ? /etc/rsyncd.motd

[root@localhost ~]# vim /etc/rsync.passwd
rsyncuser:11111         # 密码11111，用户名为rsyncuser 在配置文件中的auth users

[root@localhost ~]# chmod 600 /etc/rsync.passwd         # 目录权限必须是700或者600，否则的话身份验证会失效，设置rsync user的时候
```
- 重启rsync服务，使其读取刚才的配置文件 
```
systemctl start xinetd
systemctl enable xinetd             # 开机自启动
ps aux | gerp rsync                 # 查看当前rsync的pid
kill -9 pid                         # 干掉之前没有读取配置文件的rsync --daemon进程
rsync --daemon --config=/etc/rsync.conf         # 加载配置文件后，重新启动rsync
netstat -nlutp | grep 873       # 可以监听打破局域网内所有在线ip的873端口，因为之前配置文件中配置了192.168.0.0/24host allow
```
### 2. server162/server100
- 手动输入密码
```shell
[root@server100 ~]# rsync -avz --delete /var/www/html/ rsyncuser@192.168.0.12::wwwroot
welcome to back server              # 用户名是配置文件中的auth user，wwwroot为配置文件中的模块名，会自动查找该模块下的path路径

Password: 
sending incremental file list
```
- 配置密码文件，以便自动备份
```shell
[root@server100 ~]# vim /etc/rsync.passwd
    11111
    
[root@server100 ~]# chmod 600 /etc/rsync.passwd 
[root@server100 ~]# rsync -avz --delete /var/www/html/ rsyncuser@192.168.0.12::wwwroot --password-file=/etc/rsync.passwd
welcome to back server

sending incremental file list
./
11 22 33

sent 466 bytes  received 38 bytes  1,008.00 bytes/sec
total size is 6,087  speedup is 12.08

```
### 3. 脚本实现定时自动备份
自动化命令备份时，密码怎么传输？？
- 创建与刚才同样的文件/etc/rsync.passwd 输入密码11111，但是不需要用户名，命令中指定密码文件路径即可
- ```rsync -avz -delete /var/www/html rsyncuser@192.168.0.163::wwwroot --password-file=/etc/rsync.passwd```

```shell
[root@xuegod63 ~]# vim autobackup.sh
#!/bin/bash
rsync -avz --delete  /var/www/html rsyncuser@192.168.0.64::wwwroot --password-file=/opt/passfile 
[root@xuegod63 ~]# chmod +x autobackup.sh
[root@XueGod64 ~]# rm -rf /web-back/*                       //测试脚本
[root@xuegod63~]# sh autobackup.sh
[root@XueGod64 ~]# echo "01 3 * * * sh /root/autoback.sh &" >> /var/spool/cron/root
```


# 3. rsync + sersync 实时同步

下载sersync：  https://code.google.com/archive/p/sersync/downloads
  
1. sersync 安装在数据源，监控源数据的变化，监控到变化后就可以触发rsync服务，将增删改后到数据传输到备份服务器
    1. 用户实时的往sersync服务器上写入更新文件数据；
    2. 此时需要在同步主服务器上配置sersync服务；
    3. 在另一台服务器开启rsync守护进程服务，以同步拉取来自sersync服务器上的数据；通过rsync的守护进程服务后可以发现，实际上sersync就是监控本地的数据写入或更新事件；然后，在调用rsync客户端的命令，将写入或更新事件对应的文件通过rsync推送到目标服务器

2. /var/www/html所在服务器server162安装sersync， client163机器不需要更改 
3. sersync基于inotify开发，inotify只能监听到变化，但不知具体哪个文件变化，所以同步都是全量同步。 sersync知道具体哪个文件变化是增量同步
4. 解压后不需要安装，文件夹改个名称
    ```
    [root@server100 ~]# mv GNU-Linux-x86/ sersync
    ```
5. 到源文件目录下找到confxml.xml配置文件，改之前 备份一份

    ```shell
    24 watch="/var/www/html"
    25 remote ip="192.168.0.163" name="wwwroot"         # 远程服务器到ip，以及rsync到模块名wwwroot
    31 auth start=“true” users=“rsyncuser” passowrdfile=“/etc/rsync.passwd”
    ```
5. 还是刚才源文件目录下到另一个文件sersync2这个文件。启动sersync服务
    ```shell
    ./sersync2 -d -r -o ./confxml.xml

    config xml parse success
    watch path is /var/www/hmtl
    ```
    
    ```
    [root@server100 sersync]# ./sersync2 -d -r -o ./confxml.xml
    set the system param
    execute：echo 50000000 > /proc/sys/fs/inotify/max_user_watches
    execute：echo 327679 > /proc/sys/fs/inotify/max_queued_events
    parse the command param
    option: -d 	run as a daemon
    option: -r 	rsync all the local files to the remote servers before the sersync work
    option: -o 	config xml name：  ./confxml.xml
    daemon thread num: 10
    parse xml config file
    host ip : localhost	host port: 8008
    daemon start，sersync run behind the console 
    use rsync password-file :
    user is	rsyncuser
    passwordfile is 	/etc/rsync.passwd
    config xml parse success
    please set /etc/rsyncd.conf max connections=0 Manually
    sersync working thread 12  = 1(primary thread) + 1(fail retry thread) + 10(daemon sub threads) 
    Max threads numbers is: 22 = 12(Thread pool nums) + 10(Sub threads)
    please according your cpu ，use -n param to adjust the cpu rate
    ------------------------------------------
    rsync the directory recursivly to the remote servers once
    working please wait...
    execute command: cd /var/www/html && rsync -artuz -R --delete ./ rsyncuser@192.168.0.12::wwwroot --password-file=/etc/rsync.passwd >/dev/null 2>&1 
    run the sersync: 
    watch path is: /var/www/html
    ```
6. 启动服务之前，文件夹内已有到内容不会被监听和同步。只有在服务启动后 新创建到文件，在增删改，才会被坚挺到并同步到备份服务器
    ```
    [root@client12 web-back]# watch ls -l       # 实时监控目录变化
    ```
7. 只能用来实时同步，不能用来备份。如果备份源被删除了，那么备份源也将被同步删除

8. 应用场景，集群之间数据同步。负载均衡服务器上的内容需要完全一致，才能正常对外提供服务。同步的延迟在毫秒级
9. 多实例情况, 同步多个目录，写多个xml文件 www_conxml.xml， bbs_confxml.xml
    - ```/usr/local/sersync/sersync2 -d -o /usr/local/sersync/bbs_confxml.xml```
    - ```/usr/local/sersync/seraync2 -d -o /usr/local/sersync/www_confxml.xml```

10. 设置sersync监控开机自动执行
    ```shell
    vim /etc/rc.d/rc.local  #编辑，在最后添加一行
    /usr/local/sersync/sersync2 -d -r -o  /usr/local/sersync/confxml.xml  ＃设置开机自动运行脚本
    ```
11. 添加脚本监控sersync是否正常运行,
    ```shell
    vim  /opt/check_sersync.sh  #编辑，添加以下代码
    #!/bin/sh
    sersync="/opt /sersync/sersync2"
    confxml="/opt /sersync/confxml.xml"
    status=$(ps aux |grep 'sersync2'|grep -v 'grep'|wc -l)
    if [ $status -eq 0 ];
    then
    $sersync -d -r -o $confxml &
    else
    exit 0;
    fi
    ```
    ```shell
    chmod +x /opt /check_sersync.sh  #添加脚本执行权限
    ```
    把这个脚本加到任务计划,定期执行检测
    





  
  
  
