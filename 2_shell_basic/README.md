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

