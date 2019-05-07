
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
- ```yum install -y ntp```
- OpenStack packages for **Pike**
  - ```yum install centos-release-openstack-pike -y```
  - ```yum upgrade```
  - ```yum install python-openstackclient```
  - ```yum install openstack-selinux```
