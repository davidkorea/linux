# 跳出循环-shift参数左移-函数的使用
1. 跳出循环
2. Shift参数左移指令
3. 函数的使用
4. 实战-自动备份mysql数据库和nginx服务启动脚本

# 1. 跳出循环

- Break：跳出整个循环
- Continue：跳过本次循环，进行下次循环

break概述：跳出当前整个循环或结束当前循环，在for、while等循环语句中，用于跳出当前所在的循环体，执行循环体之后的语句，后面如果什么也不加，表示跳出当前循环等价于break 1，也可以在后面加数字,假设break 3表示跳出第三层循环

continue概述：忽略本次循环剩余的代码，直接进行下一次循环；在for、while等循环语句中，用于跳出当前所在的循环体，执行循环体之后的语句，如果后面加的数字是1，表示忽略本次条件循环，如果是2的话，忽略下来2次条件的循环

- continue break演示
  ```shell
  #! /bin/sh
  while true
  do
     echo "*******************************"
     echo "Please select your operation:"
     echo " 1 Copy"
     echo " 2 Delete"
     echo " 3 Backup"
     echo " 4 Quit"
     echo "*******************************"
     read op
   case $op in
      1)
        continue                         # 这里加了continue后，后面的echo命令就不执行了
        echo "your selection is Copy"
        ;;
      2)
         echo "your selection is Delete"
         ;;
      3)
        echo "your selection is Backup"
        ;;
      4)
        echo "Exit ..."
        break                             # 跳出循环体
       ;;
      *)
        echo "invalide selection,please try again"
    esac
  done

  ```
- 使用交互式方法批量添加用户
  ```shell
  while :
  do
          read -p "please input prefix, pasw, num: " pre pswd num
          echo "
          *******************
          ** prefix: $pre  **
          ** pswd:   $pswd **
          ** num:    $num  **
          *******************
          "
          read -p "are you sure?[y/n]: " choice
          if [ $choice == "y" ] ; then
                  break
          fi
  done
  for i in `seq $num` ; do
          user=${pre}${i}
          id $user &> /dev/null
          if [ $? -ne 0 ] ; then
                  useradd $user
                  echo "$pswd"|passwd --stdin $user &> /dev/null
                  if [ $? -eq 0 ] ; then
                          echo -e "\033[31m $user \033[0m created"
                  fi
          else
                  echo "$user exsits"
          fi
  done
  ```
  ```
  [root@localhost ~]# bash adduser.sh 
   please input prefix, pasw, num: usr 11111 2

    *******************
    ** prefix: usr   **
    ** pswd:   11111 **
    ** num:	   2     **
    *******************

   are you sure?[y/n]: y
   usr1  created
   usr2  created
  ```
# 2. Shift参数左移指令

- shift命令用于对参数的移动(左移)，通常用于在不知道传入参数个数的情况下依次遍历每个参数然后进行相应处理（常见于Linux中各种程序的启动脚本）
- 在扫描处理脚本程序的参数时，经常要用到的shift命令，如果你的脚本需要10个或10个以上的参数，你就需要用shift命令来访问第10个及其后面的参数
- 作用：每执行一次，参数序列顺次左移一个位置，$#的值减1，用于分别处理每个参数，移出去的参数，不再可用

- 示例： 加法计算器
  ```shell
  [root@localhost ~]# vim shift.sh 
    #!/bin/bash
    [ $# -eq 0 ] && echo "pls input nums"
    sum=0
    while [ $# -gt 0 ] ; do
            sum=$(($sum+$1))
            shift
    done
    echo "sum = $sum"
  ```
  ```shell
  [root@localhost ~]# bash shift.sh 1 2 3 4 5
  sum = 15
  [root@localhost ~]# bash shift.sh 
  pls input nums
  sum = 0
  ```
  - ```shift 2```  一次移动2个参数
  
# 3. 函数的使用
- 方法1：
  ```
  function name {
      commands
  }
  ```
  注意：name是函数唯一的名称
- 方法2：name后面的括号表示你正在定义一个函数
  ```
  name(){
      commands
  }
  ```
- 调用函数语法：
  ```
  函数名 参数1 参数2 …
  ```
- 调用函数时, 可以传递参数, 在函数中用$1、$2…来引用传递的参数

- 函数的使用
  -  函数名的使用，如果在一个脚本中定义了重复的函数名，那么以最后一个为准
  - 使用return命令来退出函数并返回特定的退出码
    - 状态码的确定必需要在函数一结束就运行return返回值；状态码的取值范围（0~255）
    - exit 数字 和return 数字的区别？
      - exit整个脚本就直接退出，返回数字
      - return 只是在函数最后添加一行，然后返回数字，只能让函数后面的命令不执行，无法强制退出整个脚本的
      
    ```shell
    [root@localhost ~]# vim shift.sh 
      #!/bin/bash
      func(){
              echo "hello world"
              return 55
      }
      func

    [root@localhost ~]# bash func.sh 
    hello world
    [root@localhost ~]# echo $?
    55
    ```
  - 把函数值赋给变量使用, 函数名就相当于一个命令
    ```shell
    [root@localhost ~]# vim func.sh
      #!/bin/bash
      func(){
              read -p "pls input: " var
              echo $[ $var*2 ]
      }
      num=$(func)
      echo $num

    [root@localhost ~]# bash func.sh 
    pls input: 2
    4
    ```
  - 函数中多参数传递和使用方法
    ```
    #!/bin/bash
    func(){
            echo $[$1*2]
            echo $[$2*3]
    }
    func 2 3
    ```
  - 函数中变量的处理, 函数使用的变量类型有两种：
    - 局部变量
    - 全局变量, 默认情况下，脚本中定义的变量都是全局变量，函数外面定义的变量在函数内也可以使用

    ```
    #!/bin/bash
    function fun1 {
            num1=$[var1*2]
    }
    read -p "input a num:" var1
    fun1                        # 此处并未传递参数，根据变量名执行函数的操作
    echo the new value is: $num1
    ```
