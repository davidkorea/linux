> https://docs.openstack.org/install-guide/openstack-services.html






# 4. controller - glance
https://www.cnblogs.com/jsonhc/p/7698502.html

https://docs.openstack.org/glance/pike/index.html

# 3. controller - keystone
- ```yum install openstack-keystone```, 包含来python-openstackclient
- ```yum install -y openstack-utils```

- db
  ```
  MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'keystone';
  Query OK, 0 rows affected (0.00 sec)

  MariaDB [(none)]> GRANT ALL ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'keystone';
  Query OK, 0 rows affected (0.00 sec)
  
  MariaDB [(none)]> FLUSH PRIVILEGES;
  ```
  - IDENTIFIED 'keystone'是密码


# 1. Openstack环境搭建 - controller
Offical manual: https://docs.openstack.org/install-guide/environment-packages-rdo.html
## 1.1 OpenStack packages
- ```yum install centos-release-openstack-pike```
- ```yum upgrade```
- ```yum install python-openstackclient```
- ```yum install openstack-selinux```





# 1. 系统环境搭建
## 1.1 controller
- ens33 bridge 192.168.0.11
- ens37 host only 172.16.251.11 （manage）
- ```chkconfig NetworkManager off```, or conflict with brctl
  - ping 192.168.0.1 ok!
  - ping 172.16.251.1 ok!
- ```crontab -e```
  - ```*/3 * * * * /usr/sbin/ntpdate 192.168.0.1 &> /dev/null```
- hosts
  ```
  172.16.251.11 controller.openstack.com controller
  172.16.251.12 compute1.openstack.com compute1
  ```
- SNAT 管理网络连接到外网
  - ```iptables -F```
  - ``` iptables -t nat -A POSTROUTING -s 172.16.251.0/24 -j SNAT --to 192.168.0.11```
  - **保存SNAT规则 service iptables save**
    - [centos7中service iptables save指令使用失败的解决方案 #4](https://github.com/davidkorea/linux_study/issues/4)
- Mysql ```yum install -y mariadb-server mariadb```
  - ```systemctl start mariadb```
  - ```systemctl enable mariadb```
  - ```mysql```
  
## 1.2 compute1
- ~~ens33 bridge 192.168.0.12 (openstack中用不到，只是用来联网yum安装使用)~~
- ens37 host only 172.16.251.12 **gw 172.16.251.11**（manage）
- ens38 VMnet2 ~~192.168.100.12~~ （VM）
- ```chkconfig NetworkManager off```, or conflict with brctl
  - ping 192.168.0.1 ok! 通过网关指向controller节点，并且controller节点开启来SNAT，ping外网成功
  - ping 172.16.251.1 ok!
- ```crontab -e```
  - ```*/3 * * * * /usr/sbin/ntpdate 192.168.0.1 &> /dev/null```
- hosts
  ```
  172.16.251.11 controller.openstack.com controller
  172.16.251.12 compute1.openstack.com compute1
  ```
- ping 物理网络网关192.168.0.1 icmp_seq=1 Destination Host Prohibited
  - [icmp_seq=1 Destination Host Prohibited #5](https://github.com/davidkorea/linux_study/issues/5)
  - 清空iptables 即可
  - ping baidu ip 220.181.57.216 ok

# 0. Basic

- 30-50 servers no need to implement openstack, need technical supports for openstack, devops chanllege
- over 200 servers ok
#### 1. 卷存储，各个硬盘中的存储信息，block存储，持久存储（Cinder）
- 一般来讲，云主机的 内存，cpu都是同一台物理主机提供，而存储一般也是该主机一同来提供。
  - 因为如果使用共享存储，会大大依赖于网络IO，否则延迟会很大
- 但是，一个云主机如果关机，所有cpu，内存，存储资源都将回收。下次有可能安装之前创建的参数模版（内存2G，cpu2core）再启动在其他物理主机。
  - 所以该虚拟机的磁盘镜像文件，即使存在某个物理服务器上也没有用
  - 所以，应该尽可能不实用磁盘镜像文件的方式来实现存储功能，而是单独使用一个卷存储
  - 这样每次虚拟机开机，都可以让不同的物理节点通过ISCSI去调用该卷存储中的相应存储空间，并复制到本地来运行
- 所以卷存储，也应该是一个分布式存储，因为虚拟机启动后，需要大量的复制磁盘文件

####  2. 磁盘镜像存储（Swift）
- Glance是一个前段接口，提供元数据检索服务。实际images 存储在Swift
- 压力测试，公司早上上班，一次性同时启动200个虚拟机实例，此时卷存储压力太大
  - 所以测试环境中无法预料到的问题，生产环境中会遇到
  - 所以写个脚本来进行压力测试，一次性启动200个虚拟机
- 实际中，磁盘镜像模版有可能存在分布式存储中
  - 每一个磁盘镜像都被分割成多份，分别存储在一组分布式存储中的不同的主机上
  - 此时，同时复制200个磁盘镜像文件时，网络IO和磁盘IO才不会成文瓶颈
 
####  3. 数据库，存储各个实例启动模版的参数信息 
- 启动虚拟机时的检索
- 关闭虚拟机时的更新
- 向虚拟机更新资源时的更新
####  4. 队列服务
- 一次性启动20个虚拟机实例
- 不应该手动等待启动完成这一个，再手动启动下一个虚拟机实例
- 而是以此提交请求后，自动创建好20个虚拟机实例
####  5. multi-tenancy多租户
- 多租户，创建自己的私有云
- openstack中一个租户，相当于一个project，里面可以有多个虚拟机实例
####  6. 对象存储
- 每个磁盘镜像文件自带格式的对象文件，如qcow2
