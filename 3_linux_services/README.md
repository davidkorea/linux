# sshd服务搭建管理和防止暴力破解
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
