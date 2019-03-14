# FTP

1. 支持匿名传输 在pub目录下 
  -  配置文件给匿名权限
  - /var/ftp/pub 改主组权限chown ftp.ftp /ver/ftp/pub 才可以上传
  - 如果不知道当前服务是哪个用户在哦运行，那么目录权限全部改了 755 或766，一般766就够了
    - 其实可以ps aux | grep vsftp 来查看当前服务是哪个用户在使用
  - anom_other_write_enable=YES 不建议这个开启
    - 一般情况下 匿名用户 只读，执行，写，删除权限不可有
    
2. 系统用户
  - ftp web一起使用，为了安全 
  - 创建一个系统用户，但是不允许登陆服务器
    - ```useradd -s /sbin/mologin team1```
    - ```echo "11111" | passwd --stdin team
    - 
      ```
      anonymous_enable=NO
      local_enable=YES    # 允许系统创建的本地用户
      101 local_root = /var/www/html      # 新添加这一样
      102 chroot_list_enable=YES
      104 chroot_list_file = /etc/vsftp/chroot_list   # 不存在，需要自己创建，要锁定用户的用户名
      105 allpw_writeable_chroot=YES      # 允许锁定的用户有写的权限
      ```
