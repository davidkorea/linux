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
