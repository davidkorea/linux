# FTP

1. 支持匿名传输 在pub目录下 
  -  配置文件给匿名权限
  - /var/ftp/pub 改主组权限chown ftp.ftp /ver/ftp/pub 才可以上传
  - 如果不知道当前服务是哪个用户在哦运行，那么目录权限全部改了 755 或766
    - 其实可以ps aux | grep vsftp 来查看当前服务是哪个用户在使用
