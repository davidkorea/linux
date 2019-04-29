# KVM comprehensive network based on Open vSwitch

# 1. Isolated Network
- ```yum install -y qemu-kvm```
- ```ln -sv /usr/libexec/qemu-kvm /usr/bin/```
- ```mkdir -p /images/cirros```
- ```cp cirros-0.3.4-x86_64-disk.img cirros-0.3.4-1.img```
- ```cp cirros-0.3.4-x86_64-disk.img cirros-0.3.4-2.img```

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
  bridge=br-in
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
- ```qemu-kvm -m 128 -smp 1 -name cirros1 -drive file=/images/cirros/cirros-0.3.4-1.img,media=disk,if=virtio -net nic,model=virtio,macaddr=52:54:00:00:00:01 -net tap,ifname=vif0.0,script=/etc/qemu-ifup,downscript=/etc/qemu-ifdown```








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
