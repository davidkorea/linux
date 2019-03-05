# 条件测试语句和if流程控制语句的使用
1. read命令键盘读取变量的值
2. 流程控制语句if
3. test测试命令
4. 流程控制过程中复杂条件和通配符
5. 实战-3个shell脚本实战

# 1. read命令键盘读取变量的值

从键盘读取变量的值，通常用在shell脚本中与用户进行交互的场合。该命令可以一次读取多个变量的值，变量和输入的值都需要使用空格隔开。在read命令后面，如果没有指定变量名，读取的数据将被自动赋值给特定的变量REPLY

- 读取多个值，从标准输入读取一行，直至遇到第一个空白符或换行符。把用户键入的第一个词存到变量first中，把该行的剩余部分保存到变量last中
  ```
  [root@localhost ~]# read first last
  1 2
  [root@localhost ~]# echo $first $last 
  1 2
  ```
  
- ```read -s ``` 将你输入隐藏，值赋给passwd。隐藏密码信息
  ```
  [root@localhost ~]# read -s passwd
  [root@localhost ~]# echo $passwd 
  11111
  ```
- ```read -t ``` 输入的时间限制, 超过5秒没有输入，直接退出
  ```
  [root@localhost ~]# read -t 5 time
  ```
- ```read -n ``` 输入的长度限制, 最多只接受3个字符
  ```
  [root@localhost ~]# read -n 3 length
  jgl[root@localhost ~]# echo $length 
  jgl
  ```
- ```read -r ```允许让输入中的内容包括：空格、/、\、 ？等特殊字符串
  ```
  [root@localhost ~]# read -r line
  hjkjh jg jk $#!&^ kjh
  [root@localhost ~]# echo $line 
  hjkjh jg jk $#!&^ kjh 
  ```
- ```read -p ``` 用于给出提示符，同echo –n “…“来给出提示符
  ```
  [root@localhost ~]# read -p "pls input: " pass
  pls input: 12345
  [root@localhost ~]# echo $pass
  12345
  ```
  ```
  [root@localhost ~]# echo -n "please input: "; read pass
  please input: 12345
  [root@localhost ~]# echo $pass
  12345
  ```
