# KVM_libvirt_network
# 1. Virtual Network
## 1.1 basic
- **TUN**, which stands for "tunnel", simulates a network layer device and it operates at OSI reference model's layer 3 packets, such as IP packets. 
- **TAP**(namely a network tap) simulates a link layer device and it operates at OSI reference model's layer 2 packets, such as Ethernet frames. 
- TUN is used with routing, while TAP is used to create a network bridge.
#### 1. create a bridge
- Make sure the bridge module is loaded into the kernel
  - ```lsmod | grep bridge```
- create a bridge
```
[root@server162 ~]# brctl addbr tester-bridge
[root@server162 ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
tester-bridge   8000.000000000000       no

[root@server162 ~]# ip link show tester-bridge 
15: tester-bridge: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 0a:b8:94:9d:e3:01 brd ff:ff:ff:ff:ff:ff
```
#### 2.  create and add a TAP(osi layer 2) device to the bridge
- First check if the TUN/TAP device module is loaded into the kernel
  - ```lsmod | greptun```
- create a tap device named vm-vnic
```
[root@server162 ~]# ip tuntap add dev vm-vnic mode tap
[root@server162 ~]# ip link show vm-vnic 
16: vm-vnic: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 1e:df:bf:b9:5b:9a brd ff:ff:ff:ff:ff:ff
```
- add  tap device ```vm-vnic``` to bridge ```tester-bridge```
```
[root@server162 ~]# brctl addif tester-bridge vm-vnic
[root@server162 ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
tester-bridge   8000.1edfbfb95b9a       no              vm-vnic
```

> You can see that vm-vnic is an interface added to the bridge tester. Now vm-vnic can act as the interface between your virtual machine and the bridge tester, which in turn enables the virtual machine to communicate with other virtual machines added to this bridge
> 
> ![](https://i.loli.net/2019/04/22/5cbd7c98dc7d6.png)

#### 3. remove if and br
- Remove the vm-vnic tap device from the tester bridge，解绑tap设备
  - ```brctl delif tester-bridge vm-vnic```
- remove the tap device using the ip command，删除tap设备
  - ```ip tuntap del vm-vnic mode tap```
- ```brctl delbr tester-bridge```,
## 1.2 Virtual networking using libvirt

- Isolated virtual network
- Routed virtual network
- NATed virtual network
- Bridged network using a physical NIC, VLAN interface, bond interface, and bonded VLAN interface
- MacVTap
- PCI passthrough NPIV
- OVS

we will cover each type of networks below.

# 2 Isolated virtual network
a closed network for the virtual machines. only the virtual machines which are added to this network can communicate with each other

![](https://i.loli.net/2019/04/22/5cbd807f1c416.png)

## 2.1 virt-manager
![](https://i.loli.net/2019/04/22/5cbd880d58486.png)
## 2.2 virsh with xml
- delete setting above on virt-manager first
- create the isolated network using the virsh command. For that, we need to create an XML fle with the following contents and save it as isolated.xml
### 1. create isolated.xml
create the isolated network using the virsh command. For that, we need to create an XML fle with the following contents and save it as isolated.xml
```xml
[root@server162 ~]# vim isolated.xml 
<network>
  <name>isolated</name>
</network>
```
### 2. defne a network using the XML flie created above
```
[root@server162 ~]# virsh net-define isolated.xml 
从 isolated定义网络isolated.xml
```
```
[root@server162 ~]# virsh net-list --all
 名称               状态     自动开始  持久
----------------------------------------------------------
 default            活动      是        是
 isolated           不活跃    否           
 ```
### 3. xml file that virst created based on isolated.xml we provided 
Let's see the XML fle libvirt being created based on the confguration we provided through the isolated.xml
 
```xml
[root@server162 ~]# virsh net-dumpxml isolated 
<network>
  <name>isolated</name>
  <uuid>be4e3e53-a36e-4779-b772-0ed835eef8b4</uuid>
  <bridge name='virbr1' stp='on' delay='0'/>
  <mac address='52:54:00:74:dc:e7'/>
</network>
```
- libvirt added a few additional parameters. and created a new bridge for this network, brctl show cannot find it because it has nt been activated.
- libvirt added the rest of the required parameters; you can mention these in your XML fle when required. recommendation is that you leave it to libvirt to avoid conﬂicts.
- ```net-create``` is similar to ```net-define```. 
  - ```net-create``` will not create a persistent virtual network. Once destroyed, it is removed and has to be created again using the net-create command.
 - recommand ```net-define```

Once you defne a network using net-define, the confguration fle will be stored in ```/etc/libvirt/qemu/networks/``` as an XML fle with the same name as your virtual network
```xml
<!--
WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
OVERWRITTEN AND LOST. Changes to this xml configuration should be made using:
  virsh net-edit isolated
or other application using the libvirt API.
-->

<network>
  <name>isolated</name>
  <uuid>be4e3e53-a36e-4779-b772-0ed835eef8b4</uuid>
  <bridge name='virbr1' stp='on' delay='0'/>
  <mac address='52:54:00:74:dc:e7'/>
</network>
```
### 4. activate network
- virsh net-start
```
[root@server162 networks]# virsh net-start isolated 
网络 isolated 已开
```
- virsh net-autostart
```
[root@server162 ~]# virsh net-autostart isolated 
网络isolated标记为自动启
```
```
[root@server162 ~]# virsh net-list --all
 名称               状态     自动开始  持久
----------------------------------------------------------
 default              活动     是           是
 isolated             活动     是           
 ```
### 5. add a network interface to vm

create 2 VMs as [007_KVM_libvirt_basic - virt-install](https://github.com/davidkorea/linux_study/blob/master/5_linux_virtualization/007_KVM_libvirt_basic.md#2-virt-install)

#### i. vm1 by virt-manager
![](https://i.loli.net/2019/04/23/5cbe7c0af14ea.png)

```
[root@server162 ~]# virsh domiflist centos7-raw-clone
接口     类型     源        型号      MAC
-------------------------------------------------------
vnet0      bridge     virbr0     virtio      52:54:00:ab:30:f2
vnet2      network    isolated   virtio      52:54:00:99:e4:1f
```
  - Interface - Name of the tap interface attached to the bridge.
  - Type - Type of device
  - Source - Name of the virtual network.
  - Model - Virtual NIC model.
  - MAC - MAC address of the virtual NIC(not the MAC of vnet0 or vnet2) .
#### ii. vm2 by virsh
- get the details of the current virtual NIC attached to the virtual machine centos7-raw-clone1 using domiflist
```
[root@server162 ~]# virsh domiflist centos7-raw-clone1
接口     类型     源        型号      MAC
-------------------------------------------------------
vnet1      bridge     virbr0     virtio      52:54:00:3b:22:59
```
- attach a new virtual interface to centos7-raw-clone1
```
[root@server162 ~]# virsh attach-interface --domain centos7-raw-clone1 --source isolated --type network --model virtio --config --live 
成功附加接
```
  - ```--config```: make the change persistent in the next startup of the VM.
  - ```--live```: attach the NIC to a live virtual machine. Remove --live if the virtual machine is not running
  
```
[root@server162 ~]# virsh domiflist centos7-raw-clone1
接口     类型     源        型号      MAC
-------------------------------------------------------
vnet1      bridge     virbr0     virtio      52:54:00:3b:22:59
vnet3      network    isolated   virtio      52:54:00:2d:56:9c
```
#### iii. check bridge 
```
[root@server162 ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.52540099402f       yes             virbr0-nic
                                                        vnet0
                                                        vnet1
virbr1          8000.52540074dce7       yes             virbr1-nic
                                                        vnet2
                                                        vnet3
```
- interface vnet0 and vnet1 is created when the vm is first created. they are attached to virbr0 by default
- interface vnet2 and vnet3 is we created and attached to isolated network
- The ```virbr1-nic``` interface is created by libvirt when it starts virbr1
  - The purpose of this interface is to provide a consistent and reliable MAC address for the virbr1 bridge
  - The bridge copies the MAC address of the frst interface, which is added to it
  - virbr1-nic is always the frst interface added to it by libvirt and never being removed till the bridge is destroyed

#### iv. set ip for each vm
because of minimal install, no ifconfig command. refer to [006_KVM_qemu_network](https://github.com/davidkorea/linux_study/blob/master/5_linux_virtualization/006_KVM_qemu_network.md#1-%E7%89%A9%E7%90%86%E6%9C%BA%E7%BD%91%E7%BB%9C%E8%AE%BE%E7%BD%AE) for ```ip``` related command usage. 
- vm1
```ip addr add 192.168.10.1/16 dev eth1```, /16 is a must

- vm2
```ip addr add 192.168.10.2/16 dev eth1```, /16 is a must
- ```ip addr```, to check ip address if no ifconfig command in minimal install
![](https://i.loli.net/2019/04/23/5cbe86bb15787.png)

#### v, detach-interface
```
virsh detach-interface --domain F22-02 --type network --mac
52:54:00:2b:0d:0c --config --live
Interface detached successfully
```

# 3. Routed virtual network
This mode is not commonly used, unless you have a special use case to create a network with this complexity.

![](https://i.loli.net/2019/04/23/5cbe8b42b681f.png)

## 3.1 virt-manager

![](https://i.loli.net/2019/04/23/5cbea35c28fbb.png)

## 3.2 virsh

- delete the routed network created above

### 1. net-define with xml file
- Create an XML confguration file as routed.xml
```xml
<network>
  <name>routed</name>
  <forward dev='ens33' mode='route'>
    <interface dev='ens33' />
  </forward>
  <ip address='192.168.10.1' netmask='255.255.255.0'></ip>
</network>
```
- net-define
```
[root@server162 ~]# virsh net-define routed.xml 
从 routed定义网络routed.xml
```
```
[root@server162 ~]# virsh net-list --all
 名称               状态     自动开始  持久
----------------------------------------------------------
 default              活动     是           是
 isolated             活动     是           是
 routed               不活跃  否           是
```
```xml
[root@server162 ~]# virsh net-dumpxml routed 
<network>
  <name>routed</name>
  <uuid>2c316996-204d-423d-9f73-2f095e3fed5b</uuid>
  <forward dev='ens33' mode='route'>
    <interface dev='ens33'/>
  </forward>
  <bridge name='virbr2' stp='on' delay='0'/>
  <mac address='52:54:00:75:57:2c'/>
  <ip address='192.168.10.1' netmask='255.255.255.0'>
  </ip>
</network>
```
### 2. activate routed network
```
[root@server162 ~]# virsh net-start routed 
网络 routed 已开始

[root@server162 ~]# virsh net-autostart routed 
网络routed标记为自动启动

[root@server162 ~]# virsh net-list --all
 名称               状态     自动开始  持久
----------------------------------------------------------
 default              活动     是           是
 isolated             活动     是           是
 routed               活动     是           是
```
### 3. attach an interfate to routed network 
```
virsh attach-interface --domain centos7-raw-clone   
--source routed --type network --model virtio --config --live 
```
```
[root@server162 ~]# virsh domiflist 1
接口     类型     源        型号      MAC
-------------------------------------------------------
vnet0      bridge     virbr0     virtio      52:54:00:ab:30:f2
vnet1      network    isolated   virtio      52:54:00:99:e4:1f

[root@server162 ~]# virsh attach-interface --domain centos7-raw-clone --source routed --type network --model virtio --config --live 
成功附加接口

[root@server162 ~]# virsh domiflist 1
接口     类型     源        型号      MAC
-------------------------------------------------------
vnet0      bridge     virbr0     virtio      52:54:00:ab:30:f2
vnet1      network    isolated   virtio      52:54:00:99:e4:1f
vnet4      network    routed     virtio      52:54:00:6a:bc:1c
```
delete if
```
[root@server162 ~]# virsh detach-interface --domain centos7-raw-clone --type network --mac 52:54:00:6a:bc:1c --config --live 
成功分离接
```

### 4. create routing rules

donno how to do...

## 3.3 edit a routed virtual network
- edit a routed virtual network and modify the routing confguration so that the packets from the virtual machines can be forwarded to any interface available on the host based on IP route rules specifed on the host
- The aim of this example is to show how to modify a virtual network once it is created with your confguration

### 1. stop the virtual network frst
net-destory is only de-activate the network, not delete.
```
[root@server162 ~]# virsh net-destroy routed 
网络 routed 被删除

[root@server162 ~]# virsh net-list --all
 名称               状态     自动开始  持久
----------------------------------------------------------
 default              活动     是           是
 isolated             活动     是           是
 routed               不活跃  是           
```
### 2. net-edit
```diff
  <network>
    <name>routed</name>
    <uuid>2c316996-204d-423d-9f73-2f095e3fed5b</uuid>
-   <forward dev='ens33' mode='route'>
-     <interface dev='ens33'/>
-   </forward>
+   <forward mode='route' />

    <bridge name='virbr2' stp='on' delay='0'/>
    <mac address='52:54:00:75:57:2c'/>
    <ip address='192.168.10.1' netmask='255.255.255.0'>
    </ip>
  </network>
```
virify the changes with net-dump
```xml
[root@server162 ~]# virsh net-dumpxml routed 
<network>
  <name>routed</name>
  <uuid>2c316996-204d-423d-9f73-2f095e3fed5b</uuid>
  <forward mode='route'/>
  <bridge name='virbr2' stp='on' delay='0'/>
  <mac address='52:54:00:75:57:2c'/>
  <ip address='192.168.10.1' netmask='255.255.255.0'>
  </ip>
</network>
```
To enable IPv6, you can add a similar to the preceding confguration. 
```xml
<ip family="ipv6" address="2001:db8:ca2:2::1" prefix="64" >
```
### 3. activate routed network
```
[root@server162 ~]# virsh net-start routed 
网络 routed 已开
```

# 4. NATed virtual network
- NATed mode is the most commonly used virtual networking when you want to set up a test environment on your laptop or test machine. 
- This mode allows the virtual machines to communicate with the outside network without using any additional confguration. 
- This method also allows communication between the hypervisor and the virtual machines. 
- The major drawback of this virtual network is that none of the systems outside the hypervisor can reach the virtual machines.

- The NATed virtual network is created with the help of iptables, specifcally using the masquerading option. 
- Hence, stopping iptables when VMs are in use can cause network disruption inside the virtual machines:

![](https://i.loli.net/2019/04/23/5cbebf3974ae3.png)

## 4.1 virt-manager
all the same with routed network, excpet the last step. select "any physical device + NAT"

![](https://i.loli.net/2019/04/23/5cbecaa5ef640.png)

## 4.2 Bridged network using a physical NIC, VLAN interface, bond interface, and bonded VLAN interface (aka shared physical interface)

- In most of the production environment, you will be using a bridge confguration that directly connects a physical NIC to the bridge. 
- The primary reason for using this confguration is that your virtual machine will act as a system that is in the same network as the physical NIC. 物理网卡直接连到虚拟网桥，是的虚拟机和物理机使用同一个网络
- Unlike the NATed mode, virtual machines can be accessed directly using their IP address, which is essential when you host a service
on your virtual machines.

??? ，没搞明白 到底要干嘛？？？ 默认自动生成的就是这种模式，并且默认开启了DHCP，如果虚拟机网卡配置文件也是DHCP并且onboot=yes，将会自动获取一个ip地址

#### 1. prepare ens33, ens34, ens35, br0
- let ens35 be a management interface, for ssh to physical host
- attach ens33 and ens34 to br0
- because we have ens35 to use ssh, br0 can have no ip. if no ens35, br0 hsould have ip
```
# cat ifcfg-ens33
DEVICE=ens33
TYPE=Ethernet
HWADDR=52:54:00:32:56:aa
ONBOOT=yes
NM_CONTROLLED=no
BRIDGE=br0

# cat ifcfg-br0
DEVICE=br0
TYPE=Bridge
ONBOOT=yes
NM_CONTROLLED=no
```
Enable the network service and start it.
```# systemctl enable network```
```# systemctl disable NetworkManager```
```# ifup br0; ifup eth1```
```# brctl show```
#### 2. create a bond (bond0) using ens33 and ens34 and add it to br0
```
# ifdown br0; ifdown eth1
# cat ifcfg-ens33
DEVICE=ens33
TYPE=Ethernet
HWADDR=52:54:00:32:56:aa
ONBOOT=yes
NM_CONTROLLED=no
SLAVE=yes
MASTER=bond0

# cat ifcfg-ens34
DEVICE=ens34
TYPE=Ethernet
HWADDR=52:54:00:a6:02:51
ONBOOT=yes
NM_CONTROLLED=no
SLAVE=yes
MASTER=bond0

# cat ifcfg-bond0
DEVICE=bond0
ONBOOT=yes
BONDING_OPTS='mode=1 miimon=100'
BRIDGE=br0
NM_CONTROLLED=no
```
- ```# ifup bond0```
- ```# brctl show```
#### 3. create a tagged VLAN named bond0.123 and will be added to the br0 bridge

- ```# ifdown bond0; ifdown br0```
- ```# cp ifcfg-bond0 ifcfg-bond0.123```
```diff
# cat ifcfg-bond0.123
DEVICE=bond0.123
ONBOOT=yes
BONDING_OPTS='mode=1 miimon=100'
BRIDGE=br0
NM_CONTROLLED=no
VLAN=yes

# cat ifcfg-bond0
  DEVICE=bond0
  ONBOOT=yes
  BONDING_OPTS='mode=1 miimon=100'
- BRIDGE=br0
  NM_CONTROLLED=no
```
- ```# ifup bond0.123```






# 5. MacVTap

- MacVTap is used when you do not want to create a normal bridge, but want the users in local network to access your virtual machine. 
- This connection type is not used in production systems and is mostly used on workstation systems.

Navigate to Add Hardware | Network to add a virtual NIC as the MacVTap interface using virt-manager. At Network source, select the physical NIC interface on the host where you want to enable MacVTap

