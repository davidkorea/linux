> ########################################### 
> 
> [Issue: 所有命令无法使用，报错command not found...](https://github.com/davidkorea/linux_study/blob/master/2_shell_basic/README.md#issue)
> 
> ###########################################       

1. if [ ...] ; then
2. $(( $i * $j ))

# shell脚本基础
1. shell基本语法
2. shell变量及运用
3. 数学运算
4. 实战-升级系统中的java版本到1.8版本-为后期安装Hadoop集群做准备

# 1. shell基本语法
- Shell是一个命令解释器，它在操作系统的最外层，负责直接与用户进行对话，把用户的输入解释给操作系统，并处理各种各样的操作系统的输出结果，输出到屏幕反馈给用户。这种对话方式可是交互也可以是非交互式的
  ![shell.png](https://i.loli.net/2019/03/05/5c7de3e1b7e32.png)
  
- shell脚本的执行通常有以下几种方式
  1. ```/root/test.sh```  或者 ```./test.sh``` (当前路径下执行脚本的话要有执行权限chmod +x test.sh) 
  2. ```bash test.sh``` 或 sh test.sh```  （这种方式可以不对脚本文件添加执行权限）
  3. ```source test.sh``` (可以没有执行权限)
  4. ```sh < test.sh``` 或者 ```cat test.sh |sh(bash```)

# 2. shell变量及运用

- 变量的分类

  - 按照变量的作用可以分成4类：
    1. 用户自定义变量 
    2. 环境变量：这种变量中主要保存的是和系统操作环境相关的数据
    3. 位置参数变量：这种变量主要是用来向脚本当中传递参数或数据的，变量名不能自定义，变量作用是固定的
    4. 预定义变量：是Bash中已经定义好的变量，变量名不能自定义，变量作用也是固定的

  - 按照变量作用域可以分成2类：全局变量和局部变量

    - 局部变量是shell 程序内部定义的，其使用范围仅限于定义它的程序，对其它程序不可见。包括：用户自定义变量、位置变量和预定义变量。
    - 全局变量是环境变量，其值不随shell 脚本的执行结束而消失。
### 2.1 用户自定义变量 

- 变量的赋值, 不允许数字开头，等号两边不能有空格
  ```
  [root@localhost ~]# VAR1=123
  [root@localhost ~]# echo $VAR1 
  123
  [root@localhost ~]# VAR2 = 222
  bash: VAR2: command not found...
  ```
- 变量值的叠加，使用${}
  ```
  [root@localhost ~]# VAR2=mysql
  [root@localhost ~]# echo $VAR2
  mysql
  [root@localhost ~]# echo $VAR2_db     # VAR2_db 被识别为一个变量，找不到，为空

  [root@localhost ~]# echo $VAR2.db
  mysql.db
  [root@localhost ~]# echo ${VAR2}_db   # ${VAR2}_db
  mysql_db
  ```
- 命令的替换/调用, 使用$()或反引号``` `...` ```
  ```
  [root@localhost ~]# date
  Tue Mar  5 11:14:36 CST 2019
  [root@localhost ~]# echo `date`
  Tue Mar 5 11:14:43 CST 2019
  [root@localhost ~]# echo $(date)
  Tue Mar 5 11:14:52 CST 2019

  [root@localhost ~]# echo `date +“%Y-%m”`
  “2019-03”
  ```
  1. date的使用
    ```
    [root@localhost ~]# echo `date +“%Y-%m”`  
      “2019-03”
    [root@localhost ~]# date +"%Y-%m-%d %H-%M-%S", 分隔符可以使用```/```,```-```,```:```  
      2019-03-05 11-17-32
    [root@localhost ~]# date +"%Y-%m-%d %h-%m-%s"  
      2019-03-05 Mar-03-1551755869
    [root@localhost ~]# date +"%H-%M-%S"  
      11-18-09
    ```
  2. date设定日期
    ```
    date -s 20180523               #设置成20120523，这样会把具体时间设置成空00:00:00
    date -s 01:01:01               #设置具体时间，不会对日期做更改
    date -s "2018-05-23 01:01:01"  #这样可以设置全部时间
    ```
  3. date命令加减操作：
    ```
    date +%Y%m%d                   #显示当天年月日
    date -d "+1 day" +%Y%m%d       #显示明天的日期
    date -d "-1 day" +%Y%m%d       #显示昨天的日期
    date -d "-1 month" +%Y%m%d     #显示上一月的日期
    date -d "+1 month" +%Y%m%d     #显示下一月的日期
    date -d "-1 year" +%Y%m%d      #显示前一年的日期
    date -d "+1 year" +%Y%m%d      #显示下一年的日期
    ```
- 命令的嵌套使用，使用$( $( ))
  ```
  [root@localhost ~]# find ./-name *.txt
  find: './-name': No such file or directory
  1.txt
  2.txt
  3.txt
  4.txt
  5.txt

  [root@localhost ~]# VAR3=$(tar zcvf txtgz.tar.gz $(find ./-name *.txt))
  find: './-name': No such file or directory
  
  [root@localhost ~]# ls
  1.txt  2.txt  3.txt  4.txt  5.txt  txtgz.tar.gz
  ```
- 单引号和双引号区别
  1. ```‘’```,	在单引号中所有的字符包括特殊字符（$,'',`和\）都将解释成字符本身而成为普通字符。
  2. ```“”```,	在双引号中，除了$, '', `和\以外所有的字符都解释成字符本身，拥有“调用变量的值”、“引用命令”和“转义符”的特殊含义
  3. 单引号之间的内容原封不动赋值给变量，双引号之间的内容如有特殊符号会保留它的特殊含义
  4. \转义符，跟在\之后的特殊符号将失去特殊含义，变为普通字符。如\$将输出“$”符号，而不当做是变量引用
  
  ```
  [root@localhost ~]# VAR4='hello $VAR1'
  [root@localhost ~]# echo $VAR4
  hello $VAR1
  
  [root@localhost ~]# VAR5="hello $VAR1"
  [root@localhost ~]# echo $VAR5
  hello 123
  ```

- 删除变量
  ```
  [root@localhost ~]# unset VAR1
  [root@localhost ~]# echo $VAR1

  ```

### 2.2 环境变量
在bash shell中，环境变量分为两类：全局变量和局部变量

全局变量：对于shell会话和所有的子shell都是可见的

局部变量：它只在自己的进程当中使用

- 局部变量
  ```
  [root@localhost ~]# VAR1=123
  [root@localhost ~]# echo $VAR1
  123
  
  [root@localhost ~]# vim var1.sh
    #!/bin/bash
    echo $VAR1
  [root@localhost ~]# bash var1.sh    # 执行var1.sh 时，会使用另一个bash去执行，就访问不到$VAR1的值

  ```

- 全局变量
  1. 查看全局变量
    ```
    [root@localhost ~]# env         # 查看所有全局变量

    [root@localhost ~]# env | grep PATH
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
    ```
  2. 使用export把局部变量 -> 全局变量
    ```
    [root@localhost ~]# export VAR1=david
    [root@localhost ~]# echo $VAR1
    david
    
    [root@localhost ~]# vim var1.sh 
    [root@localhost ~]# bash var1.sh 
    david
    ```
  3. 全局变量->永久生效
  
    - 可以把定义好的变量写入配置文件, 当登录系统或新开启一个ssh连接启动bash进程时，一定会加载这4个配置文件
      -  ```/etc/profile```
      -  ```/etc/bashrc```
      -  ```/root/.bashrc```
      -  ```/root/.bash_profile```
      > 互动：如何知道新建一个ssh连接，加载这4个配置文件先后顺序？ 答：可以每个文件的最后，追加一个echo命令，输出一下文件的名字
      >  ```
      >  [root@localhost ~]# echo 'echo  /etc/profile ' >> /etc/profile
      >  [root@localhost ~]# echo 'echo  /etc/bashrc' >> /etc/bashrc
      >  [root@localhost ~]# echo 'echo  /root/.bashrc ' >> /root/.bashrc
      >  [root@localhost ~]# echo 'echo  /root/.bash_profile ' >> /root/.bash_profile
      >  [root@localhost ~]# ssh root@192.168.0.162   #弹出以下信息，就知道有优先级了
      >  /etc/profile
      >  /etc/bashrc
      >  /root/.bashrc
      >  /root/.bash_profile
      >  ```
      > 可以在这里添加木马程序，只要管理登录系统，就触发木马程序
      
    - 新开的xshell连接中，还是读不到变量VAR1
      ![xshell.png](https://i.loli.net/2019/03/05/5c7e03a9b1f53.png)

    
    ```
    [root@localhost ~]# vim /etc/profile        #在文件的最后插入
    export VAR1=david                           #=等号两边不能有空格

    [root@localhost ~]# source  /etc/profile    #重新加载profile文件
    
    # open a new xshell and bash var1.sh success
    ```
### 2.3 PATH环境变量    
  shell要执行某一个程序，它要在系统中去搜索这个程序的路径，path变量是用来定义命令和查找命令的目录，当我们安装了第三方程序后，可以把第三方程序bin目录添加到这个path路径内，就可以在全局调用这个第三方程序的    
- 创建一个backup 脚本
```
[root@localhost ~]# vim /opt/backup
[root@localhost ~]# chmod +x /opt/backup 
[root@localhost ~]# /opt/backup 
backup data is ok!
[root@localhost ~]# backup
bash: backup: command not found...
```
- 将backup命令添加PATH中
```
[root@localhost ~]# PATH=/opt/:%PATH
[root@localhost ~]# backup 
backup data is ok!

[root@localhost ~]# vim /etc/profile  # 在文件最后追加以下内容，永久生效
export PATH=/opt/:$PATH   

[root@localhost ~]# source /etc/profile
```
### 2.4 shell位置变量
    
- $0  获取当前执行shell脚本的文件文件名，包括脚本路径,命令本身
- $n  获取当前脚本的第n个参数 n=1,2.....n 当n大于9时 用${10}表示

```
[root@localhost ~]# vim print.sh
  #!/bin/bash
  echo "shell 's name: $0"
  echo "first param: $!"
  echo "second param: $2"
  echo "thrid param: $3"

[root@localhost ~]# chmod +x print.sh 
[root@localhost ~]# ./print.sh 11 22 33
shell 's name: ./print.sh
first param: 
second param: 22
thrid param: 33
```

### 2.5 特殊变量

有些变量是一开始执行Script脚本时就会设定，且不能被修改，但我们不叫它只读的系统变量，而叫它特殊变量。这些变量当一执行程序时就有了，以下是一些特殊变量：

|paramater|describe|
|-|-|
|$* |以一个单字符串显示所有向脚本传递的参数；如"$*"用【"】括起来的情况、以"$1 $2 … $n"的形式输出所有参数|
|$#|传递到脚本的参数个数|
|$$|当前进程的进程号PID|
|$?|显示最后命令的退出状态；0表示没有错误，其他任何值表明有错误|
|$!|后台运行的最后一个进程的进程号pid|

例子：
```
[root@localhost ~]# vim special_vars.sh 
  #!/bin/bash
  echo "$* all parameters"
  echo "$# the amount/num of parameters"
  echo "$$ PID of this script"
  touch /tmp/a.txt &
  echo "$! PID of the last cmd in background"
  echo "$? the result of the last cmd"

[root@localhost ~]# bash special_vars.sh 11 22 33 44 55
11 22 33 44 55 all parameters
5 the amount/num of parameters
32834 PID of this script
32835 PID of the last cmd in background
0 the result of the last cmd
```
# 3. 数学运算
### 3.1 expr 做比较时，输出结果假为0，1为真；特殊符号用转义符
```
[root@localhost ~]# expr 2 \> 5
0
[root@localhost ~]# expr 8 \> 5
1
[root@localhost ~]# expr 3 \+ 5
8
[root@localhost ~]# expr 3 \* 5
15
[root@localhost ~]# expr 2 \/ 5
0
[root@localhost ~]# expr 4 \/ 2
2
```
```
[root@localhost ~]# expr length "ni hao"
6
[root@localhost ~]# expr substr "ni hao" 2 4
i ha
```

### 3.2 使用$(( ))

- 格式：$（（表达式1，表达2））
- 特点：
  1. 在双括号结构中，所有表达式可以像c语言一样，如：a++,b--等。a++  等价于 a=a+1 
  2. 在双括号结构中，所有变量可以不加入：“$”符号前缀。
  3. 双括号可以进行逻辑运算，四则运算
  4. 双括号结构 扩展了for，while,if条件测试运算
  5. 支持多个表达式运算，各个表达式之间用“，”分开


- 常用的算数运算符

  |运算符|意义|
  |-|-|
  |++   --|递增及递减，可前置也可以后置|
  |+  -  ! ~|一元运算的正负号 逻辑与取反|
  |+  -  *  /   %|加减乘除与余数|
  |<   <=   >   >=|比较大小符号|
  |==   !=|相等 不相等|
  |>>  <<|向左位移 向右位移|
  |& ^   \| |位的与 位的异或 位的或|
  |&&  \|\| |逻辑与 逻辑或|
  |？ :|条件判断|

```
[root@localhost ~]# a=$((1+2))
[root@localhost ~]# echo $a
3

[root@localhost ~]# echo $(((1+100)*100/2))     # 1到100求和
5050
```


# 4. 实战-升级系统中的java版本到1.8版本-为后期安装Hadoop集群做准备

未找到安装包，但是系统已经是最新版本，应该无需升级，升级方法如下
```
[root@localhost ~]# java -version
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```

安装jdk java运行环境

上传jdk-8u161-linux-x64.rpm软件包到xuegod63
```
[root@localhost ~]# rpm -ivh jdk-8u161-linux-x64.rpm
[root@localhost ~]#rpm -pql /root/jdk-8u161-linux-x64.rpm   #通过查看jdk的信息可以知道jdk的安装目录在/usr/java 
[root@localhost ~]#vim /etc/profile   #在文件的最后添加以下内容：
  export JAVA_HOME=/usr/java/jdk1.8.0_161
  export JAVA_BIN=/usr/java/jdk1.8.0_161/bin
  export PATH=${JAVA_HOME}/bin:$PATH
  export CLASSPATH=.:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar

[root@localhost ~]#source /etc/profile #使配置文件生效
```

验证java运行环境是否安装成功：
```
[root@localhost ~]#  java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```
如果出现安装的对应版本，说明java运行环境已经安装成功。

























-----

# Issue
搞坏了```echo "export PATH=/opt/:$PATH" >> /etc/profile```, 这一句应该有问题

```
[root@localhost ~]# vim
bash: vim: command not found...
[root@localhost ~]# cat
bash: cat: command not found...
[root@localhost ~]# ls
bash: ls: command not found...

[root@localhost ~]# echo $PATH
/opt/:/opt/:%PATH
[root@localhost ~]# export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
[root@localhost ~]# vim /etc/profile
  export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
  
[root@localhost ~]# source /etc/profile  
```
