# sshd服务搭建管理和防止暴力破解

Issue： yum install时，无法安装正在被其他程序调用
```
Another app is currently holding the yum lock; waiting for it to exit...
  The other application is: PackageKit
    Memory :  42 M RSS (998 MB VSZ)
    Started: Mon Mar 11 09:36:37 2019 - 20:55 ago
    State  : Sleeping, pid: 11129
^C

Exiting on user cancel.
[root@localhost ~]# kill -9 11129
```

Issue： CentOS关闭休眠和屏保模式 -> 开机黑屏！！！！！！！！！
```shell
vim /etc/X11/xorg.conf

Section "ServerFlags"
       Option "BlankTime" "0"
       Option "StandbyTime" "0"
       Option "SuspendTime" "0"
       Option "OffTime" "0"
EndSection

Section "Monitor"
       Option "DPMS" "false"
EndSection
```
