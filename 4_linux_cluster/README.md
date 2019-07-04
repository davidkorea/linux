linux cluster

- `ulimit -a`, open files 1024 by default 同时打开文件数1024
  - `ulimit -n 10240`, 改为10240
 - `./sbin/nginx -s reload`, 更改配置文件后，重新加载

- SAN使用ext，xfs文件系统不能实现文件同步，GFS格式可以实现不同客户端的共享存储
