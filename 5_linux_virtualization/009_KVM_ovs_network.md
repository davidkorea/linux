# KVM comprehensive network based on Open vSwitch

> **为什么配置网络过程中，总是意外的ping不通。网关！！！！！**
> - 路由器就是网关设备，而且路由器的每一个接口都对应着一块网卡。路由器的网卡通过网线一端连在交换机，一端连在路由器
> - 正是由于有一端连在连交换机上，所以成为连交换机所在子网的一部分，该交换机上的所有主机可以和该路由器网卡通信。图片中红框所示
> - 路由器上通过路由表设置网卡间转发，即可实现不同网段之间的通信
> ![](https://i.loli.net/2019/05/03/5ccba91c76f3f.png)
>
> - ip地址有32位二进制组成，分为网络部分和主机部分两段组成，而区分多少位是网络部分，多少位是主机部分，有子网掩码来完成
> - 子网掩码有32位二进制组成，网络部分全部用1来表示，主机部分全部用0来表示
>   - 192.168.0.1
>   - 255.255.255.0 
>   - 或者写成 192.168.0.1/24，因为3个255由24个数字1组成
>   ![](https://i.loli.net/2019/05/03/5ccbabe0ca40c.png)
>
> - 由交换机相连的主机之间为同一个网络，相互之间通信不只需要ip，还需要MAC地址
>   - 先发送一个arp 地址解析协议 向全网发送。相应ip地址的主机收到后，发送自己的MAC给对方，从而建立起点对点通信
> - 只有跨网段通信时，才需要ip地址来指明目标主机
> - 所以跨网络通信时，直接将信息发送到网关设备，即路由器，通过路由器转发到相应网络

# 5. VLAN(GRE)虚拟机与外网通信
> 之所以使用GRE，而不使用VXLAN，是因为使用GRE更容易抓包，看到具体封装信息。而VXLAN的封装内容看不到

- 物理节点Node3，实现其他物理节点上的虚拟机与外网通信。此思路与openstack一致
- delete the vx0 port created last STEP in both node1 and node2
  - ```ovs-vsctl del-port vx0```


## 5.1 配置思路
- 首先，虚拟机网段10.0.10.0/24 可以跨主机通信，GRE
- 再次，将网络节点主机上的 虚拟机网段 与 物理网络网段进行SNAT DNAT转发，实现虚拟机与外网的通信
## 5.2 配置网络节点Node3网卡
```
Network Adapter1 - bridge       -> ens33  - 外网通信
Network Adapter1 - Host Only    -> ens37  - 内部管理用
  - 192.168.10.12
Network Adapter1 - VMnet2       -> ens38  - 虚拟机之间通信
  - 192.168.100.3
```
## 5.3 网络节点Node3创建网桥 br-ex, br-in
### 1. br-ex
- create ifcfg-br-ex, 配置文件方式创建桥，相当于brctl命令创建，可以使用brctl show查看到
  ```diff
    TYPE="Bridge"
    BOOTPROTO="none"
    NAME="br-ex"
    DEVICE="br-ex"
    ONBOOT="yes"
  + NM_CONTROLLED="no"
  + IPADDR=192.168.0.113
  + NETMASK=255.255.255.0
  + GATEWAY=192.168.0.1
  + DNS1=168.126.63.1
  ```
- modify ifcfg-ens33
  ```diff
    TYPE="Ethernet"
    BOOTPROTO="none"
    NAME="ens33"
    DEVICE="ens33"
    ONBOOT="yes"
  + NM_CONTROLLED="no"
  + BRIDGE="br-ex"
  ```
- ```service network restart```
### 2. 使用ovs创建br-in
- ```ovs-vsctl add-br br-in```

### 3. 创建在br-in上创建gre0接口
- Node3
  - ```ovs-vsctl add-port br-in gre0 -- set interface gre0 type=gre options:remote_ip=192.168.100.1```
  - 可以再此节点创建一个虚拟机进行测试gre，此处忽略测试
- Node1
  - ```ovs-vsctl add-port br-in gre0```
  - ```ovs-vsctl set interface gre0 type=gre options:remote_ip=192.168.100.254```
  
## 5.4 网络节点Node3创建虚拟路由器r0 （netns）
- ```ip netns add r0```
- 创建2对网卡，一对用于r0和br-in，一对用于r0和br-ex
  - ```ip link add rex0 type veth peer name sex0```, router external, switch external
  - ```ip link add rin0 type veth peer name sin0```,
  - ```ip link set sin0 up```, ```ip link set sex0 up```
- r0 <-> br-in
  - sin0 -> br-in ```ovs-vsctl add-port br-in sin0```
  - rin0 -> r0 ```ip link set rin0 netns r0```
- r0 <-> br-ex
  - sex0 -> br-ex ```brctl addif br-ex sex0```
  - rex0 -> r0 ```ip link set rex0 netns r0```

- r0
  ```
  [root@node4 ~]# ip netns exec r0 ip link show
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: gre0@NONE: <NOARP> mtu 1476 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/gre 0.0.0.0 brd 0.0.0.0
  3: gretap0@NONE: <BROADCAST,MULTICAST> mtu 1462 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
  14: rin0@if13: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/ether 22:ea:5a:38:fe:1a brd ff:ff:ff:ff:ff:ff link-netnsid 0
  16: rex0@if15: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/ether 76:10:36:ef:38:e0 brd ff:ff:ff:ff:ff:ff link-netnsid 0
  ```
## 5.5 网络节点Node3配置路由器r0各接口ip
- 网卡间转发
  ```
  [root@node3 ~]# vim /etc/sysctl.conf
  net.ipv4.ip_forward = 1

  [root@node4 ~]# sysctl -p
  ```
- rin0
  - ```ip netns exec r0 ifconfig rin0 10.0.10.100/24 up```,此处是虚拟机的内部ip网段，而不是192.168.100.0的VMnet2网段
  ```
  [root@node3 ~]# ip netns exec r0 ping 10.0.10.1
  PING 10.0.10.1 (10.0.10.1) 56(84) bytes of data.
  64 bytes from 10.0.10.1: icmp_seq=1 ttl=64 time=1.45 ms
  64 bytes from 10.0.10.1: icmp_seq=2 ttl=64 time=1.64 ms
  ^C
  --- 10.0.10.1 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1002ms
  rtt min/avg/max/mdev = 1.450/1.549/1.648/0.099 ms
  ```
  - 由于前面已经搭建了GRE隧道，所以可以直接ping通Node1的虚拟机10.0.10.1
- rex0
  - ```ip netns exec r0 ifconfig rex0 192.168.0.100/16```
  ```
  [root@node3 ~]# ip netns exec r0 ping 192.168.0.1
  PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
  64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=4.92 ms
  64 bytes from 192.168.0.1: icmp_seq=2 ttl=64 time=2.39 ms
  ^C
  --- 192.168.0.1 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1002ms
  rtt min/avg/max/mdev = 2.396/3.658/4.920/1.262 ms
  ```
  - 可以ping通物理网络网关

## 5.6 Node1上的虚拟机网络配置
- Node1上虚拟机网关指向Node3路由器r0的rin0的ip 10.0.10.100
  - ```route add default gw 10.0.10.100```
- 此时虚拟机可以ping通Node3路由器r0的rex0外网ip，但是ping物理网络的网关还是不可以
  ![](https://i.loli.net/2019/05/01/5cc9afdd1f7c8.png)
- 设置Node3路由器r0的SNAT规则
  - ```ip netns exec r0 iptables -t nat -A POSTROUTING -s 10.0.10.0/24 -j SNAT --to-source 192.168.0.100```


## 5.7 外网和虚拟机内网互相通信 - openstack绑定浮动IP float ip
- 删除上面一条SNAT规则
  - ``` ip netns exec r0 iptables -t nat -F```
- Node3路由器r0的rex0网卡添加第二个外网IP192.168.0.99
  - ```ip netns exec r0 ifconfig rex0:0 192.168.0.99/24```
- 配置虚拟机IP10.0.10.1 与 路由器192.168.0.99 一对一绑定。所有虚拟机10.0.10.1的流量通过192.168.0.99来对外发出，所有回到99的流量，再转发到10.0.10.1来执行
  - SNAT
    - ```ip netns exec r0 iptables -t nat -A POSTROUTING -s 10.0.10.1/32 -j SNAT --to-source 192.168.0.99```
  - DNAT
    - ```ip netns exec r0 iptables -t nat -A PREROUTING -d 192.168.0.99 -j DNAT --to-destination 10.0.10.1```

- 此时，可以直接平通192.168.0.99这个分配给虚拟机10.0.10.1的浮动ip，抓包可以显示10.0.10.1
- 可以尝试ssh连一下？挥着虚拟机安装一个httpd试一下














# 4. 不同物理机上的VM VXLAN 通信 (2 Hosts)
> **All the operations are based on STEP2 & STEP3，inlcuding DHCP. BUT NO need the peer net interface gre0 between 2 switches located in 2 physical nodes**
> - VLAN需要借助GRE协议的隧道来实现跨物理机通信，VXLAN无需借助GRE可以直接实现跨物理机通信

- VXLAN 支持 VLAN的功能，同时还不需要gre0这个接口，其机制几乎和VLAN一模一样，但是VXLAN支持比VLAN4096多得多的分段

- remove VLAN tag set last step in both phtsical nodes.
  - ```ovs-vsctl remove port vif0.0 tag 10```
  - ```ovs-vsctl remove port vif1.0 tag 20```
- remove gre0 in both phtsical nodes.
  - ```ovs-vsctl del-port gre0```

- 此时，由于切断了两个物理节点交换机的连接gre0，所以每个物理节点上的虚拟机可以互相通信，但是跨物理机无法通信

## 4.1 Node1
- 创建VXLAN接口 ```ovs-vsctl add-port br-in vx0 -- set interface vx0 type=vxlan options:remote_ip=192.168.100.2```
  ```
  [root@node2 ~]# ovs-vsctl add-port br-in vx0 -- set interface vx0 type=vxlan options:remote_ip=192.168.100.2
  [root@node2 ~]# ovs-vsctl show
  df20662b-d6a9-46dd-99c8-198b637319f9
      Bridge br-in
          Port "vif0.0"
              Interface "vif0.0"
          Port "vx0"
              Interface "vx0"
                  type: vxlan
                  options: {remote_ip="192.168.100.2"}
          Port "sif0"
              Interface "sif0"
          Port br-in
              Interface br-in
                  type: internal
          Port "vif1.0"
              Interface "vif1.0"
      ovs_version: "2.0.0"
  ```
  ```
  [root@node2 ~]# ovs-vsctl list interface vx0
  mac_in_use          : "3a:80:db:bc:d4:6c"
  mtu                 : []
  name                : "vx0"
  ofport              : 16
  ofport_request      : []
  options             : {remote_ip="192.168.100.2"}
  other_config        : {}
  statistics          : {collisions=0, rx_bytes=0, rx_crc_err=0, rx_dropped=0, rx_errors=0, rx_frame_err=0, rx_over_err=0, rx_packets=0, tx_bytes=0, tx_dropped=0, tx_errors=0, tx_packets=0}
  status              : {tunnel_egress_iface="ens38", tunnel_egress_iface_carrier=up}
  type                : vxlan
  ```

## 4.2 Node2
- ```ovs-vsctl add-port br-in vx0 -- set interface vx0 type=vxlan options:remote_ip=192.168.100.1```

此时，2个物理节点上的虚拟机全部可以互相通信













# 3. 不同物理机上的VM VLAN 通信 - GRE (2 Hosts)
> **All the operations are based on STEP2，inlcuding DHCP and the peer net interface gre0 between 2 switches located in 2 physical nodes**
> 
> - GRE: Generic Routing Encapsulation 通用路由封装
> - 借助GRE协议来实现VLAN

- Node1
  - VM1  --- VLAN1
  - VM2  -.-.-.-.-.-.-.-.-.- VLAN2
- Node2
  - VM3  --- VLAN1
  - VM4  -.-.-.-.-.-.-.-.-.-  VLAN2
## 3.1 创建虚拟机
- use DHCP (netns) created 
### 1. Node1
- VM1 
  - ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-1.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:00:01 -net tap,ifname=vif0.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```
  - DHCP IP 10.0.10.209
- VM2
  - ```qemu-kvm -m 128 -smp 1 -name cirros2 -drive file=/images/cirros/cirros-0.3.4-2.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:00:02 -net tap,ifname=vif1.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```
  - DHCP IP 10.0.10.210
### 2. Node2
- VM1 
  - ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-1.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:01:01 -net tap,ifname=vif0.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```
  - DHCP IP 10.0.10.204
- VM2
  - ```qemu-kvm -m 128 -smp 1 -name cirros2 -drive file=/images/cirros/cirros-0.3.4-2.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:01:02 -net tap,ifname=vif1.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```
  - DHCP IP 10.0.10.205

All the 4 DHCP IPs could ping each other.

## 3.2 划分VLAN

### 1. Node1
- ```ovs-vsctl set port vif0.0 tag=10```
- ```ovs-vsctl set port vif1.0 tag=20```
  ```
  [root@node2 ~]# ovs-vsctl set port vif0.0 tag=10
  [root@node2 ~]# ovs-vsctl set port vif1.0 tag=20
  [root@node2 ~]# ovs-vsctl list port vif0.0
  _uuid               : 03f6b1a9-9612-4619-a698-8250dbbd68e5
  bond_downdelay      : 0
  bond_fake_iface     : false
  bond_mode           : []
  bond_updelay        : 0
  external_ids        : {}
  fake_bridge         : false
  interfaces          : [39ba85c3-b33c-4f9f-8719-2ae6781e6085]
  lacp                : []
  mac                 : []
  name                : "vif0.0"
  other_config        : {}
  qos                 : []
  statistics          : {}
  status              : {}
  tag                 : 10
  trunks              : []
  vlan_mode           : []
  [root@node2 ~]# ovs-vsctl list port vif1.0
  _uuid               : 8b4e55c7-5178-40ae-96ff-5535046b1c70
  bond_downdelay      : 0
  bond_fake_iface     : false
  bond_mode           : []
  bond_updelay        : 0
  external_ids        : {}
  fake_bridge         : false
  interfaces          : [da2a90df-becd-49f2-9284-018679a3de92]
  lacp                : []
  mac                 : []
  name                : "vif1.0"
  other_config        : {}
  qos                 : []
  statistics          : {}
  status              : {}
  tag                 : 20
  trunks              : []
  vlan_mode           : []
  ```
- 10.0.10.209 and 10.0.10.210 ping failed.

### 2. Node2
- ```ovs-vsctl set port vif0.0 tag=10```
- ```ovs-vsctl set port vif1.0 tag=20```
- 
  ```
  [root@node3 ~]# ovs-vsctl show
  3892b9a9-c656-44dc-8ad1-d7e5711bd42b
      Bridge br-in
          Port br-in
              Interface br-in
                  type: internal
          Port "vif1.0"
              tag: 20
              Interface "vif1.0"
          Port "vif0.0"
              tag: 10
              Interface "vif0.0"
          Port "gre0"
              Interface "gre0"
                  type: gre
                  options: {remote_ip="192.168.100.1"}
      ovs_version: "2.0.0"
  ```
- 10.0.10.204 ping 10.0.10.205 failed.
- 10.0.10.205 ping 10.0.10.210 success.











# 2. 不同物理机上的VM之间通信 - GRE (2 Hosts)
- 2 physical node different network
- VMs on that 2 nodes can communicate
- GRE in OSI layer3，是一种隧道技术，使用一种报文来承载和传输另一种报文，如以太网帧
- 为简化，将2个物理节点放在同一网络，虽然不同网络可以实现GRE，但是需要再配置路由
  - 当然，因为是再同一个网段，可以直接将物理网卡绑定至br-in桥上面，实现通信
  - 这里，演示GRE的方式进行连接
  - 如果两个物理节点不在同一个网段，就只能使用GRE才能实现通信（也需要路由器使得两个网段可以通信）
- 删除上一步创建的所有虚拟机，只留下br-in桥，删除其他桥和接口
- 物理节点上面的虚拟交换机br-in借助物理网卡ens38来完成隧道协议封装，但是不能将物理网卡ens38绑定至br-in上

## 2.1 Node 1
### 1. 创建DHCP（netns）
- ```yum update iproute -y```
- ```ip netns add r0```
- ```ip link add sif0 type veth peer name rif0```
- ```ip link set rif0 up```, ```ip link set sif0 up```
- ```ip link set rif0 netns r0```
- ```ovs-vsctl add-port br-in sif0```
- ```ip netns exec r0 ifconfig rif0 up```
- ```ip netns exec r0 ip addr add 10.0.10.254/24 dev rif0```
- ```yum install -y dnsmasq```
- ```ip netns exec r0 dnsmasq -F 10.0.10.200,10.0.10.220,86400 -i rif0```
- ```ip netns exec r0 ss -unl```, listen on port 67
  ```
  [root@node2 ~]# ip netns exec r0 ss -unl
  State      Recv-Q Send-Q   Local Address:Port                  Peer Address:Port              
  UNCONN     0      0                    *:53                               *:*                  
  UNCONN     0      0                    *:67                               *:*                  
  UNCONN     0      0                   :::53                              :::*    
  ```
### 2. 创建VM
- ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-1.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:00:01 -net tap,ifname=vif0.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```
  - 开机自动获取ip 10.0.10.209，并自动获取掩码255.255.255.0，虽然创建dnsmasq时并没有指定

### 3. 配置ens38（VMNET2）
- ```ip addr add 192.168.100.1/24 dev ens38```
### 4. 创建并配置GRE接口
- ```ovs-vsctl add-port br-in gre0```
- ```ovs-vsctl set interface gre0 type=gre options:remote_ip=192.168.100.2```
- 查看gre0接口
  ```
  [root@node2 ~]# ovs-vsctl list interface gre0
  _uuid               : 440d94ea-36ee-4e18-9fd5-a75719d5694f
  ......
  mac_in_use          : "b2:53:69:e6:7e:a2"
  mtu                 : []
  name                : "gre0"
  ofport              : 14
  ofport_request      : []
  options             : {remote_ip="192.168.100.2"}
  other_config        : {}
  statistics          : {collisions=0, rx_bytes=0, rx_crc_err=0, rx_dropped=0, rx_errors=0, rx_frame_err=0, rx_over_err=0, rx_packets=0, tx_bytes=0, tx_dropped=0, tx_errors=0, tx_packets=0}
  status              : {tunnel_egress_iface="ens38", tunnel_egress_iface_carrier=up}
  type                : gre
  ```

## 2.2 Node 2
### 1. 创建VM
- Node2不指定DHCP，连接GRE后，使用Node1的 DHCP服务器
- ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-1.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:01:01 -net tap,ifname=vif0.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```, 注意更改mac地址
  - 此时并没有获取的ip地址，因为GRE还没有建立
### 2. 配置ens38（VMNET2）
- ```ip addr add 192.168.100.2/24 dev ens38```，因为同在vmware的VMnet2，192.168.100.0/24已经可以通信

### 3. 创建并配置GRE接口
- ```ovs-vsctl add-port br-in gre0```
- ```ovs-vsctl set interface gre0 type=gre options:remote_ip=192.168.100.2```
- 查看gre0接口
  ```
  [root@node2 ~]# ovs-vsctl list interface gre0
  _uuid               : 440d94ea-36ee-4e18-9fd5-a75719d5694f
  ......
  mac_in_use          : "b2:53:69:e6:7e:a2"
  mtu                 : []
  name                : "gre0"
  ofport              : 14
  ofport_request      : []
  options             : {remote_ip="192.168.100.2"}
  other_config        : {}
  statistics          : {collisions=0, rx_bytes=0, rx_crc_err=0, rx_dropped=0, rx_errors=0, rx_frame_err=0, rx_over_err=0, rx_packets=0, tx_bytes=0, tx_dropped=0, tx_errors=0, tx_packets=0}
  status              : {tunnel_egress_iface="ens38", tunnel_egress_iface_carrier=up}
  type                : gre
  ```
### 4. 再次创建VM
- 删掉之前创建的虚拟机，再次创建，查看是否可以获取到dhcp ip地址
- ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-1.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:01:01 -net tap,ifname=vif0.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```
  - 开机自动获取ip 10.0.10.204
  - ping 物理节点Node1上的虚拟机 10.0.10.209 成功


## 3. tcpdump
- Node1 的ens38上抓包
  ```
  [root@node2 ~]# tcpdump -i ens38 -nn
  tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
  listening on ens38, link-type EN10MB (Ethernet), capture size 262144 bytes
  17:14:30.805701 IP 192.168.100.2 > 192.168.100.1: GREv0, length 102: IP 10.0.10.204 > 10.0.10.209: ICMP echo request, id 24577, seq 0, length 64
  17:14:30.806629 IP 192.168.100.1 > 192.168.100.2: GREv0, length 102: IP 10.0.10.209 > 10.0.10.204: ICMP echo reply, id 24577, seq 0, length 64
  ```


















# 1. Isolated Network with tagged VLAN (1 Host)
![](https://i.loli.net/2019/04/30/5cc7bce026936.png)
## 1.1 Prepare
- ```yum install -y qemu-kvm```
- ```ln -sv /usr/libexec/qemu-kvm /usr/bin/```
- ```mkdir -p /images/cirros```
- ```cp cirros-0.3.4-x86_64-disk.img cirros-0.3.4-1.img```
- ```cp cirros-0.3.4-x86_64-disk.img cirros-0.3.4-2.img```
- ```cp cirros-0.3.4-x86_64-disk.img cirros-0.3.4-3.img```
## 1.2 Isolated Network - 1 （br-in）
- qemu-ifup script
  ```bash
  #!/bin/bash
  bridge='br-in'

  if [ -n '$1' ]; then
          ip link set $1 up
          ovs-vsctl add-port $bridge $1
          [ $? -eq 0 ] && exit 0 || exit 1
  else
          echo "Error: no interface specified."
          exit 1
  fi
  ```
  - ```chmod +x /etc/qemu-ifup ```
- qemu-ifdown script
  ```bash
  #!/bin/bash
  bridge='br-in'
  if [ -n "$1" ]; then
          ip link set $1 down
          ovs-vsctl del-port $bridge $1
          [ $? -q 0 ] && exit 0 || exit 1
  else
          echo "Error: no port specified"
          exit 2
  fi
  ```
  - ```chmod +x /etc/qemu-ifdown```
  
- ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-1.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:00:01 -net tap,ifname=vif0.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown --nographic```，使用-nographic测试一下，然后退出，该虚拟机会自动被删除，检查网口是否会自动被删除
- VM1 ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-1.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:00:01 -net tap,ifname=vif0.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```

- VM2 ```qemu-kvm -m 128 -smp 1 -name cirros2 -drive file=/images/cirros/cirros-0.3.4-2.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:00:02 -net tap,ifname=vif1.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```

- ```vncviewer :5900```
  - ```ifconfig eth0 10.0.10.1 netmask 255.255.255.0```
- ```vncviewer :5901```
  - ```ifconfig eth0 10.0.10.2 netmask 255.255.255.0```
- ping each other ok

## 1.3 VLAN in same switch
- ```ovs-vsctl set port vif0.0 tag=10```, ping failed even 2 VMs on the same switch and in same subnet.
  ```
  [root@node2 ~]# ovs-vsctl list port
  _uuid               : 7e1bf174-3bcc-4ab1-b2b5-6837215a396c
  bond_downdelay      : 0
  bond_fake_iface     : false
  bond_mode           : []
  bond_updelay        : 0
  external_ids        : {}
  fake_bridge         : false
  interfaces          : [42e743b2-9410-4c74-8f05-21943c28ab3f]
  lacp                : []
  mac                 : []
  name                : "vif1.0"
  other_config        : {}
  qos                 : []
  statistics          : {}
  status              : {}
  tag                 : []
  trunks              : []
  vlan_mode           : []

  _uuid               : 6b248af5-a47e-4848-bd40-39a123b73d91
  bond_downdelay      : 0
  bond_fake_iface     : false
  bond_mode           : []
  bond_updelay        : 0
  external_ids        : {}
  fake_bridge         : false
  interfaces          : [1cb0bba2-78bb-4838-ad7f-623c42822c08]
  lacp                : []
  mac                 : []
  name                : br-in
  other_config        : {}
  qos                 : []
  statistics          : {}
  status              : {}
  tag                 : []
  trunks              : []
  vlan_mode           : []

  _uuid               : a5d43441-d4bf-4e87-a780-a6cc4243e10c
  bond_downdelay      : 0
  bond_fake_iface     : false
  bond_mode           : []
  bond_updelay        : 0
  external_ids        : {}
  fake_bridge         : false
  interfaces          : [071debbf-7695-45d1-813e-46d5c5e572fe]
  lacp                : []
  mac                 : []
  name                : "vif0.0"
  other_config        : {}
  qos                 : []
  statistics          : {}
  status              : {}
  tag                 : 10
  trunks              : []
  vlan_mode           : []
  ```
  
- ```ovs-vsctl set port vif1.0 tag=10```，all port in tag=10 VLAN，ping ok

## 1.4 Isolated Network - 2 （br-in-2）

- ```ovs-vsctl add-br br-in-2```
- qemu-ifup/down script
  - ```cp /etc/qemu-ifup /etc/qemu-ifup-2```
  ```bash
  #!/bin/bash
  bridge='br-in-2'

  if [ -n '$1' ]; then
          ip link set $1 up
          ovs-vsctl add-port $bridge $1
          [ $? -eq 0 ] && exit 0 || exit 1
  else
          echo "Error: no interface specified."
          exit 1
  fi
  ```
  - ```cp /etc/qemu-ifup /etc/qemu-ifdown-2```
  ```bash
  #!/bin/bash
  bridge='br-in-2'
  if [ -n "$1" ]; then
          ip link set $1 down
          ovs-vsctl del-port $bridge $1
          [ $? -q 0 ] && exit 0 || exit 1
  else
          echo "Error: no port specified"
          exit 2
  fi
  ```
- ```cp /images/cirros/cirros-0.3.4-1.img /images/cirros/cirros-0.3.4-3.img```
  
- VM3```qemu-kvm -m 128 -smp 1 -name cirros3 -drive file=/images/cirros/cirros-0.3.4-3.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:00:03 -net tap,ifname=vif2.0,script=/etc/qemu-ifup-2,downscript=/etc/qemu-ifdown-2 -daemonize```

- ```[root@node2 cirros]# ovs-vsctl show```
  ```
  df20662b-d6a9-46dd-99c8-198b637319f9
    Bridge "br-in-2"
        Port "vif2.0"
            Interface "vif2.0"
        Port "br-in-2"
            Interface "br-in-2"
                type: internal
    Bridge br-in
        Port br-in
            Interface br-in
                type: internal
        Port "vif1.0"
            tag: 10
            Interface "vif1.0"
        Port "vif0.0"
            tag: 10
            Interface "vif0.0"
    ovs_version: "2.0.0"
  ```
- ```vncviewer :5902```
  - ```ifconfig eth0 10.0.10.3 netmask 255.255.255.0```
  - ping 10.0.10.1 and 10.0.10.2 failed，because in different VLAN
  
## 1.5 Connect 2 vSwitch br-in and br-in-2
- create a pair of if
  - ```ip link add ifovs1 type veth peer name ifovs2```
  - ```ip link set ifovs1 up```
  - ```ip link set ifovs2 up```
- add ifovs1 to br-in
  - ```ovs-vsctl add-port br-in ifovs1```
- add ifovs2 to br-in-2
  - ```ovs-vsctl add-port br-in-2 ifovs2```
- set VM on br-in-2 VLAN tag=10
  - ```ovs-vsctl set port vif2.0 tag=10```
  - ping VMs on br-in VLAN tag=10 success.
  
- **ifovs1和ifovs2 虽然没有设置为trunk模式，默认已经可以转发各个VLAN的信息**



# 0. Basic
## 1. Open vSwitch
- 使用yum安装
```yum install -y openvswitch openvswitch-devel openvswitch-test openvswitch-debuginfor```
- 启动服务
```
systemctl enable  openvswitch
systemctl start  openvswitch
```
参考：[虚拟化云计算-centos7上安装测试Open vSwitch](https://blog.51cto.com/11555417/2163495)

- ovs vsctl add-br br0
- ovs vsctl show
- ovs vsctl list-br
- ovs vsctl del-br br0
- ovs vsctl add-port br0 eth0
- ovs-vsctl del-port ens37
- ovs vsctl list-ports br0
- ovs-vsctl list-ifaces br0
- database list
  - ovs-vsctl list br [不指定显示全部br，也可制定显示某一br]
  - ovs-vsctl list port [ens38]
  - ovs-vsctl list interface [ens38]
- database find
  - ovs-vsctl find br name=br-in
  - ovs-vsctl find port name=ens38
- database set,add,remove,clear,destroy
    - ovs-vsctl set,add,remove,clear,destroy

## 2. 创建2个计算节点
- 网卡1：仅主机，用于传送控制命令 192.168.10.10, 192.168.10.11
- 网卡2：VMnet，用于node之间的通信，内网 192.168.100.1, 192.168.100.2
  
## 3. 创建1个控制节点
- 网卡1：桥接物理网络，连接外网
- 网卡2：VMnet，作为node节点的网关 192.168.10.1
- SNAT 网卡1和2，将所有内部网转到外网，实现节点与外网通信
