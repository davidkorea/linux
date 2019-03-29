# 配置 docker 静态 IP 地址-配置 docker 私有仓库

1. 创建 docker 静态化 IP
2. 创建 docker 私有化仓库
3. 使用阿里云私有仓库存储自己的 docker 镜像

# 1. 创建 docker 静态化 IP

- Docker 的 4 种网络模式
  1. host 模式，使用--net=host 指定
  2. container 模式，使用--net=container:NAME_or_ID 指定
  3. none 模式，使用--net=none 指定
  4. bridge 模式，使用--net=bridge 指定，默认设置。默认选择 bridge 的情况下，容器启动后会通过 DHCP 获取一个地址，这可能丌是我们想要的，在centos7 系统上， docker 环境下可以使用 pipework 脚本对容器分配固定 IP（这个 IP 可以是和物理机同网段 IP）

- docker 默认是 bridge（--net=bridge）模式，相当于 VMware 中 NAT 模式。
- docker 环境下可以使用 pipework 脚本对容器分配固定 IP，相当于 VMware 中桥接模式。
- Pipework 有个缺陷，容器重吭后 IP 设置会自动消失，需要重新设置。

## 1.1  配置桥接网络





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
