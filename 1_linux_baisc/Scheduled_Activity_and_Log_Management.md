# Linux计划任务与日志管理

1. 计划任务-at-cron-计划任务使用方法
2. 日志的种类和记录的方式-自定义ssh服务日志类型和存储位置
3. 实战-日志切割-搭建远程日志收集服务器
4. 实战-配置公司内网服务器每天定时自动开关机

# 1. 计划任务-at-cron-计划任务使用方法

计划任务的作用是做一些周期性的任务，在生产中的主要用来定期备份数据。CROND这个守护进程是为了周期性执行任务或处理等待事件而存在。 
- 任务调度分两种：系统任务调度，用户任务调度
- 计划任务的安排方式分两种:
  - 一种是定时性的，也就是例行。就是每隔一定的周期就要重复来做这个事情
  - 一种是突发性的，就是这次做完了这个事，就没有下一次了，临时决定，只执行一次的任务
- at和crontab这两个命令：
  - at：它是一个可以处理仅执行一次就结束的指令
  - crontab：它是会把你指定的工作或任务，比如：脚本等，按照你设定的周期一直循环执行下去

## 1.1 at 计划任务的使用
- 语法格式： at  时间
- 服务：atd
#### 1. 查看at服务状态
```
[root@localhost ~]# systemctl status atd
[root@localhost ~]# systemctl start atd
[root@localhost ~]# systemctl is-enabled atd
enabled

[root@localhost ~]# chkconfig --list | grep atd   # centos6 执行此命令，7不能执行
```

#### 2. 创建计划任务
```
[root@localhost at]# date
Fri Mar  1 09:30:38 CST 2019

[root@localhost at]# at 9:35
at> mkdir /abc
at> touch /abc/1.txt<EOT>       # crtl+d 保存退出
job 6 at Fri Mar  1 09:35:00 2019

[root@localhost at]# atq        # 或者at -l
6	Fri Mar  1 09:35:00 2019 a root

[root@localhost at]# at -c 6    # 查看at计划源文件中包含的命令，长篇最后
#!/bin/sh
... ...
mkdir /abc
touch /abc/1.txt
marcinDELIMITER6ed4ae62

[root@localhost at]# tail -5 /var/spool/at/       # 查看at计划源文件中包含的命令，简单方法
a00006018a8adf  .SEQ            spool/          
[root@localhost at]# tail -5 /var/spool/at/a00006018a8adf 
}
${SHELL:-/bin/sh} << 'marcinDELIMITER6ed4ae62'
mkdir /abc
touch /abc/1.txt
marcinDELIMITER6ed4ae62

[root@localhost at]# at -l      # 或者 atq命令
5	Fri Mar  1 09:35:00 2019 a root
[root@localhost at]# atrm 5     # 删除计划任务

```
at计划任务的特殊写法
```
[root@localhost ~]# at 20:00 2018-10-1   在某天 
[root@localhost ~]# at now +10min        在 10分钟后执行
[root@localhost ~]# at 17:00 tomorrow    明天下午5点执行
[root@localhost ~]# at 6:00 pm +3 days   在3天以后的下午6点执行
[root@localhost ~]# at 23:00 < a.txt
```
## 1.2 crontab定时任务的使用
- crond命令定期检查是否有要执行的工作，如果有要执行的工作便会自动执行该工作
- cron是一个linux下的定时执行工具，可以在无需人工干预的情况下运行作业。
- 系统周期性所要执行的工作，如更新whatis数据库, updatedb数据库，日志定期切割，收集系统状态信息，/tmp定期清理
- crontab的参数：
  - ```crontab -u david```       #指定david用户的cron服务
  - ```crontab -l```             #列出当前用户下的cron服务的详细内容
  - ```crontab -u david -l```    #列出指定用户david下的cron服务的详细内容
  - ```crontab -r```             #删除cron服务
  - ```crontab -u david -r```    #root想删除david的cron计划任务
  - ```crontab -e```             #编辑cron服务

- cron -e 中编辑时的语法
  - 格式说明
  ```
  Example of job definition:
  .---------------- minute (0 - 59)
  |  .------------- hour (0 - 23)
  |  |  .---------- day of month (1 - 31)
  |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
  |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
  |  |  |  |  |
  *  *  *  *  * user-name  command to be executed
  ```
  - 参数说明
  
  |符号|说明|示例|
  |-|-|-|
  |* |代表取值范围内的数字| (任意/每)|
  | / | 指定时间的间隔频率 | */10   0-23/2(0-23时每两小时) |
  | -| 代表从某个数字到某个数字 | 8-17  |
  | ，| 分开几个离散的数字 |6,10-13,20|

#### 1. 创建计划任务
- 用户级别的计划任务 /var/spool/cron/
  ```
  [root@localhost ~]# systemctl start crond
  [root@localhost ~]# systemctl enable crond

  [root@localhost ~]# crontab -e    # 进入vim页面编辑命令
  """
  2 10 * * * tar zcvf /opt/grub2.tar.gz /boot/grub2
  """
  no crontab for root - using an empty one
  crontab: installing new crontab
  [root@localhost ~]# crontab -l    # 查看
  2 10 * * * tar zcvf /opt/grub2.tar.gz /boot/grub2

  [root@localhost ~]# ll /var/spool/cron/     # 查看系统中全部的cron计划任务
  total 4
  -rw-------. 1 root root 50 Mar  1 10:01 root
  ```
- 系统级别的计划任务 /etc/crontab 
    也可以直接在/etc/crontab中添加计划任务，与上面使用语法相同。 不建议用户级别的任务全部放到系统级别下执行，但如果都是root身份执行，不好区分的话就都可以。
    
  - 使用crontab命令的注意事项：
    - 环境变量的问题
      - ```SHELL=/bin/bash```                        #指定操作系统使用哪个shell
      - ```PATH=/sbin:/bin:/usr/sbin:/usr/bin```     #系统执行命令的搜索路径
    - 清理邮件日志，比如使用重定向 ```> /dev/null  2>&1```
    
  ```
  [root@localhost ~]# ll /etc/crontab 
  -rw-r--r--. 1 root root 451 Jun 10  2014 /etc/crontab
  
  [root@localhost ~]# vim /etc/crontab 

  SHELL=/bin/bash                        #指定操作系统使用哪个shell
  PATH=/sbin:/bin:/usr/sbin:/usr/bin     #系统执行命令的搜索路径
  MAILTO=root                            #将执行任务的信息通过邮件发送给xx用户

  # For details see man 4 crontabs

  # Example of job definition:
  # .---------------- minute (0 - 59)
  # |  .------------- hour (0 - 23)
  # |  |  .---------- day of month (1 - 31)
  # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
  # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
  # |  |  |  |  |
  # *  *  *  *  * user-name  command to be executed

  ~                                                                                              
  ```

  ```
  [root@localhost ~]# ll /etc/cron    # 2 times tab
  cron.d/       cron.deny     cron.monthly/ cron.weekly/  
  cron.daily/   cron.hourly/  crontab     
  
  [root@localhost ~]# find /etc/cron* -type f
  /etc/cron.d/0hourly
  /etc/cron.d/raid-check
  /etc/cron.d/sysstat
  /etc/cron.daily/logrotate
  /etc/cron.daily/man-db.cron
  /etc/cron.daily/mlocate
  /etc/cron.deny
  /etc/cron.hourly/0anacron
  /etc/crontab
  ```
    - cron.d/       #是系统自动定期需要做的任务，但是又不是按小时，按天，按星期，按月来执行的，那么就放在这个目录下面。
    - cron.deny     #控制用户是否能做计划任务的文件;
    - cron.monthly/  #每月执行的脚本;
    - cron.weekly/   #每周执行的脚本;
    - cron.daily/     #每天执行的脚本;
    - cron.hourly/   #每小时执行的脚本;
    - crontab       #主配置文件 也可添加任务;
  
  - 常见写法：
    - 每天晚上21:00 重启apache
      - ```0 21 * * * /etc/init.d/httpd  restart```
    - 每月1、10、22日的4 : 45重启apache。
      - ```45 4 1,10,22 * *  /etc/init.d/httpd  restart```
    - 每月1到10日的4 : 45重启apache。
      - ```45 4 1-10 * *   /etc/init.d/httpd  restart```
    - 每隔两天的上午8点到11点的第3和第15分钟重启apach
      - ```3,15 8-11 */2 * *  /etc/init.d/httpd  restart```
    - 晚上11点到早上7点之间，每隔一小时重启apach
      - ```0 23-7/1 * * * /etc/init.d/apach restart```
    - 周一到周五每天晚上 21:15 寄一封信给 root@panda:
      - ```15 21 * * 1-5  mail -s "hi" root@panda < /etc/fstab```
    - 互动：crontab不支持每秒。 每2秒执行一次脚本，怎么写？ 在脚本的死循环中，添加命令 sleep 2 ，执行30次自动退出，然后添加，计划任务：```* * * * *  /back.sh``` 


#### 2. 案例

> 每天2：00备份/etc/目录到/tmp/backup下面
> 将备份命令写入一个脚本中
> 每天备份文件名要求格式： 2017-08-19_etc.tar.gz
> 在执行计划任务时，不要输出任务信息
> 存放备份内容的目录要求只保留三天的数据

```
[root@localhost ~]# vim /etc/crontab 

  SHELL=/bin/bash                        # 指定操作系统使用哪个shell
  PATH=/sbin:/bin:/usr/sbin:/usr/bin     # 系统执行命令的搜索路径
  MAILTO=root                            # 将执行任务的信息通过邮件发送给xx用户

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

0 2 * * * root /bin/back.sh   # 系统执行命令的搜索路径，已在上方定义。
                              # 或者直接写绝对路径 /root/backup.sh
~                                            
```
```
===========================================================
# 提前测试需执行命令
mkdir /tmp/backup
tar zcf etc.tar.gz /etc
find /tmp/backup -name “*.tar.gz” -mtime +3 -exec rm -rf {}\;
============================================================
[root@localhost ~]# crontab -l
0 2 * * * /root/backup.sh & > /dev/null   # 后台执行backu.sh并将标准输出放入null，即不输出

[root@localhost ~]# cat backup.sh 
#!/bin/bash
find /tmp/backup -name "*.tar.gz" -mtime +3 -exec rm -f {}\;
#find /tmp/backup -name "*.tar.gz" -mtime +3 -delete
#find /tmp/backup -name "*.tar.gz" -mtime +3 |xargs rm -f
tar zcf /tmp/backup/`date +%F`_etc.tar.gz /etc
```
工作中备份的文件不要放到/tmp,因为过一段时间，系统会清空备/tmp目录。

# 2. 日志的种类和记录的方式-自定义ssh服务日志类型和存储位置

在centos7中，系统日志消息由两个服务负责处理：systemd-journald和rsyslog

## 2.1 常见日志文件的作用
系统日志文件概述：/var/log目录保管由rsyslog维护的，里面存放的一些特定于系统和服务的日志文件

|日志文件|用途|
|-|-|
|/var/log/message|大多数系统日志消息记录在此处。有也例外的：如与身份验证，电子邮件处理相关的定期作业任务等|
|/var/log/secure|安全和身份验证相关的消息和登录失败的日志文件。  ssh远程连接产生的日志|
|/var/log/maillog|与邮件服务器相关的消息日志文件|
|/var/log/cron|与定期执行任务相关的日志文件|
|/var/log/boot.log|与系统启动相关的消息记录|
|/var/log/dmesg|与系统启动相关的消息记录|


#### 1. 使用/var/log/secure文件查看暴力破解系统的ip
```
[root@localhost ~]# grep Failed /var/log/secure
Feb 28 17:19:16 localhost sshd[31061]: Failed password for invalid user ROOT from 192.168.0.219 port 4743 ssh2
Feb 28 17:19:22 localhost sshd[31061]: Failed password for invalid user ROOT from 192.168.0.219 port 4743 ssh2
Mar  1 11:09:29 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2
Mar  1 11:09:33 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2
Mar  1 11:09:36 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2
Mar  1 11:09:40 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2
Mar  1 11:09:44 localhost sshd[42809]: Failed password for xerox from 192.168.0.219 port 6052 ssh2

[root@localhost ~]# grep Failed /var/log/secure | awk '{print $11}' | uniq -c  
      2 ROOT                                    # awk按列（空格区分）查看，打印第十一列，count unique
      5 192.168.0.219
```
#### 2. 使用/var/log/btmp文件查看暴力破解系统的用户

/var/log/btmp文件是记录错误登录系统的日志。如果发现/var/log/btmp日志文件比较大，大于1M，就算大了，就说明很多人在暴力破解ssh服务，此日志需要使用lastb程序查看。

```
[root@localhost ~]# lastb
xerox    ssh:notty    192.168.0.211    Fri Mar  1 11:09 - 11:09  (00:00)    
xerox    ssh:notty    192.168.0.211    Fri Mar  1 11:09 - 11:09  (00:00)    
xerox    ssh:notty    192.168.0.211    Fri Mar  1 11:09 - 11:09  (00:00)    

btmp begins Fri Mar  1 11:09:29 2019
```

发现后，使用防火墙，拒绝掉：命令如下：
```
iptables -A INPUT -i eth0 -s. 192.168.0.211 -j DROP
```
查看恶意ip试图登录次数：
```
lastb | awk  '{ print $3}'  | sort | uniq -c | sort -n
```
清空日志：
- 方法1：```[root@localhost ~]# > /var/log/btmp```
- 方法2：```rm -rf /var/log/btmp  && touch /var/log/btmp```
两者的区别？使用方法2，因为创建了新的文件，而正在运行的服务，还用着原来文件的inode号和文件描述码，所需要重启一下rsyslog服务。建议使用方法1  > /var/log/btmp


#### 3. /var/log/wtmp记录每个用户成功登录次数和持续时间等信息
可以用last命令输出wtmp中内容，last显示到目前为止，成功登录系统的记录

```
[root@localhost ~]# last    # 或者last -f /var/log/wtmp 
xerox    pts/1        192.168.0.219    Fri Mar  1 11:09   still logged in   
root     pts/0        192.168.0.219    Fri Mar  1 09:27   still logged in   
```


## 2.2 日志的记录方式: 分类→ 级别→

- 日志的分类:
  - daemon,  后台进程相关  
  - kern,  	内核产生的信息
  - lpr,   	 打印系统产生的
  - authpriv,  安全认证
  - cron,   	 定时相关
  - mail, 	 邮件相关
  - syslog,  	日志服务本身的
  - news, 	 新闻系统
  - local0~7,  自定义的日志设备，8个系统保留的类，供其它的程序使用或者是用户自定义

- 日志的级别: 0 严重 -> 7 不严重

    |编码|优先级|严重性|
    |-|-|-|
    |7|debug|信息对开发人员调试应用程序有用，在操作过程中无用|
    |6|info|正常的操作信息，可以收集报告，测量吞吐量等|
    |5|notice|注意，正常但重要的事件
    |4|warning|警告，提示如果不采取行动。将会发生错误。比如文件系统使用90%|
    |3|err|错误，阻止某个模块或程序的功能不能正常使用|
    |2|crit|关键的错误，已经影响了整个系统或软件不能正常工作的信息|
    |1|alert|警报,需要立刻修改的信息|
    |0|emerg|紧急，内核崩溃等严重信息|

## 2.3 rsyslog日志服务

- rhel5: 服务名称syslog  -> 配置文件  /etc/syslog.conf
- rhel6-7: 服务名称rsyslog -> 配置文件  /etc/rsyslog.conf

```
[root@localhost ~]# vim /etc/rsyslog.conf 

# rsyslog configuration file

# For more information see /usr/share/doc/rsyslog-*/rsyslog_conf.html
# If you experience problems, see http://www.rsyslog.com/doc/troubleshoot.html

#### MODULES ####

# The imjournal module bellow is now used as a message source instead of imuxsock.
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imjournal # provides access to the systemd journal
#$ModLoad imklog # reads kernel messages (the same are read from journald)
#$ModLoad immark  # provides --MARK-- message capability

# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514

# Provides TCP syslog reception
#$ModLoad imtcp
#$InputTCPServerRun 514


#### GLOBAL DIRECTIVES ####

```

