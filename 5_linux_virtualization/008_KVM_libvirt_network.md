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

## 2 Isolated virtual network
a closed network for the virtual machines. only the virtual machines which are added to this network can communicate with each other

![](https://i.loli.net/2019/04/22/5cbd807f1c416.png)

## 2.1 virt-manager
![](https://i.loli.net/2019/04/22/5cbd880d58486.png)
## 2.2 virsh with xml
- delete setting above on virt-manager first
- create the isolated network using the virsh command. For that, we need to create an XML fle with the following contents and save it as isolated.xml
#### 1. create isolated.xml
create the isolated network using the virsh command. For that, we need to create an XML fle with the following contents and save it as isolated.xml
```xml
[root@server162 ~]# vim isolated.xml 
<network>
  <name>isolated</name>
</network>
```
#### 2. defne a network using the XML flie created above
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
#### 3. xml file that virst created based on isolated.xml we provided 
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
- libvirt added a few additional parameters. and created a new bridge for this network
- libvirt added the rest of the required parameters; you can mention these in your XML fle when required. recommendation is that you leave it to libvirt to avoid conﬂicts.
- ```net-create``` is similar to ```net-define```. 
  - ```net-create``` will not create a persistent virtual network. Once destroyed, it is removed and has to be created again using the net-create command.
 - recommand ```net-define```

