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
# 2. 正则表达式

正则表达式不只有一种，而且LINUX中不同的程序可能会使用不同的正则表达式，如：grep   sed   awk

- 基础正则表达式

|特别字符|描述|
|-|-|
|$|匹配输入字符串的结尾位置。要匹配 $ 字符本身，请使用 \$|
|( )|标记一个子表达式的开始和结束位置。要匹配这些字符，请使用 \( 和 \)|
|*|匹配前面的子表达式零次或多次。要匹配 * 字符，请使用 \*|
|+|匹配前面的子表达式一次或多次。要匹配 + 字符，请使用 \+|
|.|匹配除换行符 \n 之外的任何单字符。要匹配 . ，请使用 \. |
|[|标记一个中括号表达式的开始。要匹配 [，请使用 \[|
|?|匹配前面的子表达式零次或一次，或指明一个非贪婪限定符。要匹配 ? 字符，请使用 \?|
| \\ |将下一个字符标记为或特殊字符、或原义字符、或向后引用、或八进制转义符。例如， 'n' 匹配字符 'n'。'\n' 匹配换行符。序列 '\\' 匹配 "\"，而 '\(' 则匹配 "("|
|^|匹配输入字符串的开始位置，除非在方括号表达式中使用，此时它表示不接受该字符集合。要匹配 ^ 字符本身，请使用 \^|
|{|标记限定符表达式的开始。要匹配 {，请使用 \{|
| \| |指明两项之间的一个选择要匹配，请使用 \|   如：  Y \| y |

|定位符|描述|
|-|-|
|^|匹配输入字符串开始的位置|
|$|匹配输入字符串结尾的位置|

|非打印字符||描述|
|-|-|
|\n|匹配一个换行符|
|\r|匹配一个回车符|
|\t|匹配一个制表符|

