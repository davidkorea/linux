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






















