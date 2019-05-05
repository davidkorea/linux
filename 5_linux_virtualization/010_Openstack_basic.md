> https://docs.openstack.org/install-guide/openstack-services.html




# 4. controller - glance
- https://www.cnblogs.com/jsonhc/p/7698502.html
- https://docs.openstack.org/glance/pike/index.html

## 4.1 Prerequisites
Before you install and configure the Image service, you must create a database, service credentials, and API endpoints
- ```mysql -u root -p11111```
  - MariaDB [(none)]> ```CREATE DATABASE glance;```
  - MariaDB [(none)]> ```GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'glance';```
  - MariaDB [(none)]> ```GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'glance';```
    - GLANCE_DBPASS: glance
  
- Source the admin credentials to gain access to admin-only CLI commands
  - ```. admin-openrc```
- To create the service credentials, complete these steps
  - Create the glance user
    - ```openstack user create --domain default --password-prompt glance```
      ```
      User Password: 11111
      Repeat User Password: 11111
      ```
  - Add the admin role to the glance user and service project
    - ```openstack role add --project service --user glance admin```
  - Create the glance service entity
    - ```openstack service create --name glance --description "OpenStack Image" image```
  
- Create the Image service API endpoints
  - ```openstack endpoint create --region RegionOne image public http://controller:9292```
  - ```openstack endpoint create --region RegionOne image internal http://controller:9292```
  - ```openstack endpoint create --region RegionOne image admin http://controller:9292```
  
## 4.2 Install and configure components
- Install the packages
  - ```yum install openstack-glance -y```
- Edit the /etc/glance/glance-api.conf file
  ```diff
    1805 [database]
  - 1823 #connection = <None>
  + 1823 connection = mysql+pymysql://glance:glance@controller/glance
  
    3283 [keystone_authtoken]
  + auth_uri = http://controller:5000
  + auth_url = http://controller:35357
  + memcached_servers = controller:11211
  + auth_type = password
  + project_domain_name = default
  + user_domain_name = default
  + project_name = service
  + username = glance
  + password = 11111                           # GLANCE_PASS=11111
  
    4210 [paste_deploy]
  - 4235 #flavor = keystone
  + 4235 flavor = keystone
  
  
    1916 [glance_store]  
  - 1943 #stores = file,http
  - 1975 #default_store = file
  - 2294 #filesystem_store_datadir = /var/lib/glance/images  
  + 1943 stores = file,http
  + 1975 default_store = file
  + 2294 filesystem_store_datadir = /var/lib/glance/images
  ```
  ```
  [root@controller ~]# grep -v "^#" /etc/glance/glance-api.conf | grep -v "^$"
  [DEFAULT]
  [cors]
  [database]
  connection = mysql+pymysql://glance:glance@controller/glance
  [glance_store]
  stores = file,http
  default_store = file
  [image_format]
  [keystone_authtoken]
  auth_uri = http://controller:5000
  auth_url = http://controller:35357
  memcached_servers = controller:11211
  auth_type = password
  project_domain_name = default
  user_domain_name = default
  project_name = service
  username = glance
  password = 11111
  [matchmaker_redis]
  [oslo_concurrency]
  [oslo_messaging_amqp]
  [oslo_messaging_kafka]
  [oslo_messaging_notifications]
  [oslo_messaging_rabbit]
  [oslo_messaging_zmq]
  [oslo_middleware]
  [oslo_policy]
  [paste_deploy]
  flavor = keystone
  [profiler]
  [store_type_location_strategy]
  [task]
  [taskflow_executor]
  ```
  
  
  
  
  
  
  
  
  
  
  
# 3. controller - keystone
Offical manual: [Keystone Installation Tutorial for Red Hat and CentOS](https://docs.openstack.org/keystone/pike/install/index-rdo.html)
## 3.1 Prerequisites
Before you install and configure the Identity service, you must create a database.
- ```mysql -u root -p11111```
  - MariaDB [(none)]> ```CREATE DATABASE keystone;```
  - MariaDB [(none)]> ```GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'keystone';```
  - MariaDB [(none)]> ```GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'keystone';```
    - KEYSTONE_DBPASS: keystone
## 3.2 Install and configure components
- ```yum install openstack-keystone httpd mod_wsgi -y```
- Edit the /etc/keystone/keystone.conf
  ```diff
    643  [database]
  - 661  connection = <None>
  + 661  connection = mysql+pymysql://keystone:keystone@controller/keystone

    2730 [token]
  - 2774 #provider = fernet
  + 2774 provider = fernet
  ```
- Populate the Identity service database
  - ```su -s /bin/sh -c "keystone-manage db_sync" keystone```
- Initialize Fernet key repositories
  - ```keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone```
  - ```keystone-manage credential_setup --keystone-user keystone --keystone-group keystone```
- Bootstrap the Identity service
  - ``` keystone-manage bootstrap --bootstrap-password 11111  --bootstrap-admin-url http://controller:35357/v3/  --bootstrap-internal-url http://controller:5000/v3/ --bootstrap-public-url http://controller:5000/v3/ --bootstrap-region-id RegionOne```
  
  ```
  keystone-manage bootstrap --bootstrap-password 11111 \
  --bootstrap-admin-url http://controller:35357/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
  ```
  - ADMIN_PASS: 11111
  - Replace ADMIN_PASS with the password used in the keystone-manage bootstrap command in ```keystone-install-configure-rdo```
## 3.3 Configure the Apache HTTP server
- Edit the /etc/httpd/conf/httpd.conf
  ```diff
  -  95 ServerName www.example.com:80
  +  95 ServerName controller
  ```
- ```ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/```
## 3.4 Finalize the installation
- Start the Apache HTTP service
  ```
  systemctl enable httpd.service
  systemctl start httpd.service
  ```
- Configure the administrative account
  - ```export OS_USERNAME=admin```
  - ```export OS_PASSWORD=11111```, ADMIN_PASS=11111
  - ```export OS_PROJECT_NAME=admin```
  - ```export OS_USER_DOMAIN_NAME=Default```
  - ```export OS_PROJECT_DOMAIN_NAME=Default```
  - ```export OS_AUTH_URL=http://controller:35357/v3```
  - ```export OS_IDENTITY_API_VERSION=3```

## 3.5 Create a domain, projects, users, and roles
- service project
  ```openstack project create --domain default --description "Service Project" service```
- Regular (non-admin) demo project
  ```openstack project create --domain default --description "Demo Project" demo```
  - Create the demo user
    - ```openstack user create --domain default --password-prompt demo```
      ```
      User Password: 11111
      Repeat User Password: 11111
      ```
  - Create the user role
    - ```openstack role create user```
  - Add the user role to the demo project and user
    - ```openstack role add --project demo --user demo user```

## 3.6 Verify operation

- Unset the temporary OS_AUTH_URL and OS_PASSWORD environment variable
  - ```unset OS_AUTH_URL OS_PASSWORD```

- As the admin user, request an authentication token
  - ```openstack --os-auth-url http://controller:35357/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name admin --os-username admin token issue```
    - password: 11111
    - This command uses the password for the admin user
- As the demo user, request an authentication token
  - ```openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name demo --os-username demo token issue```
    - password: 11111
    - This command uses the password for the demo user and API port 5000 which only allows regular (non-admin) access to the Identity service AP
## 3.7 Create OpenStack client environment scripts
> The paths of the client environment scripts are unrestricted. For convenience, you can place the scripts in any location
- reate and edit the admin-openrc file located in /root
  ```bash
  [root@controller ~]# vim admin-openrc
  export OS_PROJECT_DOMAIN_NAME=Default
  export OS_USER_DOMAIN_NAME=Default
  export OS_PROJECT_NAME=admin
  export OS_USERNAME=admin
  export OS_PASSWORD=11111                        # ADMIN_PASS=11111
  export OS_AUTH_URL=http://controller:35357/v3
  export OS_IDENTITY_API_VERSION=3
  export OS_IMAGE_API_VERSION=2
  ```
- Create and edit the demo-openrc file located in /root
  ```bash
  [root@controller ~]# vim demo-openrc
  export OS_PROJECT_DOMAIN_NAME=Default
  export OS_USER_DOMAIN_NAME=Default
  export OS_PROJECT_NAME=demo
  export OS_USERNAME=demo
  export OS_PASSWORD=11111                        # DEMO_PASS=11111
  export OS_AUTH_URL=http://controller:5000/v3
  export OS_IDENTITY_API_VERSION=3
  export OS_IMAGE_API_VERSION=2
  ```
- ```. admin-openrc```
  - ```openstack token issue```



# 2. Openstack环境搭建 - controller
Offical manual: https://docs.openstack.org/install-guide/environment-packages-rdo.html
## 1.1 OpenStack packages
- ```yum install centos-release-openstack-pike```
- ```yum upgrade```
- ```yum install python-openstackclient -y```
- ```yum install openstack-selinux -y```
## 1.2 SQL database
- ```yum install mariadb mariadb-server python2-PyMySQL -y```
- Create and edit the /etc/my.cnf.d/openstack.cnf
  ```
  [mysqld]
  bind-address = 172.16.251.111

  default-storage-engine = innodb
  innodb_file_per_table = on
  max_connections = 4096
  collation-server = utf8_general_ci
  character-set-server = utf8
  ```
  - bind-address: management IP address of the controller node 
  - character-set-server: 提前设置好字符集，否则创建数据库时，需要手动制定 SET uft8
- ```systemctl enable mariadb.service```, ```systemctl start mariadb.service```
- ```mysql_secure_installation```
  - set root password: 11111



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
