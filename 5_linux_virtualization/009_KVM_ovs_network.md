# KVM comprehensive network based on Open vSwitch

## 1. Open vSwitch
- 使用yum安装
```yum install -y openvswitch openvswitch-devel openvswitch-test openvswitch-debuginfor```
- 启动服务
```
systemctl enable  openvswitch
systemctl start  openvswitch
```
```
1. download atr.gz：http://www.openvswitch.org//download/
2. tar xf, cd openvswitch folder
3. ```./configure```
4. ```make -j 4```
5. ```make install```
```
## 2. 创建2个计算节点
- 网卡1：仅主机，用于传送控制命令
- 网卡2：VMnet，用于node之间的通信，内网
  
