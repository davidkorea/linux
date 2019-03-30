# 配置docker静态IP地址 & 配置docker私有仓库

1. 创建 docker 静态化 IP
2. 创建 docker 私有化仓库
3. 使用阿里云私有仓库存储自己的 docker 镜像

# 1. 创建docker静态IP

- Docker 的 4 种网络模式
  1. host 模式，使用--net=host 指定
  2. container 模式，使用--net=container:NAME_or_ID 指定
  3. none 模式，使用--net=none 指定
  4. bridge 模式，使用--net=bridge 指定，默认设置。默认选择 bridge 的情况下，容器启动后会通过 DHCP 获取一个地址，这可能不是我们想要的，在centos7 系统上， docker 环境下可以使用 pipework 脚本对容器分配固定 IP（这个 IP 可以是和物理机同网段 IP）

- docker 默认是 bridge（--net=bridge）模式，相当于 VMware 中 NAT 模式。
- docker 环境下可以使用 pipework 脚本对容器分配固定 IP，相当于 VMware 中桥接模式。
- Pipework 有个缺陷，容器重吭后 IP 设置会自动消失，需要重新设置。

## 1.1  配置桥接网络
桥接本地物理网络的目的,是为了局域网内用户方便访问 docker 实例中服务,不需要各种端口映射即可访问服务。 但是这样做,又违背了 docker 容器的安全隔离的原则,工作中辩证的选择.
#### 1. 挂载cdrom后，安装bridge-utils
```rpm -ivh /mnt/Packages/bridge-utils-1.5-9.el7.x86_64.rpm```
#### 2. 配置原有ens33网卡
```diff
  TYPE="Ethernet"
  PROXY_METHOD="none"
  BROWSER_ONLY="no"
  BOOTPROTO="none"
  DEFROUTE="yes"
  IPV4_FAILURE_FATAL="no"
  IPV6INIT="yes"
  IPV6_AUTOCONF="yes"
  IPV6_DEFROUTE="yes"
  IPV6_FAILURE_FATAL="no"
  IPV6_ADDR_GEN_MODE="stable-privacy"
  NAME="ens33"
  UUID="d42fbd6a-105b-4925-a39e-35a7b8101e72"
  DEVICE="ens33"
  ONBOOT="yes"
- IPADDR=192.168.0.162
- GATEWAY=192.168.0.1
- DNS1=168.126.63.1
- DNS2=164.124.101.2
+ BRIDGE="br0"
```
#### 3. 创建网桥ifcfg-br0配置文件
```diff
+ TYPE="Bridge"
+ DEVICE="br0"
+ NM_CONTROLLED="yes"
+ ONBOOT="yes"
+ BOOTPROTO=none
+ IPADDR=192.168.0.162
+ GATEWAY=192.168.0.1
+ DNS1=168.126.63.1
+ DNS2=164.124.101.2
```
#### 4. 重启网络
```
[root@server162 network-scripts]# service network restart
Restarting network (via systemctl):             [ 确定 ]
```
## 1.2 pipework 工具包
#### 1. 下载
```
[root@server162 ~]# git clone https://github.com/jpetazzo/pipework.git
```
#### 2. 复制pipework脚本至/usr/local/bin/ 
pipwork是一个shell脚本，所以不用安装，把脚本复制到/usr/local/bin下即可使用pipwork命令
```
[root@server162 ~]#  cp /root/pipework/pipework /usr/local/bin/
```

## 1.3 配置docker静态ip

- 一定要保持docker容器-d运行，否则```Docker inspect returned invalid PID 0```报错
- 有没有docker --privileged都可以
  - --privileged=true, 允许开启特权功能。大约在 docker 0.6 版以后，privileged 被引入 docker。使用该参数，container 内的 root 拥有真
正的 root 权限。否则，container 内的 root 只是外部的一个普通用户权限。使用 privileged 的容器，可以看到徆多 host 上的设备，并且可以执行 mount。甚至允许你在 docker容器中启动docker 容器。
- pipework可以指定容器id或者name
- pipework语法：```pipework 网桥名 容器实ID或name 分配给容器的IP/掩码@网关```

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

## 1.4 实战： 使用静态IP 启动一个web服务器

#### 0. systemctl start httpd 等同 httpd
```
[root@server162 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
[root@server162 ~]# netstat -anutp | grep 80
tcp        0      0 0.0.0.0:6000            0.0.0.0:*               LISTEN      9480/X              
tcp6       0      0 :::6000                 :::*                    LISTEN      9480/X    

[root@server162 ~]# httpd
[root@server162 ~]# netstat -anutp | grep 80
tcp        0      0 0.0.0.0:6000            0.0.0.0:*               LISTEN      9480/X              
tcp6       0      0 :::80                   :::*                    LISTEN      10443/httpd         
tcp6       0      0 :::6000                 :::*                    LISTEN      9480/X              
```

#### 1. 运行docker并安装httpd

```
[root@server162 ~]# docker run -itd --name webserver centos:httpd 
d72c7e99a3ba1c688a886ad19e242976631d7b8973d9a128b8f0dcb81b704e78
[root@server162 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
d72c7e99a3ba        centos:httpd        "/bin/bash"         8 seconds ago       Up 5 seconds                            webserver

[root@server162 ~]# docker exec -it webserver bash
[root@d72c7e99a3ba /]# ifconfig
bash: ifconfig: command not found
```
#### 2. 安装net-tools工具
```
[root@d72c7e99a3ba /]# yum install -y net-tools
... ...
Installed:
  net-tools.x86_64 0:2.0-0.24.20131004git.el7                                     

Complete!
[root@d72c7e99a3ba /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 0.0.0.0
        inet6 fe80::42:acff:fe11:2  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 160  bytes 342129 (334.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 150  bytes 11324 (11.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[root@d72c7e99a3ba /]# exit
exit
```
#### 3. 配置静态ip
```
[root@server162 ~]# pipework br0 webserver 192.168.0.12/24@192.168.0.1
[root@server162 ~]# ping 192.168.0.12
PING 192.168.0.12 (192.168.0.12) 56(84) bytes of data.
64 bytes from 192.168.0.12: icmp_seq=1 ttl=64 time=0.299 ms

[root@server162 ~]# docker exec -it webserver bash
[root@d72c7e99a3ba /]# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 0.0.0.0
        inet6 fe80::42:acff:fe11:2  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 160  bytes 342129 (334.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 150  bytes 11324 (11.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.12  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::9cc1:94ff:fea6:bd5e  prefixlen 64  scopeid 0x20<link>
        ether 9e:c1:94:a6:bd:5e  txqueuelen 1000  (Ethernet)
        RX packets 4226  bytes 396324 (387.0 KiB)
        RX errors 0  dropped 518  overruns 0  frame 0
        TX packets 42  bytes 4539 (4.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### 4. 启动httpd服务
有没有privilege都无法使用systemctl start httpd
```
[root@server162 ~]# docker exec -it webserver bash
[root@d72c7e99a3ba /]# systemctl start httpd
Failed to get D-Bus connection: Operation not permitted
[root@d72c7e99a3ba /]# netstat -anutp | grep 80           # 查询为空白

[root@d72c7e99a3ba /]# httpd
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. 
Set the 'ServerName' directive globally to suppress this message
[root@d72c7e99a3ba /]# netstat -anutp | grep 80
tcp6       0      0 :::80            :::*             LISTEN      72/httpd   
```

访问 http://192.168.0.12/ 成功

# 2. 创建 docker私有化仓库

有时候使用 Docker Hub 这样的公共仓库可能丌方便（有时候无法访问），用户可以创建一个本地仓库供私人使用，这里使用官方提供的工具docker-registry镜像来配置私有库。私有仓库好处：节约带宽，可以自己定制系统。






















