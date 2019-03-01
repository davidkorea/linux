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
  - crontab -u david       #指定david用户的cron服务
  - crontab -l             #列出当前用户下的cron服务的详细内容
  - crontab -u david -l    #列出指定用户david下的cron服务的详细内容
  - crontab -r             #删除cron服务
  - crontab -e             #编辑cron服务































