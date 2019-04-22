# KVM_libvirt_network
# 1. Virtual Network
## 1. brctl 
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
#### 2.  create and add a TAP(osi layer 2) device the bridge
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
