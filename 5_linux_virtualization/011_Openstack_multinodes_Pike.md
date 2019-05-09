# Networking

- VLAN 
  - VLAN is a networking technology that enables a single switch to act as if it was multiple independent switches. Specifically, two hosts that are connected to the same switch but on different VLANs do not see each other’s traffic. 
  - OpenStack is able to take advantage of VLANs to isolate the traffic of different tenants, even if the **tenants happen to have instances running on the same compute host**. Each VLAN has an associated numerical ID, between 1 and 4095. We say “VLAN 15” to refer to the VLAN with numerical ID of 15.
- two syntaxes for expressing a netmask
  - dotted quad, 192.168.1.5, 255.255.255.0
  - classless inter-domain routing (CIDR), 192.168.1.5/24

## Classic with Open vSwitch

- OpenStack services - controller node
  - Operational SQL server with neutron database and appropriate configuration in the neutron.conf file.
  - Operational message queue service with appropriate configuration in the neutron.conf file.
  - Operational OpenStack Identity service with appropriate configuration in the neutron.conf file.
  - Operational OpenStack Compute controller/management service with appropriate configuration to use neutron in the nova.conf file.
  - **Neutron server service, ML2 plug-in, and any dependencies.**
- OpenStack services - network node
  - Operational OpenStack Identity service with appropriate configuration in the neutron.conf file.
  - **Open vSwitch service, Open vSwitch agent, L3 agent, DHCP agent, metadata agent, and any dependencies.**
- OpenStack services - compute nodes
  - Operational OpenStack Identity service with appropriate configuration in the neutron.conf file.
  - Operational OpenStack Compute controller/management service with appropriate configuration to use neutron in the nova.conf file.
  - **Open vSwitch service, Open vSwitch agent, and any dependencies.**
  






















































# 1. keystone
- provider = fernet
  - 当集群运行较长一段时间后，访问其 API 会变得奇慢无比，究其原因在于 Keystone 数据库存储了大量的 token 导致性能太差，解决的办法是经常清理 token。为了避免上述问题，社区提出了Fernet token，fernet 是当前主流推荐的token格式，它采用 cryptography 对称加密库(symmetric cryptography，加密密钥和解密密钥相同) 加密 token，具体由 AES-CBC 加密和散列函数 SHA256 签名。Fernet 是专为 API token 设计的一种轻量级安全消息格式，不需要存储于数据库，减少了磁盘的 IO，带来了一定的性能提升。为了提高安全性，需要采用 Key Rotation 更换密钥。

- provider = keystone.token.providers.uuid.Provider
  - UUID认证原理：当用户需要进行操作时（比如访问nova创建虚拟机），用户拿着有效的用户名/密码先去keystone认证，keystone返回给用户一个token(即UUID)。之后用户进行其他操作（如访问nova），先出示这个token给nova-api,nova收到请求后，就用这个token去向keystone进行请求验证。keystone通过比对token，以及检查token的有效期，判断token的有效性，最后返回给nova结果。缺陷：每次请求都要经过keystone进行验证，造成性能瓶颈。


















-----

- controller
  - ens33: 192.168.0.111, bridge
  - ens37: 172.16.251.111, Host Only
  - ens38: no ip, VMnet2

- compute1
  - ens33: 192.168.0.112, bridge
  - ens37: 172.16.251.112, Host Only
  - ens38: no ip, VMnet2

- network
  - ens33: 192.168.0.110, bridge
  - ens37: 172.16.251.110, Host Only
  - ens38: no ip, VMnet2
- ```yum install -y ntp```, **All Nodes**
- OpenStack packages for **Pike** - **All Nodes**
  - ```yum install centos-release-openstack-pike -y```
  - ```yum upgrade```
  - ```yum install python-openstackclient```
  - ```yum install openstack-selinux```
