# 配置 docker 静态 IP 地址-配置 docker 私有仓库

有没有privilege都可以，容器id或者name都可以指定
```
[root@server162 ~]# docker run -itd --privileged centos:httpd bash
566fc8a03074a875cfb8a9370cae6679c49a1f7115d61e3744ae74b6faac716f
[root@server162 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
566fc8a03074        centos:httpd        "bash"              4 seconds ago       Up 3 seconds                            festive_davinci
[root@server162 ~]# pipework br0 566fc8a03074 192.168.0.12/24@192.168.0.1
[root@server162 ~]# ping 192.168.0.12
PING 192.168.0.12 (192.168.0.12) 56(84) bytes of data.
64 bytes from 192.168.0.12: icmp_seq=1 ttl=64 time=0.284 ms
64 bytes from 192.168.0.12: icmp_seq=2 ttl=64 time=0.093 ms
^C
```
```
[root@server162 ~]# docker run -itd --name nopri centos:httpd bash
bb9eda1d51a69a32f9e707c72510084133312197afc4f379bb80d9e56df3f879
[root@server162 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
bb9eda1d51a6        centos:httpd        "bash"              4 seconds ago       Up 3 seconds                            nopri
566fc8a03074        centos:httpd        "bash"              2 minutes ago       Up 2 minutes                            festive_davinci
[root@server162 ~]# pipework br0 nopri 192.168.0.13/24@192.168.0.1
[root@server162 ~]# ping 192.168.0.13
PING 192.168.0.13 (192.168.0.13) 56(84) bytes of data.
64 bytes from 192.168.0.13: icmp_seq=1 ttl=64 time=0.501 ms
```
一定哟啊保持docker容器运行，否则报错
```
Docker inspect returned invalid PID 0
```
