# Linux网络管理技术
1. OSI七层模型和TCP/IP
2. linux网络相关的调试命令
3. 实战-在局域网中使用 awl伪装MAC地址进行多线程SYN洪水攻击

# 1. OSI七层模型和TCP/IP
- OSI七层模型：OSI（Open System Interconnection）开放系统互连参考模型是国际标准化组织（ISO）制定的一个用于计算机或通信系统间互联的标准体系。
- TCP/IP四层模型：TCP/IP参考模型是计算机网络的祖父ARPANET和其后继的因特网使用的参考模型。
- 七层模型优点：
  1. 把复杂的网络划分成为更容易管理的层（将整个庞大而复杂的问题划分为若干个容易处理的小问题）
  2. 没有一个厂家能完整的提供整套解决方案和所有的设备，协议. 
  3. 独立完成各自该做的任务，互不影响，分工明确，上层不关心下层具体细节，分层同样有益于网络排错
  功能与代表设备
- TCP与UDP的区别：
  1. 基于连接与无连接；
  2. 对系统资源的要求（TCP较多，UDP少）；
  3. UDP程序结构较简单；UDP信息包的标题很短，只有8个字节，相对于TCP的20个字节信息包的额外开销很小。所以传输速度可更快
  4. TCP保证数据正确性，UDP可能丢包；TCP保证数据顺序，UDP不保证。
  场景： 视频，语音通讯使用udp，或网络环境很好，比如局域网中通讯可以使用udp。  udp数据传输完整性，可以通过应用层的软件来校对就可以了。
  tcp传文件，数据完整性要求高。
- 如果你不知道哪个端口对应哪个服务怎么办？如873端口是哪个服务的？
  ```
  [root@localhost ~]# vim /etc/services 
  ```
  此文件可以查看常用端口对应的名字。iptables或netstat要把端口解析成协议名时，都需要使用到这个文件。另外后期xinetd服务管理一些小服务时，也会使用到此文件来查询对应的小服务端口号。注：有的服务是UDP和TCP端口都会监听的
  
- IP地址分类, 常见的地址是A、B、C 三类
  1. A类地址:范围从0-127，0是保留的并且表示所有IP地址，而127也是保留的地址，并且是用于测试环回口用的。因此A类地址的可用的范围其实是从1-126之间。以子网掩码：255.0.0.0.
  2. B类地址：范围从128-191，如172.168.1.1，以子网掩码来进行区别：255.255.0.0
  3. C类地址：范围从192-223，以子网掩码来进行区别： 255.255.255.0
    - ABC 3类中私有IP地址范围：
      - A： 10.0.0.0--10.255.255.255  /8
      - B:  172.16.0.0--172.31.255.255  /16
      - C:  192.168.0.0--192.168.255.255  /24
  4. D类地址：范围从224-239，被用在多点广播(Multicast)中。多点广播地址用来一次寻址一组计算机，它标识共享同一协议的一组计算机。
  5. E类地址：范围从240-254，为将来使用保留。 

> 互动： ping 127.0.0.1 可以ping通。ping 127.23.23.23 可以ping通吗？
> 结论：这个127这个网段都用于环回口
> ```
>  [root@localhost ~]# ping 127.3.4.5
>  PING 127.3.4.5 (127.3.4.5) 56(84) bytes of data.
>  64 bytes from 127.3.4.5: icmp_seq=1 ttl=64 time=0.105 ms
>  64 bytes from 127.3.4.5: icmp_seq=2 ttl=64 time=0.054 ms
>  ```

# 2. linux网络相关的调试命令

### 2.1 查看网络连接是否正常
```
[root@localhost ~]# mii-tool ens33
ens33: negotiated 1000baseT-FD flow-control, link ok
```
### 2.2 查看ip
```
[root@localhost ~]# ifconfig
```
常见的一些网络接口
- eth0 ..... eth4 ...   以太网接口(linux6)
- waln0      无线接口
- eno177776  以太网接口 (linux7)
- ens33   以太网接口(linux7)
- bond0  team0   网卡绑定接口
- virbr0  虚拟交换机桥接接口
- br0    虚拟网桥接口
- lo      本地回环接口
- vnet0   KVM虚拟机网卡接口

###  2.3 修改网卡IP地址 - 手工修改网卡配置文件

```
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33 

TYPE="Ethernet"         # 设置类型是以太网设备
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"        # 参数：static静态IP 或dhcp 或none无（不指定），如是none，配上IP地址和static效果一样
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"            # 网卡名字
UUID="542350e0-61ad-4f1f-937b-7ee3c71f0f8d"        # 网卡UUID，全球唯一
DEVICE="ens33"          # 设备名字，在内核中识别的名字
ONBOOT="yes"            # 启用该设备，如果no，表示不启动此网络设备
IPADDR=192.168.0.162
PREFIX=24
GATEWAY=192.168.0.1
DNS1=114.114.114.114
DNS2=8.8.8.8
~                      
```

### 2.4 添加一个新网卡
vm add a new network adapter

```
[root@localhost ~]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.162  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::24cf:1100:bafc:e91e  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:57:73:32  txqueuelen 1000  (Ethernet)
        RX packets 11802  bytes 1338943 (1.2 MiB)
        RX errors 0  dropped 5  overruns 0  frame 0
        TX packets 304  bytes 31115 (30.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens39: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.89  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::a652:9222:f87a:530d  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:57:73:3c  txqueuelen 1000  (Ethernet)
        RX packets 1273  bytes 103404 (100.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 26  bytes 4590 (4.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
默认新增加的网卡没有配置文件，现在手动添加一个

```
[root@localhost ~]# cd /etc/sysconfig/network-scripts/

[root@localhost network-scripts]# cp ifcfg-ens33 ifcfg-ens39

[root@localhost network-scripts]# cp vim ifcfg-ens39
NAME=ens39
UUID=c713acec-674b-411d-9e61-646482a292ca   #这一行删除掉
DEVICE=ens39
IPADDR=192.168.0.33   #改成33 IP

[root@localhost ~]# systemctl restart NetworkManager   
[root@localhost ~]# ifconfig  #发现ens39 ，IP地址没有修改成功
[root@localhost ~]# service  network restart   #重启网络服务生效
[root@localhost ~]# ifconfig  #发现ens39 ，IP地址配置成功
```
- 启动关闭指定网卡
```
[root@localhost ~]# ifconfig ens39 down
[root@localhost ~]# ifconfig ens39 up
```
- 临时配置IP地址
```
[root@localhost ~]# ifconfig ens39 192.168.0.89
```
- 给一个网络临时配置多个IP地址
```
[root@localhost ~]# ifconfig ens39:1 192.168.3.3
```

### 2.5 查看端口的监听状态
netstat 命令： 查看系统中网络连接状态信息。常用的参数格式 :  ```netstat -anutp```  
- a, --all  显示本机所有连接和监听的端口
- n, --numeric    don't resolve names  以数字形式显示当前建立的有效连接和端口
- u  显示udp协议连接
- t  显示tcp协议连接
- p, --programs   显示连接对应的PID与程序名

```
[root@localhost ~]# netstat -anutp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd              
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      9503/sshd           
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      9502/cupsd          
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      9816/master         
tcp        0      0 192.168.0.162:22        192.168.0.219:4737      ESTABLISHED 11920/sshd: root@pt 
tcp        0      0 192.168.0.162:22        192.168.0.219:4434      ESTABLISHED 10353/sshd: root@pt 
```
- Proto===连接协议的种类
- Recv-Q====接收到字节数
- Send-Q====从本服务器，发出去的字节数
- Local Address====本地的IP地址，可以是IP，也可以是主机名
- Foreign Address====远程主机的IP 地址
- 网络连接状态STATE:
  - CLOSED ： 初始（无连接）状态。
  - LISTEN ：  侦听状态，等待远程机器的连接请求。
  - ESTABLISHED： 完成TCP三次握手后，主动连接端进入ESTABLISHED状态。此时，TCP连接已经建立，可以进行通信。
  - TIME_WAIT ：  在TCP四次挥手时，主动关闭端发送了ACK包之后，进入TIME_WAIT状态，等待最多MSL时间，让被动关闭端收到ACK包。
    - MSL，即Maximum Segment Lifetime，一个数据分片（报文）在网络中能够生存的最长时间，在RFC 793中定义MSL通常为2分钟，即超过两分钟即认为这个报文已经在网络中被丢弃了。对于一个TCP连接，在双方进入TIME_WAIT后，通常会等待2倍MSL时间后，再关闭掉连接，作用是为了防止由于FIN报文丢包，对端重发导致与后续的TCP连接请求产生顺序混乱

- 实战：服务器上有大量TIME_WAI连接，如何优化TCP连接，快速释放tcp连接 ？
> [root@localhost ~]# netstat  -antup | grep   TIME_WAI 
> tcp        0      0 123.57.82.225:80            111.196.245.241:4002        TIME_WAIT   -                   
> tcp        0      0 123.57.82.225:80            111.196.245.241:3970        TIME_WAIT   -                   
> tcp        0      0 123.57.82.225:80            111.196.245.241:4486        TIME_WAIT   -                   
> tcp        0      0 123.57.82.225:80            111.196.245.241:3932        TIME_WAIT   -                   
> tcp        0      0 123.57.82.225:80            111.196.245.241:3938        TIME_WAIT   -   

```
[root@localhost ~]# cat /proc/sys/net/ipv4/tcp_fin_timeout 
60

[root@localhost ipv4]# echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout  #通过缩短时间time_wait时间来快速释放链接
```

### 2.6 路由信息
```
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.0.1     0.0.0.0         UG    100    0        0 ens33
192.168.0.0     0.0.0.0         255.255.255.0   U     100    0        0 ens33
192.168.3.0     0.0.0.0         255.255.255.0   U     0      0        0 ens39
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
```
注：```0.0.0.0         192.168.0.1     0.0.0.0         UG    100    0        0 ens33```, 0.0.0.0是32位二进制转换成十进制的写法。32位子网掩码都为0。表示IP地址32位都是主机位。如果IP地址是0.0.0.0，子网掩码也是0.0.0.0，则表示所有的IP地址，或者是没有IP地址。

