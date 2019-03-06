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





