# KVM comprehensive network based on Open vSwitch
# 2. GRE (2 Hosts)
- 2 physical node different network
- VMs on that 2 nodes can communicate
- GRE in OSI layer3，是一种隧道技术，使用一种报文来承载和传输另一种报文，如以太网帧
- 为简化，将2个物理节点放在同一网络，虽然不同网络可以实现GRE，但是需要再配置路由
  - 当然，因为是再同一个网段，可以直接将物理网卡绑定至br-in桥上面，实现通信
  - 这里，演示GRE的方式进行连接
  - 如果两个物理节点不在同一个网段，就只能使用GRE才能实现通信（也需要路由器使得两个网段可以通信）
- 删除上一步创建的所有虚拟机，只留下br-in桥，删除其他桥和接口

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

## 2.2 Node 2

- Node2不指定DHCP，连接GRE后，使用Node1的 DHCP服务器
- ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-1.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:01:01 -net tap,ifname=vif0.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown -daemonize```, 注意更改mac地址




























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
- 网卡1：仅主机，用于传送控制命令
- 网卡2：VMnet，用于node之间的通信，内网
  
## 3. 创建1个控制节点
- 网卡1：桥接物理网络，连接外网
- 网卡2：VMnet，作为node节点的网关
- SNAT 网卡1和2，将所有内部网转到外网，实现节点与外网通信
