# 结构化命令case和for, while循环

1. case - 流程控制语句
2. for 循环语句
3. while 循环语句和循环嵌套
4. 实战-3个shell脚本实战

# 1. 流程控制语句：case

```shell
case var in 
  ocndition1)
    command1
    ;;
  condition2)
    command2
    ;;
  ...
  *)
    default command
esac
```

- 示例1
  ```shell
  [root@localhost ~]# vim case.sh
    #!/bin/bash
    cat<<eof
    ***************
    ** 1. backup **
    ** 2. copy   **
    ** 3. quit   **
    eof
    read -p "please select: " choice
    case $choice in
            1|backup)
                    echo "backup"
                    ;;
            2|copy)
                    echo "copy"
                    ;;
            3|quit)
                    echo "quit"
                    exit
                    ;;
            *)
                    echo "please select"
    esac
  ```
  ```shell
  [root@localhost ~]# bash case.sh 
  ***************
  ** 1. backup **
  ** 2. copy   **
  ** 3. quit   **
  please select: 3
  quit
  [root@localhost ~]#
  ```
- 示例2-启动httpd脚本
  ```shell
  [root@localhost ~]# vim httpd.sh
    #!/bin/bass
    case $1 in
            start)
                    systemctl $1 httpd
                    ps -aux | grep httpd
                    echo "$1 httpd"
                    ;;
            stop)
                    systemctl $! httpd
                    ps -aux | grep https
                    echo "$1 httpd"
                    ;;
            status)
                    systemctl $1 httpd
                    ;;
            restart)
                    systemctl $1 httpd
                    ;;
            *)
                    echo "Usage: $0 start|stop|status|restart"
    esac
  ```
  
  ```shell
  [root@localhost ~]# bash httpd.sh 
  Usage: httpd.sh start|stop|status|restart
  [root@localhost ~]# bash httpd.sh status
  ● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
     Active: inactive (dead)
       Docs: man:httpd(8)
             man:apachectl(8)
  ```
# 2.循环语句 for

```shell
for var in list ; do
  commands
done
```

- 可以直接读取in 后面的值，默认以空格做分隔
  ```shell
  [root@localhost ~]# vim for.sh
    #!/bin/bash
    for var in a b c ; do
            echo "$var"
    done

  [root@localhost ~]# bash for.sh 
  a
  b
  c
  ```
  
- 列表中的复杂值，可以使用 引号或转义字符”/”来加以约束
  ```shell
  [root@localhost ~]# vim for.sh
    #!/bin/bash
    for var in "a b c" d "e f " \$ ; do
            echo "$var"
    done
   
  [root@localhost ~]# bash for.sh 
  a b c
  d
  e f 
  $
  ```
- 从变量中取值
  ```shell
  #!/bin/bash
  list="a b \$"
  for var in $list ; do
          echo "$var"
  done
  ```

- 从命令中取值
  ```
  #!/bin/bash"
  for var in `ls` ; do
          echo "$var"
  done
  ```
- 自定义shell分隔符
  - 默认情况下，base shell会以空格、制表符、换行符做为分隔符。通过IFS来自定义为分隔符
  - 指定单个字符做分隔符：
    - ```IFS=：```   #以：冒号做分隔符
  - 可以指定多个
    - ```IFS='\n':;"```    #这个赋值会将反斜杠、n、冒号、分号和双引号作为字段分隔符。

  - ```$'\n'```与```'\n'```时的区别
    - ```IFS='\n'```    #将字符\和字符n作为IFS的换行符。
    - ```IFS=$'\n'```   #正真的使用换行符, 回车做为字段分隔符。

  ```
  [root@localhost ~]# vim for.sh
    #!/bin/bash
    IFS=$'\n'            # 回车 分隔
    for var in `cat /etc/hosts` ; do
            echo "$var"
    done

  [root@localhost ~]# bash for.sh 
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  ```
- C语言风格的for
  ```
  #!/bin/bash
  for ((i=1 ,j=10 ; i<=10 ; i++,j-- )) ; do   # ((  ))有没有空格都可以，注意，和；的使用
          echo "$i - $j"
  done
  ```

# 3. while循环语句和循环嵌套
### 3.1 while
```shell
while condition
do
  command
done
```
- 降序输出10到1的数字
  ```
  #!/bin/bash
  var=10
  while [ $var -gt 0 ]
  do
      echo $var
      var=$[$var-1]
  done
  ```

- 两数相乘
  ```
  #!/bin/bash
  var=1
  while [ $var -lt 10 ] ; do
          echo "$var * $var = $(( $var * $var  ))"    # $((...))数字运算
  #       var=$[ $var+1 ]   # 二者都有变量自加1的效果
          (( var++ ))       # 二者都有变量自加1的效果
  done
  ```
  ```
  1 * 1 = 1
  2 * 2 = 4
  ...
  9 * 9 = 81
  ```
### 3.2 嵌套循环
- 批量创建用户
  ```shell
  [root@localhost ~]# echo "user1 user2 user3" > user.txt
  
  [root@localhost ~]# vim adduser.sh 
    #!/bin/bash
    userlist=`cat user.txt`                   # 读取文档中用户
    for user in $userlist ; do          
            id $user &> /dev/null             # 运行id username命令，返回信息指定为空
            if [ $? -ne 0 ] ; then            # 如果上一条命令运行成功，则用户已存在，否则可以创建该新用户
                                              # if [   ] 方括号为test命令，作比较
                    useradd $user
                    echo "user $user created"
            else
                    echo "$user exsits"
            fi
    done
  
  [root@localhost ~]# bash adduser.sh 
  user user1 created
  user user2 created
  user user3 created  
  ```
- 九九乘法表
  ```shell
  [root@localhost ~]# vim 99.sh
    #!/bin/bash
    for i in `seq 9` ; do                        # seq 9 = range(9) = 0-9
            for j in `seq $i` ; do
                    res=$(($i * $j))             # 2个数数学运算 $(( $i * $j ))
                    echo -n  "$i * $j = $res"    # echo -n 所有内容输出为一行
            done
            echo ""
    done
  
  [root@localhost ~]# bash 99.sh 
  1 * 1 = 1
  2 * 1 = 22 * 2 = 4
  3 * 1 = 33 * 2 = 63 * 3 = 9
  4 * 1 = 44 * 2 = 84 * 3 = 124 * 4 = 16
  5 * 1 = 55 * 2 = 105 * 3 = 155 * 4 = 205 * 5 = 25
  6 * 1 = 66 * 2 = 126 * 3 = 186 * 4 = 246 * 5 = 306 * 6 = 36
  7 * 1 = 77 * 2 = 147 * 3 = 217 * 4 = 287 * 5 = 357 * 6 = 427 * 7 = 49
  8 * 1 = 88 * 2 = 168 * 3 = 248 * 4 = 328 * 5 = 408 * 6 = 488 * 7 = 568 * 8 = 64
  9 * 1 = 99 * 2 = 189 * 3 = 279 * 4 = 369 * 5 = 459 * 6 = 549 * 7 = 639 * 8 = 729 * 9 = 81
  ```
