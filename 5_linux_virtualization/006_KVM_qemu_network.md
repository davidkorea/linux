# 4. net namespace basics
- 创建网络名称空间
  - ```ip netns add r1```, ```ip netns add r2```
  - ```ip netns exec r1 COMMAND```
- 创建一对网卡，分别添加到r1和r2
  - ```ip link add veth1.1 type veth peer name veth1.2```，type=veth表示创建一对网卡
  ```
  [root@server162 ~]# ip link add veth1.1 type veth peer name veth1.2
  [root@server162 ~]# ip link show
  21: veth1.2@veth1.1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/ether 82:09:a5:60:8b:e8 brd ff:ff:ff:ff:ff:ff
  22: veth1.1@veth1.2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/ether 5a:6d:52:eb:69:d3 brd ff:ff:ff:ff:ff:ff
  ```
- 将网卡添加至网络名称空间
  - ```ip link set veth1.1 netns r1```
  - ```ip link set veth1.2 netns r2```
  ```
  [root@server162 ~]# ip link set veth1.1 netns r1
  [root@server162 ~]# ip netns exec r1 ifconfig -a
  lo: flags=8<LOOPBACK>  mtu 65536
          loop  txqueuelen 1000  (Local Loopback)
  
  veth1.1: flags=4098<BROADCAST,MULTICAST>  mtu 1500
          ether 5a:6d:52:eb:69:d3  txqueuelen 1000  (Ethernet)
  ```
- 更改网络名称空间内网卡名称
  - ```ip netns exec r1 ip link set veth1.1 name eth0```
  - ```ip netns exec r2 ip link set veth1.2 name eth0```
  ```
  [root@server162 ~]# ip netns exec r1 ip link set veth1.1 name eth0
  [root@server162 ~]# ip netns exec r1 ifconfig -a
  eth0: flags=4098<BROADCAST,MULTICAST>  mtu 1500
          ether 5a:6d:52:eb:69:d3  txqueuelen 1000  (Ethernet)
  ```
- 分别设置两个名称空间中eth0的ip进行通信
  ```
  [root@server162 ~]# ip netns exec r1 ifconfig eth0 10.10.10.1
  [root@server162 ~]# ip netns exec r2 ifconfig eth0 10.10.10.2
  [root@server162 ~]# ip netns exec r1 ping 10.10.10.2
  PING 10.10.10.2 (10.10.10.2) 56(84) bytes of data.
  64 bytes from 10.10.10.2: icmp_seq=1 ttl=64 time=0.738 ms
  64 bytes from 10.10.10.2: icmp_seq=2 ttl=64 time=0.072 ms

  [root@server162 ~]# ip netns exec r2 ping 10.10.10.1
  PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
  64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.097 ms
  64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.071 ms
  ```
# 5. 复杂虚拟机网路实现（net namespace）

## 5.1 create bridge br-ex, br-in
- br-ex: attach physical interface to br-ex
- br-in: attach all VM backend tap interface to br-in

```
[root@server162 ~]# brctl addbr br-ex
[root@server162 ~]# brctl addbr br-in

[root@server162 ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
br-ex           8000.000000000000       no
br-in           8000.000000000000       no
```
- ```[root@server162 ~]# ip link set br-ex up```
- ```[root@server162 ~]# ip link set br-in up```

## 5.2 create VMs
- create /etc/qemu-ifup
  ```bash
  #!/bin/bash
  bridge=br-in
  if [ -n '$1' ]; then
          ip link set $1 up
          brctl addif $bridge $1
          [ $? -eq 0 ] && exit 0 || exit 1
  else
          echo "Error: no interface specified."
          exit 1
  fi
  ```
- ```chmod +x /etc/qemu-ifup```
- ```bash -n !$```, check syntax
- ```ln -sv /usr/libexec/qemu-kvm /usr/bin/```
- create VMs
  - ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-x86_64-disk.img,media=disk,if=virtio -net nic,macaddr=52:54:00:11:22:33 -net tap,ifname=vf1.0,script=/etc/qemu-ifup --nographic```
  - ```qemu-kvm -m 128 -smp 1 -name cirros2 -drive file=/images/cirros/cirros-0.3.4-x86_64-disk-copy.img,media=disk,if=virtio -net nic,macaddr=52:54:00:11:22:44 -net tap,ifname=vf2.0,script=/etc/qemu-ifup --nographic```
- brctl show
  ```
  [root@server162 ~]# brctl show
  bridge name     bridge id               STP enabled     interfaces
  br-ex           8000.000c295e80e5       no              ens33
  br-in           8000.6e18297c0a83       no              vf1.0
                                                          vf2.0
  ```
### 2. create virtual router(netns)
- ```if netns add r1```

### 3. 创建一对网卡，一个连接虚拟机br-in，一个连接路由器r1
- 创建路由器内网网卡,router in ruuter, router in switch
  - ```ip link add rinr type veth peer name rins```
  ```
  [root@server162 ~]# ip link show
  28: rins@rinr: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 6a:4d:20:b7:d6:9a brd ff:ff:ff:ff:ff:ff
  29: rinr@rins: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether ca:d6:74:ad:69:1e brd ff:ff:ff:ff:ff:ff
  ```
- ```ip link set rinr up```
- ```ip link set rins up```







### 2. attach phsical if to br-ex
all commands here is temporary, create ifcfg can make it permanent
- ```ip addr del 192.168.0.162/16 dev ens33; ip addr add 192.168.0.162/16 dev br-ex; brctl addif br-ex ens33 ```
  ```
  br-ex: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 192.168.0.162  netmask 255.255.0.0  broadcast 0.0.0.0
  
  ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet6 fe80::20c:29ff:fe5e:80e5  prefixlen 64  scopeid 0x20<link>
          ether 00:0c:29:5e:80:e5  txqueuelen 1000  (Ethernet)
  ```
  
### 3. create peer interfaces, one attach to br-in, one attach to router(netns)

#### i. 打开网络转发功能
```
[root@server162 ~]# vim /etc/sysctl.conf 
net.ipv4.ip_forward = 1

[root@server162 ~]# sysctl -p
net.ipv4.ip_forward = 1
```
#### ii. 创建一对网卡
- ```ip link add veth1.1 type veth peer name veth1.2```





















































-----

-----

- 使用网络安装boot order=n，因为没有pxe网络或者cobbler，所以安装会失败，需要指定安装镜像iso
  ```
  qemu-kvm -m 512 -smp 2 -name centos6 -drive file=/images/centos/centos6.qcow2,media=disk -net nic,macaddr=52:54:00:11:22:33 -net tap,ifname=centos6.0,script=/etc/qemu-ifup -boot order=nc,once=n -nographic
  ```
- 写了两个drive file，指定虚拟硬盘qcow2，指定安装镜像iso。
  ```
  qemu-kvm -m 512 -smp 2 -name centos -drive file=/images/centos/centos6.qcow2,format=qcow2,media=disk,if=virtio -drive file=/windowsshare/CentOS-7-x86_64-DVD-1810.iso,media=cdrom  -net nic,model=virtio -net tap,ifname=centos6.0 -boot order=dc,once=d
  ```
  ```
  qemu-kvm -m 512 -smp 2 -name xp -drive file=/images/win/xp.qcow2,format=qcow2,media=disk,if=virtio -drive file=/windowsshare/WindowsXPSP3.iso,media=cdrom  -net nic,model=virtio,macaddr=52:54:00:12:12:12  -net tap,ifname=xp1.0 -boot order=dc,once=d
  ```
-----

# qemu-kvm 网络
1. 虚拟机网络类型
2. KVM虚拟机网络基础
3. 实现各网络模型的KVM虚拟机
4. 复杂网络实现（netns网络名称空间）




# 1. 虚拟机网络类型
## 1.1 隔离模型
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
- linux dnsmasq 既可以提供dhcp服务，又可以提供给dns服务
  - 安装qemu-kvm时，会自动安装上dnsmasq
    ```
    [root@server15 ~]# ps -aux | grep dns
    nobody 10020  0.0  0.0  53884  1112 ?  S    22:04   0:00 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
    root   10021  0.0  0.0  53856   376 ?  S    22:04   0:00 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
    ```
- 创建kvm虚拟机时，要创建网卡的前半段和后半段，并需要脚本连接到相应的虚拟网桥上面，主机脚本需要手动写/etc/qemu-ifup

## 1.2 路由模型
- ~~在 隔离模型 的基础上，给网桥上面再增加一个接口virnet1，或者说是在物理机上再创建一个虚拟网卡并关联到网桥~~ 直接给网桥配置也虚拟机相同网段的IP地址，并且物理机开启核心转发功能ip_forward=1，```/proc/sys/net/ipv4/ip_forward = 1```
- ~~将新端口virnet1连至物理机网卡，~~ 虚拟机可以访问外网，但是外网无法回访虚拟机
  ![](https://i.loli.net/2019/04/13/5cb1b435428cd.png)
- 物理机上在创建一个虚拟网卡，一端连接物理网卡，一端连接需要虚拟网桥，来使虚拟机连接外网
- 虚拟机的网卡网关指向物理机虚拟网卡的ip地址
- 缺点依然是，路由模型无法让外网回访虚拟机
- 所以vmware中根本没有这种模型

## 1.3 NAT模型
- 在上述 路由模型 的基础上，添加一个NAT server（添加一条路由规则即可），即可实现外网可以访问虚拟机
- 回访时，物理网卡将数据发给NAT server，由NAT server决定发给哪一个虚拟机
  ![](https://i.loli.net/2019/04/13/5cb1b5ed66a98.png)
- 在物理机上开启NAT功能，NAT会话表可以知道将报文发送给哪一个虚拟机
- linux中的NAT服务就是一条iptables规则，
  - ```iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -j SNAT --to-source 192.168.0.162```
- 虚拟机可以ping通外网，但是外网不能直接ping通虚拟机，即虚拟机不能作为目标地址来通信。因为NAT使用的都是私有地址
  - **在物理网卡上面配置多个IP地址，每个ip地址对应一个虚拟机。即目标地址转发**
  - SDN，软件定义网络，来实现服务网络实现
  - openswitch

## 1.4 桥接模型
- 在 隔离模型 的基础上，直接将网桥设备关联至物理网卡，可以通过物理网卡访问外网
- 问题是，虚拟机和物理机都通过同一个物理网卡访问外网，回访时，应该找物理机还是虚拟机？
- 此时，物理网卡会变成一个交换机（物理网桥）。给物理机创建一个虚拟网卡，并将物理网卡自己的MAC地址给这个虚拟网卡
- 在物理网卡上打开 混杂模式，即不管mac地址是谁，物理网卡都将其信息接收下来。接下来后，再根据MAC地址（物理机mac还是虚拟机mac）来判断信息发送给谁


# 2. KVM虚拟机网络基础

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
- ```ifconfig br0 up```，或使用下面命令
```
[root@server15 ~]# ip link set dev br0 up	# ip link set br0 up
[root@server15 ~]# ip link show 		# 显示up状态	

6: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether d6:c4:00:a1:84:1e brd ff:ff:ff:ff:ff:ff
```
- 要想永久有效，需要创建配置文件ifcfg-br0

## 2.2. qemu-kvm网络属性相关选项
- 添加虚拟网卡，并将虚拟机接入网络。三种网络属性：nic前半段，tap后半段，user
- nic
- tap，是二层设备，通信靠MAC地址
  - tap，需要在创建虚拟机时注意网卡mac不能一样，默认mac地址都是一样的，需要手动指定，否则互相不能通信
  - 另一种tun三层设备，通信靠IP地址
  
### 2.2.1. net nic，向虚拟机创建一个网络设备
- ```qemu-kvm -net nic[,vlan=n][,macaddr=mac][,model=type][,name=name][,addr=addr][,vectors=v]```
  - macaddr，需要手动指定虚拟机内前半段网卡的mac地址，否则多个虚拟机都使用默认的mac，导致mac冲突，无法网络通信
  - qemu可以模拟多种网卡设备，默认为intel的千兆以太网网络控制器e1000类型，一般只会用来更改为半虚拟化virtio
    - ```qemu-kvm -net nic,model=?```
      ```
      qemu: Supported NIC models: ne2k_pci,i82551,i82557b,i82559er,rtl8139,e1000,pcnet,virtio
      ```
### 2.2.2. net tap, 创建网卡后半段
- ```qemu-kvm -net tap[,vlan=n][,name=name][,fd=h][,ifname=name][,script=file][,downscript=dfile]```
- 可以通过物理机的TAP网络接口连接至指定vlan n
- 也可以使用script=file，来指定网卡后半段连接至某一个网桥，脚本默认路径/etc/qemu-ifup，没有脚本的话script=no
- 通过downscript，在虚拟机关机时，将网卡后半段与网桥分离，默认路径为/etc/qemu-ifdown，没有脚本的话downscript=no
- ifname指定后半段在物理机上叫什么名字，默认为tap0
#### 1. qemu-ifup
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
**```chmod +x /etc/qemu-ifup```, 一定要将脚本添加执行权限，否则报错 **
#### 2. qemu-ifdown
- 其实关机脚本qemu-ifdown可以不用指定，会自动执行
- 当虚拟机关机后，其使用的后半段网卡会自动消失
- 通过brctl show查询绑定在br0的端口也会消失

# 3. 实现各网络模型的KVM虚拟机

- 参照上面步骤创建网桥br0
- 创建kvm虚拟机
```
qemu-kvm -m 128 -cpu host -smp 2 -name test -drive file=cirros-0.3.4-x86_64-disk.img,\
if=virtio,media=disk,format=qcow2,cache=writeback -nographic -net nic \
-net tap,ifname=vif0.0,script=/etc/qemu-ifup
```
```
qemu-kvm -m 128 -cpu host -smp 2 -name test -drive file=cirros-0.3.4-x86_64-disk.img,if=virtio,media=disk,format=qcow2,cache=writeback -nographic -net nic -net tap,ifname=vif0.0,script=/etc/qemu-ifup
```
> - Issue: 创建时could not configure /dev/net/tun (vif0.0): Device or resource busy
> - Issue: 创建时qemu-ifup: could not configure /dev/net/tun: Operation not permitted
>   - 使用**管理员**账户；将上面的命令复制到同一行内，不要用断行符号\，命令可以执行成功
> - Issue:  登录虚拟机后EXT4-fs error (device vda1): ext4_lookup:1584: inode #6150: comm sh: deleted inode referenced: 6275
>   - 使用cirros-0.4.0全是错，改用cirros-0.3.4-x86_64-disk.img版本
> - Issue: Nothing to boot: No such file or directory (http://ipxe.org/2d03e13b) No more network devices
>   - 执行上面创建命令，我发启动ipxe，找不到问题所在。还原快照，重新安装qemu-kvm，解决
> - Issue: qemu-kvm: -drive file=cirros-0.3.4-x86_64-disk.img,: drive with bus=0, unit=0 (index=0) exists”
>   - 多数原因是参数前面少了 -，还是复制到同一行来执行命令


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
#### 1. 创建第二个虚拟机
- 上面创建了test虚拟机
- 再次创建test1虚拟机，使用相同script，将网卡后半段绑定至相同网桥设备br0。需要手动指定虚拟机内前半段网卡的mac地址
  ```diff
    qemu-kvm -m 128 -cpu host -smp 2 -name test1 -drive file=cirros-0.3.4-x86_64-disk.img,\
  - if=virtio,media=disk,format=qcow2,cache=writeback -nographic -net nic \
  + if=virtio,media=disk,format=qcow2,cache=writeback -nographic -net nic,macaddr=52:54:00:12:34:57 \
    -net tap,ifname=vif1.0,script=/etc/qemu-ifup
  ```
  ```
  qemu-kvm -m 128 -cpu host -smp 2 -name test1 -drive file=cirros-0.3.4-x86_64-disk.img,\
  if=virtio,media=disk,format=qcow2,cache=writeback -nographic -net nic,macaddr=52:54:00:12:34:57 \
  -net tap,ifname=vif1.0,script=/etc/qemu-ifup
  ```
- 查看网桥br0端口
  ```
  bridge name	bridge id		STP enabled	interfaces
  br0		8000.5ee803a9cbdb	no		vif0.0
  						       	vif1.0
  ```
#### 2. 两个虚拟机配置ip地址
- test
  ```
  $ ifconfig eth0 192.168.0.1 up
  $ ifconfig 
  eth0      Link encap:Ethernet  HWaddr 52:54:00:12:34:56  
            inet addr:192.168.0.1  Bcast:10.255.255.255  Mask:255.0.0.0

  ```
- test1
  ```
  $ ifconfig eth0 192.168.0.2 up
  $ ifconfig 
  eth0      Link encap:Ethernet  HWaddr 52:54:00:12:34:56  
            inet addr:192.168.0.2  Bcast:10.255.255.255  Mask:255.0.0.0
  
  $ ping 192.168.0.1
  PING 192.168.0.1 (10.0.0.1): 56 data bytes
  
  --- 10.0.0.1 ping statistics ---
  4 packets transmitted, 0 packets received, 100% packet loss
  ```
- ping不通，查看两个虚拟机的网卡MAC地址一样。所以创建虚拟机时，网卡mac要手动指定
  ```
  qemu-kvm -m 128 -cpu host -smp 2 -name test1 -drive file=cirros-0.3.4-x86_64-disk.img,\
  if=virtio,media=disk,format=qcow2,cache=writeback -nographic -net nic,macaddr=52:54:00:12:34:57 \
  -net tap,ifname=vif1.0,script=/etc/qemu-ifup
  ```
- 再次创建test1虚拟机，配置ip地址，虚拟机指向相互ping通
  ```
  
  $ ifconfig eth0 192.168.0.2 up
  $ ifconfig 
  eth0      Link encap:Ethernet  HWaddr 52:54:00:12:34:56  
            inet addr:192.168.0.2  Bcast:10.255.255.255  Mask:255.0.0.0

  $ ping 192.168.0.1
  PING 192.168.0.1 (10.0.0.1): 56 data bytes
  64 bytes from 192.168.0.1: seq=0 ttl=64 time=2.488 ms
  64 bytes from 192.168.0.1: seq=1 ttl=64 time=1.916 ms
  
  --- 192.168.0.1 ping statistics ---
  2 packets transmitted, 2 packets received, 0% packet loss
  round-trip min/avg/max = 1.916/2.202/2.488 ms
  ```

## 3.2 组建NAT网络

### 3.1 物理机 -> 虚拟机 
配置物理机可以ping通虚拟机
```
[root@server15 ~]# ifconfig br0 192.168.0.254 
[root@server15 ~]# ifconfig 
br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.254  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::d4c4:ff:fea1:841e  prefixlen 64  scopeid 0x20<link>
        ether a2:e4:0d:02:90:3d  txqueuelen 1000  (Ethernet)
        RX packets 361  bytes 30192 (29.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 529  bytes 49456 (48.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
此时物理机可以ping通虚拟机192.168.0.0网段
### 3.2 虚拟机 < - >物理机 相互通信
- 在虚拟机test中测试ping网桥br0，ok
  ```
  $ ping 192.168.0.254 
  PING 10.0.0.254 (10.0.0.254): 56 data bytes
  64 bytes from 192.168.0.254: seq=0 ttl=64 time=6.578 ms
  64 bytes from 192.168.0.254: seq=1 ttl=64 time=1.527 ms
  64 bytes from 192.168.0.254: seq=2 ttl=64 time=1.551 ms
  
  --- 192.168.0.254 ping statistics ---
  3 packets transmitted, 3 packets received, 0% packet loss
  round-trip min/avg/max = 1.527/3.218/6.578 ms
  ```
- 在虚拟机test中测试ping网桥物理机，failed. 需要在虚拟机中加一条路由规则
  ```
  $ ping 172.30.1.34
  PING 172.30.1.34 (172.30.1.34): 56 data bytes
  ping: sendto: Network is unreachable
  ```
  - 将虚拟机网关指向br0，ping物理机ok
  ```
  $ route add default gw 192.168.0.254
  
  $ ping 172.30.1.34
  PING 172.30.1.34 (172.30.1.34): 56 data bytes
  64 bytes from 172.30.1.34: seq=0 ttl=64 time=0.906 ms
  64 bytes from 172.30.1.34: seq=1 ttl=64 time=1.621 ms
  64 bytes from 172.30.1.34: seq=2 ttl=64 time=1.807 ms
  
  --- 172.30.1.34 ping statistics ---
  3 packets transmitted, 3 packets received, 0% packet loss
  round-trip min/avg/max = 0.906/1.444/1.807 ms
  ```
### 3.2 虚拟机 < - >外网 相互通信
- 目前只是虚拟机和通物理机ip互通而已。但是ping物理网卡的网关172.30.1.1失败，ping任何一个172.30.1.0网段的ip都不通，除了本机ip
- 是因为没有路由转发，报文发出去了，但是回不来。打开物理机路由转发功能（核心转发），也还是ping不出去
  ```
  [root@server15 ~]# cat /proc/sys/net/ipv4/ip_forward     # 如果是0，需要手动置1
  1
  ```
- 此时如果要访问外网，可以添加路由，或者添加NAT。因为物理网络的路由不能随意添加，所以还是选择NAT模式
  - ```[root@server15 ~]# iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source 172.30.1.34```

  ```
  [root@server15 ~]# iptables -t nat -L -n
  Chain PREROUTING (policy ACCEPT)
  target     prot opt source               destination         
  
  Chain INPUT (policy ACCEPT)
  target     prot opt source               destination         
  
  Chain OUTPUT (policy ACCEPT)
  target     prot opt source               destination         
  
  Chain POSTROUTING (policy ACCEPT)
  target     prot opt source               destination          
  SNAT       all  --  10.0.0.0/24          0.0.0.0/0            to:172.30.1.34     # 可能是/24的问题，配置这个网段ping不通外网
  SNAT       all  --  192.168.0.0/24       0.0.0.0/0            to:172.30.1.34
  ```
- 虚拟机中ping物理网关172.30.1.1还是失败，有可能是咖啡厅的路由器设置了不允许ping网关
- ping物理网段中的其他ip正常，ping220.181.57.216（baidu）也正常。因此设置与外网通信成功
  ```
  $ ping 220.181.57.216
  PING 220.181.57.216 (220.181.57.216): 56 data bytes
  64 bytes from 220.181.57.216: seq=0 ttl=47 time=124.110 ms
  64 bytes from 220.181.57.216: seq=1 ttl=47 time=75.594 ms
  
  --- 220.181.57.216 ping statistics ---
  2 packets transmitted, 2 packets received, 0% packet loss
  round-trip min/avg/max = 75.594/99.852/124.110 ms
  ```

## 3.3 组建桥接网络

物理网卡直接添加到虚拟网桥br0，此时物理网卡就成了交换机，将原来物理网卡的ip地址给br0，物理网卡不设置ip地址。启动交换机的混在模式

#### 1. 物理机网络设置
- 拆掉br0的ip地址
  ```ip addr del 192.168.0.254/24 dev br0```

- 拆掉物理网卡ip，将ip给br0，绑定物理网卡到br0
  ```
  ip addr del 172.30.1.34 dev ens33; brctl addif br0 ens33; ip addr add 172.30.1.34/16 dev br0
  ```
  - 会报错，但是ip地址可以配置成功
  - 注意ip addr add/del中的参数dev 一定要有
  - 注意添加br0的ip地址时/16 一定要有，否则无法平通通网段其他ip
  ```
  [root@server15 ~]# brctl show
  bridge name	bridge id		STP enabled	interfaces
  br0		8000.000c2954fad5	no		ens33
  							vif0.0
  							vif1.0
  ```
  ```
  [root@server15 ~]# ifconfig br0
  br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 172.30.1.34  netmask 255.255.255.255  broadcast 0.0.0.0
          inet6 fe80::a4b5:cbff:fead:cc20  prefixlen 64  scopeid 0x20<link>
          ether 00:0c:29:54:fa:d5  txqueuelen 1000  (Ethernet)
          RX packets 2580  bytes 269720 (263.3 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 324  bytes 40104 (39.1 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  ```
  **物理机外网ping不通！！！？？？ping百度失败。通过配置文件的方式，完全复制物理网卡的设定给br0试一下？估计是16，24这样的位数的原因**
  > [把 ens33 绑到 br0 桥设备上](https://github.com/davidkorea/linux_study/blob/master/5_linux_virtualization/01_KVM_build.md#2-%E6%8A%8A-ens33-%E7%BB%91%E5%88%B0-br0-%E6%A1%A5%E8%AE%BE%E5%A4%87%E4%B8%8A)，使用配置文件的方式，完整配置桥接网络
  
#### 2. 虚拟机网络设置
```
$ ifconfig eth0 172.30,1.51					# 直接配置新ip失败
ifconfig: bad address '172.30,1.51'
$ 
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast qlen 1000
    link/ether 52:54:00:12:34:56 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.1/24 brd 192.168.0.255 scope global eth0
    inet6 fe80::5054:ff:fe12:3456/64 scope link 
       valid_lft forever preferred_lft forever
$ ip addr del 192.168.0.1/24 dev eth0				# 先拆除就ip
$ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 52:54:00:12:34:56  
          inet6 addr: fe80::5054:ff:fe12:3456/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:853 errors:0 dropped:0 overruns:0 frame:0
          TX packets:219 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:69760 (68.1 KiB)  TX bytes:20410 (19.9 KiB)

$ ifconfig eth0 172.30.1.51					# 设置新ip
$ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 52:54:00:12:34:56  
          inet addr:172.30.1.51  Bcast:172.30.255.255  Mask:255.255.0.0
          inet6 addr: fe80::5054:ff:fe12:3456/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:859 errors:0 dropped:0 overruns:0 frame:0
          TX packets:219 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:70120 (68.4 KiB)  TX bytes:20410 (19.9 KiB)

$ ping 172.30.1.50						# 可以ping通物理网络
PING 172.30.1.50 (172.30.1.50): 56 data bytes
64 bytes from 172.30.1.50: seq=0 ttl=64 time=1581.620 ms
64 bytes from 172.30.1.50: seq=1 ttl=64 time=585.848 ms
64 bytes from 172.30.1.50: seq=2 ttl=64 time=289.564 ms
64 bytes from 172.30.1.50: seq=3 ttl=64 time=174.459 ms

--- 172.30.1.50 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 174.459/657.872/1581.620 ms
```
- 此时虚拟机可以使用物理网络中的地址与物理网络通信


































