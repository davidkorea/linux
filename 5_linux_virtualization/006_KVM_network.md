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
- 创建kvm虚拟机时，要创建网卡的前半段和后半段，并需要脚本连接到相应的虚拟网桥上面，主机脚本需要手动写/etc/qemu-ifup

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


# 2. KVM虚拟机网络

## 2.1 网桥
- modinfo bridge，linux内核本身自带这个模块，需要bridge-utils的brctl命令来创建虚拟桥设备
- brctl addbr，创建桥
- brctl addif，添加端口至网桥

#### 1. 创建网桥
- ```brctl addbr br0```
#### 2. 关闭stp
- ```brctl stp br0 off```
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

## 2.2. KVM网络属性相关选项
### 2.2.1. net nic，向虚拟机创建一个网络设备
- ```qemu-kvm -net nic[,vlan=n][,macaddr=mac][,model=type][,name=name][,addr=addr][,vectors=v]```

  - qemu可以模拟多种网卡设备，默认为intel的千兆以太网网络控制器e1000类型，一般只会用来更改为半虚拟化virtio
    - ```qemu-kvm -net nic,model=?```
      ```
      qemu: Supported NIC models: ne2k_pci,i82551,i82557b,i82559er,rtl8139,e1000,pcnet,virtio
      ```
### 2.2.2. net tap, 创建网卡后半段
- ```qemu-kvm -net tap[,vlan=n][,name=name][,fd=h][,ifname=name][,script=file][,downscript=dfile]```
- 可以童工物理机的TAP网络接口连接至指定vlan n
- 也可以使用script=file，来指定网卡后半段连接至某一个网桥，脚本默认路径/etc/qemu-ifup，没有脚本的话script=no
- 通过downscript，在虚拟机关机时，将网卡后半段与网桥分离，默认路径为/etc/qemu-ifdown，没有脚本的话downscript=no
- ifname指定后半段在物理机上叫什么名字，默认为tap0
#### qemu-ifup
```
[root@server15 ~]# vim /etc/qemu-ifup
#!/bin/bash
bridge=br0

if [ -n '$1' ]; then
	ip link set $1 up
	brctl addif $bridge $1
 	[ $? -eq 0 ] && exit 0 || exit 1
else
	echo "Error: no interface specified."
	exit 1
fi

[root@server15 ~]# bash -n !$
bash -n /etc/qemu-ifup
[root@server15 ~]# chmod +x !$
chmod +x /etc/qemu-ifup
```
# 3. 创建有网络设备的KVM虚拟机
- 创建kvm虚拟机
```
qemu-kvm -m 128 -cpu host -smp 2 -name test -drive file=cirros-0.3.4-x86_64-disk.img,\
if=virtio,media=disk,format=qcow2,cache=writeback -nographic -net nic \
-net tap,ifname=vif0.0,script=/etc/qemu-ifup
```
```
$ sudo su
$ poweroff
The system is going down NOW!
Sent SIGTERM to all processes
Requesting system poweroff
[  343.310066] Power down.
/etc/qemu-ifdown: could not launch network script
```

- 查看网卡后半段已经被添加到网桥br0
  ```
  bridge name	bridge id		STP enabled	interfaces
  br0		8000.1221de5c63f8	no		vif0.0
  ```
- 查看虚拟机进程
  ```
  [root@server15 ~]# ps -aux | grep kvm
  root       8993  0.0  0.0      0     0 ?        S<   12:58   0:00 [kvm-irqfd-clean]
  root      12659  2.9  2.3 973728 235588 pts/0   Sl+  13:57   0:06 qemu-kvm -m 128 -cpu host -smp 2 -name test -drive file=cirros-0.3.4-x86_64-disk.img,if=virtio,media=disk,format=qcow2,cache=writeback -nographic -net nic -net tap,ifname=vif0.0,script=/etc/qemu-ifup
  ```
## 3.1 组建隔离网络
- 上面创建了test虚拟机
- 再次创建test1虚拟机，使用相同script，将网卡后半段绑定至相同网桥设备br0
  ```
  qemu-kvm -m 128 -cpu host -smp 2 -name test1 -drive file=cirros-0.3.4-x86_64-disk.img,\
  if=virtio,media=disk,format=qcow2,cache=writeback -nographic -net nic \
  -net tap,ifname=vif1.0,script=/etc/qemu-ifup
  ```
- 查看网桥br0端口
  ```
  bridge name	bridge id		STP enabled	interfaces
  br0		8000.5ee803a9cbdb	no		vif0.0
							vif1.0
  ```
