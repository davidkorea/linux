


# 1. keystone
- provider = fernet
  - 当集群运行较长一段时间后，访问其 API 会变得奇慢无比，究其原因在于 Keystone 数据库存储了大量的 token 导致性能太差，解决的办法是经常清理 token。为了避免上述问题，社区提出了Fernet token，fernet 是当前主流推荐的token格式，它采用 cryptography 对称加密库(symmetric cryptography，加密密钥和解密密钥相同) 加密 token，具体由 AES-CBC 加密和散列函数 SHA256 签名。Fernet 是专为 API token 设计的一种轻量级安全消息格式，不需要存储于数据库，减少了磁盘的 IO，带来了一定的性能提升。为了提高安全性，需要采用 Key Rotation 更换密钥。




















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
