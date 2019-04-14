# qemu-kvm 网络

- 添加虚拟网卡，并将虚拟机接入网络。三种网络属性：nic前半段，tap后半段，user
- nic
- tap，是二层设备，通信靠MAC地址
  - tap，需要在创建虚拟机时注意网卡mac不能一样，默认mac地址都是一样的，需要手动指定，否则互相不能通信
  - 另一种tun三层设备，通信靠IP地址
# KVM网络类型
## 1. 隔离模型
- 每一个虚拟机都有自己的前半段网卡eth0
- 其对应的后半段网卡在物理主机vnet0，vnet1...
- 将物理机上的各个后半段网卡都关联至虚拟网桥(虚拟交换机)上面，实现虚拟机之间的通信，但不能和物理机通信
  ![](https://i.loli.net/2019/04/13/5cb1b27917e06.png)
- 一个物理机上面，有多个隔离网络类型的虚拟机？
- linux内核有6个名称空间namespace，其中一个是 网络名称空间。未每一个虚拟子网（VMNET1，VMNET2...）提供一个相互隔离的dhcp来给新虚拟机提供ip
- 虚拟子网，其实是通过不同的虚拟网桥来实现的，不同网桥之间不能通信
- 连接至同一个虚拟网桥的虚拟机，都需要有一个dhcp服务器为其分配IP地址
- 因此每一个虚拟网桥都需要搭配一个dhcp服务器
- linux的网络命名空间即可实现多个dhcp之间的隔离
- vmware给仅主机模式和NAT模式提供来dhcp服务
- linux dnsmaster 既可以提供dhcp服务，又可以提供给dns服务
- 创建kvm虚拟机时，要创建网卡的前半段和后半段，并需要脚本连接到相应的虚拟网桥上面，主机脚本需要手动写

## 2. 路有模型
- 在 隔离模型 的基础上，给网桥上面再增加一个接口virnet1，或者说是在物理机上再创建一个虚拟网卡并关联到网桥
- 将新端口virnet1连至物理机网卡，虚拟机可以访问外网，但是外网无法回访虚拟机
  ![](https://i.loli.net/2019/04/13/5cb1b435428cd.png)
- 物理机上在创建一个虚拟网卡，一端连接物理网卡，一端连接需要虚拟网桥，来使虚拟机连接外网
- 虚拟机的网卡网关指向物理机虚拟网卡的ip地址
- 缺点依然是，路由模型无法让外网回访虚拟机
- 所以vmware中根本没有这种模型

## 3. NAT模型
- 在上述 路由模型 的基础上，添加一个NAT server（添加一条路由规则即可），即可实现外网可以访问虚拟机
- 回访时，物理网卡将数据发给NAT server，由NAT server决定发给哪一个虚拟机
  ![](https://i.loli.net/2019/04/13/5cb1b5ed66a98.png)
- 在物理机上开启NAT功能，NAT会话表可以知道将报文发送给哪一个虚拟机
- linux中的NAT服务就是一条iptables规则

## 4. 桥接模型
- 在 隔离模型 的基础上，直接将网桥设备关联至物理网卡，可以通过物理网卡访问外网
- 问题是，虚拟机和物理机都通过同一个物理网卡访问外网，回访时，应该找物理机还是虚拟机？
- 此时，物理网卡会变成一个交换机（物理网桥）。给物理机创建一个虚拟网卡，并将物理网卡自己的MAC地址给这个虚拟网卡
- 在物理网卡上打开 混杂模式，即不管mac地址是谁，物理网卡都将其信息接收下来。接下来后，再根据MAC地址（物理机mac还是虚拟机mac）来判断信息发送给谁


# 2. KVM实现网络模型

- modinfo bridge，linux内核本身自带这个模块，需要bridge-utils的brctl命令来创建虚拟桥设备
- brctl addbr，创建桥
- brctl addif，添加端口至网桥

#### 1. 创建网桥
```
[root@server15 ~]# ifconfig -a
br0: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether d6:c4:00:a1:84:1e  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
#### 2. 关闭stp
- ```[root@server15 ~]# brctl stp br0 off```
  ```
  [root@server15 ~]# brctl show
  bridge name	bridge id		STP enabled	interfaces
  br0		8000.000000000000	no		  
  ```
#### 3. 激活br0
```
[root@server15 ~]# ip link set dev br0 up
[root@server15 ~]# ip link show 

6: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether d6:c4:00:a1:84:1e brd ff:ff:ff:ff:ff:ff
```
- 要想永久有效，需要创建配置文件ifcfg-br0

## 2.1 隔离模型

## 2.2 路由模型

## 2.3 NAT模型



