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

- 加法计算器
  ```
  
  ```



