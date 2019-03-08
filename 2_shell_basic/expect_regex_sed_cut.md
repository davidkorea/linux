# expect-正则表达式-sed-cut的使用

1. expect实现无交互登录
2. 正则表达式
3. sed流编辑器
4. cut命令
5. 实战-bash脚本语法检查和查看详细的执行过程

# 1. expect实现无交互登录
如果你想要写一个能够自动处理输入输出的脚本（如向用户提问并且验证密码）又不想面对C或者Perl，那么expect是你的最好的选择。它可以用来做一些linux下无法做到交互的一些命令操作




- 免密码通过SSH登录服务器
  ```
  [root@localhost ~]# vim ssh.exp
    #!/usr/bin/expect
    set ipaddr "192.168.0.162"
    set name "root"
    set passwd "11111"
    set timeout 30
    spawn ssh $name@$ipaddr
    expect {
    "yes/no" { send "yes\r";exp_continue }
    "password" { send "$passwd\r" }
    }
    expect "#"
    send "touch /root/xuegod1011.txt\r"
    send "ls /etc > /root/xuegod1011.txt\r"
    send "mkdir /tmp/xuegod1011\r"
    send "exit\r"
    expect eof

  [root@localhost ~]# expect ssh.exp 
  spawn ssh root@192.168.0.162
  root@192.168.0.162's password: 
  Last login: Fri Mar  8 10:19:13 2019 from 192.168.0.219
  [root@localhost ~]# touch /root/xuegod1011.txt
  [root@localhost ~]# ls /etc > /root/xuegod1011.txt
  [root@localhost ~]# mkdir /tmp/xuegod1011
  [root@localhost ~]# exit
  logout
  Connection to 192.168.0.162 closed.
  ```
- 对服务器批量管理（了解一下）
  ```
  [root@xuegod63 ~]# cat ip_pass.txt    #这里写上要执行的IP地址和root用户密码
  192.168.1.63  123456
  192.168.1.63  123456
  192.168.1.63  123456
  
  [root@xuegod63 ~]# cat ssh2.exp   #编写要执行的操作
  #!/usr/bin/expect
  set ipaddr [lindex $argv 0]
  set passwd [lindex $argv 1]
  set timeout 30
  spawn ssh root@$ipaddr
  expect {
  "yes/no" { send "yes\r";exp_continue }
  "password" { send "$passwd\r" }
  }
  expect "#"
  send "touch /root/xuegod1011.txt\r"
  send "ls /etc > /root/xuegod1011.txt\r"
  send "mkdir /tmp/xuegod1011\r"
  send "exit\r"
  expect eof

  [root@xuegod63 ~]# cat login.sh    #开始执行
  #!/bin/bash
  echo    # 空行
  for ip in `awk '{print $1}' /root/ip_pass.txt`
  do
    pass=`grep $ip /root/ip_pass.txt|awk '{print $2}'`
    expect /root/ssh.exp $ip $pass
  done
  ```
