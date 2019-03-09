# expect-正则表达式-sed-cut的使用

1. expect实现无交互登录
2. 正则表达式
3. sed流编辑器
4. cut命令
5. 实战-bash脚本语法检查和查看详细的执行过程

# 1. expect实现无交互登录
如果你想要写一个能够自动处理输入输出的脚本（如向用户提问并且验证密码）又不想面对C或者Perl，那么expect是你的最好的选择。它可以用来做一些linux下无法做到交互的一些命令操作

- 使用expect创建脚本的方法
  1. 定义脚本执行的shell, ```#!/usr/bin/expect ```, 这里定义的是expect可执行文件的链接路径（或真实路径），功能类似于bash等shell功能
  2. ```set timeout 30```	设置超时时间，单位是秒，如果设为timeout -1 意为永不超时
  3. spawn 是进入expect环境后才能执行的内部命令，如果没有装expect或者直接在默认的SHELL下执行是找不到spawn命令的。不能直接在默认的shell环境中进行执行主要功能，它主要的功能是给ssh运行进程加个壳，用来传递交互指令。
  4. expect	这里的expect同样是expect的内部命令
	  - 主要功能：判断输出结果是否包含某项字符串，没有则立即返回，否则就等待一段时间后返回，等待时间通过timeout进行设置
  5. send	执行交互动作，将交互要执行的动作进行输入给交互指令。命令字符串结尾要加上"\r"，如果出现异常等待的状态可以进行核查
  6. exp_continue	继续执行接下来的交互操作
  7. interact	执行完后保持交互状态，把控制权交给控制台；如果不加这一项，交互完成会自动退出
  8. $argv, expect 脚本可以接受从bash传递过来的参数，可以使用 [lindex $argv n]获得，n从0开始，分别表示第一个，第二个，第三个……参数


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
**一组 组合命令之间 不加空格**

|特别字符|描述|
|-|-|
|$|匹配输入字符串的结尾位置。要匹配 $ 字符本身，请使用 \\$|
|( )|标记一个子表达式的开始和结束位置。要匹配这些字符，请使用 \\( 和 \\)|
|*|匹配前面的子表达式零次或多次。要匹配 * 字符，请使用 \\*|
|+|匹配前面的子表达式一次或多次。要匹配 + 字符，请使用 \\+|
|.|匹配除换行符 \n 之外的任何单字符。要匹配 . ，请使用 \\. |
|[|标记一个中括号表达式的开始。要匹配 [，请使用 \\[|
|?|匹配前面的子表达式零次或一次，或指明一个非贪婪限定符。要匹配 ? 字符，请使用 \\?|
| \\ |将下一个字符标记为或特殊字符、或原义字符、或向后引用、或八进制转义符。例如， 'n' 匹配字符 'n'。'\n' 匹配换行符。序列 '\\ \\' 匹配 "\\"，而 '\\(' 则匹配 "("|
|^|匹配输入字符串的开始位置，除非在方括号表达式中使用，此时它表示不接受该字符集合。要匹配 ^ 字符本身，请使用 \\^|
|{|标记限定符表达式的开始。要匹配 {，请使用 \\ { |
| \| |指明两项之间的一个选择要匹配，要匹配 \|，请使用 \\ \|  如：  Y \| y |

|定位符|描述|
|-|-|
|^|匹配输入字符串开始的位置|
|$|匹配输入字符串结尾的位置|

|非打印字符|描述|
|-|-|
|\n|匹配一个换行符|
|\r|匹配一个回车符|
|\t|匹配一个制表符|

- 统计/etc/ssh/sshd_config文件中除去空行和#号开头的行的行数
  - ```[root@localhost ~]# grep -v  "^$\|^#" /etc/ssh/sshd_config ```, 整个正则表达式中不加空格，使用转义或者符号
  - ```[root@localhost ~]# grep -E -v "^$|^#" /etc/ssh/sshd_config ```, -E, 可以不转义或者符号|
  - ```[root@localhost ~]# egrep -v "^$|^#" /etc/ssh/sshd_config ```，egrep = grep -E

- 查找passwd文件包括.ot 的字符
  ```shell
  [root@localhost ~]# grep .ot /etc/passwd
  root:x:0:0:root:/root:/bin/bash
  operator:x:11:0:operator:/root:/sbin/nologin
  setroubleshoot:x:990:984::/var/lib/setroubleshoot:/sbin/nologin
  ```
# 3. sed流编辑器 sed strem editor 

sed编辑器是一行一行的处理文件内容的。正在处理的内容存放在模式空间(缓冲区)内，处理完成后按照选项的规定进行输出或文件的修改。

接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件,简化对文件的反复操作。

sed的执行过程：
- 一次读取一行数据
- 根据我们提供的规则来匹配相关的数据，比如查找root。
- 按照命令修改数据流中的数据，比如替换
- 将结果进行输出
- 重复上面四步

语法格式：```sed  [options] ‘[commands]’ filename```
```
[root@localhost david]# echo "i am a cat" | sed 's/cat/dog/'
i am a dog
```
- sed选项|参数options:
	- a	在当前行下面插入文件
	- n	读取下一个输入行，用下一个命令处理新的行而不是用第一个命令
	- e	执行多个sed指令
	- f	运行脚本
	- i	编辑文件内容 ***
	- i.bak	编辑的同时创造.bak的备份
	- r	使用扩展的正则表达式 
- 命令：
	- i	在当前行上面插入文件
	- c	把选定的行改为新的指定的文本
	- p	打印 ***
	- d	删除 ***
	- r/R	读取文件/一行
	- w	另存
	- s	查找
- sed匹配字符集
	- ^ 匹配行开始，如：/^sed/匹配所有以sed开头的行。
	- $ 匹配行结束，如：/sed$/匹配所有以sed结尾的行。
	- . 匹配一个非换行符的任意字符，如：/s.d/匹配s后接一个任意字符，最后是d。
	- \* 匹配0个或多个字符，如：/*sed/匹配所有模板是一个或多个空格后紧跟sed的行。

- s 替换/etc/passwd
	```
	[root@localhost david]# sed 's/root/hello/' /etc/passwd
	hello:x:0:0:root:/root:/bin/bash
	
	[root@localhost david]# sed 's/root/hello/g' /etc/passwd	# g参数 全部替换一整行
	hello:x:0:0:hello:/hello:/bin/bash
	```
- 按行替换
	1. 用数字表示行范围；$表示行尾
	2. 用文本模式配置来过滤	

	- 单行替换
	```
	[root@localhost david]# sed '2s/bin/hello/' /etc/passwd | more
	root:x:0:0:root:/root:/bin/bash
	hello:x:1:1:bin:/bin:/sbin/nologin
	```
	- 多行替换，如果涉及到多行处理，用逗号表示行间隔
	```
	[root@localhost david]# sed '2,$s/bin/okok/' /etc/passwd | more #从第二行一直到文档结尾
	root:x:0:0:root:/root:/bin/bash
	okok:x:1:1:bin:/bin:/sbin/nologin
	daemon:x:2:2:daemon:/sokok:/sbin/nologin
	adm:x:3:4:adm:/var/adm:/sokok/nologin
	```
	
- d  删除第2行到第4行的内容
	```
	[root@localhost david]# sed '2,4d' /etc/hosts
	127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
	[root@localhost david]#
	```
- 添加行
	1. 命令i(insert插入)，在当前行前面插入一行  i\
	2. 命令a(append附加)，在当前行后面添加一行 a\
	
	```
	[root@localhost david]# echo "hello "| sed 'i\ world'
	 world
	hello 
	```
	- 在文件最后追加内容 $a
		```
		[root@localhost david]# cat a.txt 
		kjhkjh
		[root@localhost david]# sed '$a\ hello world' a.txt 
		kjhkjh
		 hello world

		```
