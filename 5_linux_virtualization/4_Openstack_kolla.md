# 使用Kolla部署Pike版本的OpenStack all-in-one云平台

1. openstack 概述
2. Kolla 概述和 openstack 所有结点 linux 系统初始配置
3. 安装 kolla-ansible
4. 自定义 kolla-ansible 安装 openstack 的相关配置文件
5. 开始基于 kolla-ansible 安装 openstack 私有云

# 1. openstack概述

OpenStack是一个NASA美国国家航空航天局和Rackspace合作研发的，以Apache许可证授权，并且是一个自由软件和开放源代码项目。
## 1.1 基本概念
OpenStack是一个云平台管理的项目，它不是一个软件。这个项目由几个主要的组件组合起来完成一些具体的工作。OpenStack是一个旨在为公共及私有云的建设与管理提供软件的开源项目。它的社区拥有超过130家企业及1350位开发者，这些机构与个人都将OpenStack作为基础设施即服务（简称IaaS）资源的通用前端。
- IaaS（Infrastructure as a Service），即基础设施即服务
  - 消费者通过Internet 可以从完善的计算机基础设施获得服务。这类服务称为基础设施即服务。基于 Internet 的服务（如存储和数据库）是 IaaS的一部分。 比如： 在腾讯云上买一台云主机（8个CPU，32G，5T硬盘云主机等）
- PaaS是Platform-as-a-Service的缩写，意思是开发即。 把平台作为一种服务提供的商业模式。例：OpenShift是红帽的云开发平台即服务（PaaS）
  - OpenShift是自由和开放源码的云计算平台，它可以使开发人员能够创建、测试和运行他们的应用程序，并且可以把它们部署到云中。Openshift广泛支持多种编程语言和框架，如Java，Ruby和PHP等。另外它还提供了多种集成开发工具如Eclipse integration，JBoss Developer Studio和 Jenkins等。OpenShift 基于一个开源生态系统为移动应用，数据库服务等，提供支持。
- SaaS是Software-as-a-Service的简称。docs.google.com或 微软的office  365

- 按拥有者分类：
  - 公有云（Public Cloud）、私有云（Private Cloud）、混合云（Hybrid Cloud）
  - 按照技术厂商分类：VMware vSphere、微软云计算解决方案、亚马逊AWS、OpenStack等
  - 注：国内云平台使用opensctack二次开发比较多。

## 1.2 openstack各组件关系

openstack核心组成主要有:
- Keystone（身份认证）
- Nova（计算）
- Neutron（网络）
- Glance（镜像存储）
- Cinder（块存储）
- Swift（对象存储）
- Horizon（web UI界面）
- Ceilometer（计量）
- Heat（部署编排）
- Trove（数据库）

#### 1. 身份认证（Keystone）
统一的授权、认证管理。所有组件都依赖于Keystone提供3A(Account, Authentication, Authorization)服务。
- 3A认证 
  1. 认证(Authentication)，验证用户的身份与可使用的网络服务；
  2. 授权(Authorization)：依据认证结果开放网络服务给用户；
  3. 计帐(Accounting)：记录用户对各种网络服务的用量，并提供给计费系统。整个系统在网络管理与安全问题中十分有效。
  - 比如：宽带收费就是3A认证的典型例子：输入帐号密码（认证）-》开10M带宽（授权）-》在营业厅（计帐）
#### 2. 计算管理（Nova）
Nova是OpenStack云中的计算组织控制器。Nova自身并没有提供任何虚拟化能力，相反它使用libvirt API来与被支持的虚拟技术Hypervisors交互。如：kvm、Xen、VMware等虚拟化技术。
#### 3. 网络（Neutron）
实现虚拟机的网络资源管理如网络连接、ip管理、公网映射

#### 4. 存储
1. 镜像管理（Glance）：主要存储和管理系统镜像，如cento镜像 
2. 块存储（Cinder）：为虚拟机提供存储空间。 比如硬盘，分区，目前支持ip-san、fc-san等。
3. 对象存储（Swift）：OpenStack Swift 开源项目提供了弹性可伸缩、高可用的分布式对象存储服务，适合存储大规模非结构化数据。通过key/value的方式实现对文件的存储，现在的云盘就是这样的，和MFS，HDFS类似

注：如果客户需要一个1000T的存储空间，使用Cinder或Glance就不行，效率太低。这时就用Swift。

#### 5. 界面（Horizon）
安装好后，openstack的web界面控制台DashBoard 

#### 6. 计量（Ceilometer）
Ceilometer是OpenStack中的一个子项目，它像一个漏斗一样，能把OpenStack内部发生的几乎所有的事件都收集起来，然后为计费和监控以及其它服务提供数据支撑。
Ceilometer  [sɪ'lɒmɪtə]  云幂测量仪

#### 7. 部署编排（Heat）
是一个编排引擎，它可以基于文本文件形式的模板启动多个复合云应用程序（这些文件可以被视为代码）。简单来说，Heat为OpenStack用户提供了一种自动创建云组件（如网络、实例、存储设备等）的方法。

#### 8. 数据库（Trove）
为关系型数据库和非关系型数据库引擎提供可扩展的和可靠的云数据库服务，并继续改进其功能齐全、可扩展的开源框架。Trove  [trəʊv] 宝库

![](https://i.loli.net/2019/03/22/5c9459a84104b.png)

## 1.3 Openstack的网络模式有5种
1. Local模式：一般测试时使用，只需一台物理机即可。
2. GRE模式：隧道模式， VLAN数量没有限制，性能有点问题。
3. Vlan模式：vlan数量有4096的限制
4. VXlan模式：vlan数量没有限制，性能比GRE好。
5. Flat模式：管理员创建租户直接到外网，不需要NAT。

- XLAN概述，VXLAN是由思科与VMware提出。VLAN与VXLAN之间有何区别：VXLAN显然更具可扩展性(4,096个VLAN网 vs 1600万个VXLAN网)
  - LAN     局域网
  - VLAN    虚拟局域网
  - VXLAN   虚拟扩展局域网
  - VXLAN (Virtual Extensible LAN，虚拟扩展局域网) 它是一种在UDP中封装MAC的简单机制，可以创建跨多个物理IP子网的虚拟2层子网
## 1.4  OpenStack部署方法，

1. 社区手册http://docs.openstack.org
2. RDOhttps://www.rdoproject.org（http://openstack.redhat.com）
3. RedHat Enterprise Linux OpenStack Platform  （E210 考试）http://www.redhat.com/en/technologies/linux-platforms/openstack-platform
4. Mirantis（Fuel）https://www.mirantis.com
5. 高级定制 Puppet、Chef
6. kolla 基于docker安装openstack ，把openstack每个组件做成docker实例 

**选择基于docker的kolla方式，优点是占用内存资源少，一般8G内存即可。社区手册方法至少12G内存**


# 2. Openstack all-in-one 配置

kolla是openstack下面用于自动化部署的一个项目，它基于docker和ansible来实现，docker主要负责镜像制作，容器管理。而ansible主要负责环境的部署和管理。

Kolla实际上是分为两大块的，一部分，Kolla提供了生产环境级别的镜像，涵盖了Openstack用到的各个服务，另一部分是自动化的部署，也就是上面说的ansible部分。最开始两个部分是在一个项目中的（也就是Kolla），从O版本开始将两个部分独立开来，Kolla项目用来构建所有服务的镜像，Kolla-ansible用来执行自动化部署。

## 2.1  linux系统环境配置

#### 1.关闭Selinux和防火墙
```
[root@server162 ~]# vim /etc/selinux/config
SELINUX=disabled
[root@server162 ~]# reboot   #如果原来的系统开着selinux，那么需要重启，才能关闭selinux  
```
#### 2.关闭Firewalld
```
[root@server162 ~]# systemctl stop firewalld
[root@server162 ~]# systemctl disable firewalld
[root@server162 ~]# systemctl status firewalld
```
#### 3.安装 Epel源
```
[root@server162 ~]# yum install epel-release -y
```
#### 4.配置 Hostname
```
[root@server162 ~]# cat /etc/hostname
server162
```
#### 5.配置/etc/hosts
```
[root@server162 ~]# cat /etc/hosts     # 添加以下两行
192.168.0.162 server162.com server162
192.168.0.163 client163.com client163
```

#### 6.同步时间
```
[root@server162 ~]# yum install ntp
[root@server162 ~]# systemctl enable ntpd.service
[root@server162 ~]# systemctl start ntpd.service
```
#### 7.配置 pip 镜像源，方便快速下载python库（这一步很重要）
并没有做
```
[root@server162 ~]# mkdir ~/.pip
[root@server162 ~]# vim  ~/.pip/pip.conf  #写入下以内容

[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
```
#### 8. ens33, ens34
ens33
```
[root@server162 ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
  BOOTPROTO="none"        # 添加双网卡，此处自动被更改为dhcp，所以手动改回none会static
  IPADDR=192.168.0.162
  GATEWAY=192.168.0.1
  DNS1=168.126.63.1
  DNS2=164.124.101.2
```
ens34
```
[root@server162 kolla]# vim /etc/sysconfig/network-scripts/ifcfg-ens34
  TYPE=Ethernet
  BOOTPROTO=none
  NAME=ens34
  DEVICE=ens34
  ONBOOT=yes
````

## 2.2 安装基础包和docker服务
#### 1. 安装基础包
```
[root@server162 ~]# yum install python-devel libffi-devel gcc openssl-devel git python-pip -y
[root@server162 ~]# pip install -U pip      # 升级一下pip，不然后，后期安装会报警告
[root@server162 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2   # 安装必要的一些系统工具
```
#### 2. 添加docker yum源并安装docker
```
[root@server162 ~]# systemctl stop libvirtd && systemctl disable libvirtd && systemctl status libvirtd
                          # 停止kvm的服务libvirt，否则和docker不兼容
[root@server162 ~]# yum remove  docker docker-io docker-selinux python-docker-py 
                          # 如果有的话，卸载旧的Docker，否则可能会不兼容
                          
[root@server162 ~]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@server162 ~]# cat /etc/yum.repos.d/docker-ce.repo  # 查看新生成的yum源配置文件 

[root@server162 ~]# yum -y install docker-ce             # 安装 Docker-CE社区版本
[root@server162 ~]# systemctl start docker && systemctl enable docker && systemctl status docker   # 启动Docker服务
```
#### 3. 设置docker volume卷挂载方式
```
[root@server162 ~]# mkdir /etc/systemd/system/docker.service.d
[root@server162 ~]# tee /etc/systemd/system/docker.service.d/kolla.conf << 'EOF'
[Service]
MountFlags=shared
EOF
```
注：加上MountFlags=shared后，当docker宿主机新增分区时，docker服务不用重启。如果不加docker服务服务重启，docker中的实例才可以使用新加的磁盘或分区。 添加这个参考后，后期在openstack中使用cinder存储服务时，新加磁盘比较方便。
#### 4. 指定docker 镜像加速器 （很重要，不然后期从国外下载docker镜像会直接报错，而且速度慢） 
并没有做
```
[root@server162 ~]# mkdir /etc/docker/             # 安装docker之后会自动创建该目录，没有的话需要手动创建
[root@server162 ~]# vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://e9yneuy4.mirror.aliyuncs.com"]  
}

[root@server162 ~]# systemctl daemon-reload        # 修改了启动脚本，需要执行
[root@server162 ~]# systemctl enable docker && systemctl restart docker && systemctl status docker
```
注：如果需要使用自己的本地私有仓库，写成如下：
```
{
  "registry-mirrors": ["https://e9yneuy4.mirror.aliyuncs.com"]  
  "insecure-registries": ["192.168.1.63:4000"]
}
```

## 2.3 安装kolla-ansible
#### 1. 安装 ansible
```
[root@server162 ~]# yum install ansible -y
```
#### 2. 安装 kolla-ansible
1. 下载kolla-ansible的代码
```
[root@server162 ~]# cd /root
[root@server162 ~]# git clone http://git.trystack.cn/openstack/kolla-ansible -b stable/pike    # 下载pike版本
```
2. 安装kolla-ansible需要依赖包
```
[root@server162 ~]# cd /root/kolla-ansible
[root@server162 kolla-ansible]# pip install .    #这里.代表当前目录 安装目录下所有requirements.txt文件
```
3. 复制kolla-ansible的相关配置文件
```
[root@server162 kolla-ansible]# cp -r etc/kolla /etc/kolla/
[root@server162 kolla-ansible]# cp ansible/inventory/* /etc/kolla/
[root@server162 kolla-ansible]# ls /etc/kolla/
all-in-one  globals.yml  multinode  passwords.yml
```
  - all-in-one #安装单节点openstack的ansible自动安装配置文件
  - multinode #安装多节点openstack的ansible自动安装配置文件
  - globals.yml #openstack 部署的自定义配置文件 
  - passwords.yml  #openstack中各个服务的密码

#### 3. 修改虚拟机类型为qemu
# 这一步超级重要！！！
```diff
- **如果vmware开了“虚拟化Intel VT”功能，就不用写这个了**
+ 简直不要太重要了，不设置qemu，取法启动虚拟机，卡到硬盘启动grub位置。
+ 所以之后ping网络不同，内网10地址和外网192地址都不同。只有网络拓扑中显示的网关可以10.0.0.1和192.168.0.103可以
```
![](https://i.loli.net/2019/03/24/5c97798f3e10b.png)


注：如果是在虚拟机里装kolla，希望可以启动再启动虚拟机，那么你需要把virt_type=qemu，默认是kvm。
```
[root@server162 kolla-ansible]# mkdir -p /etc/kolla/config/nova
[root@server162 kolla-ansible]# cat << EOF > /etc/kolla/config/nova/nova-compute.conf
[libvirt]
virt_type=qemu
cpu_mode = none
EOF
```
## 2.4 自定义kolla-ansible安装openstack的相关配置文件

#### 1. 自动生成openstack各服务的密码文件
```diff
[root@server162 kolla-ansible]# kolla-genpwd         # /etc/kolla/passwords.yml自动为该文件生产随机密码

[root@server162 ~]# vim /etc/kolla/passwords.yml

- 158 keystone_admin_password: HsPbEQHxTqmewKYNoRPpIOyQNdEYpHy36OX67TG3
+ 158 keystone_admin_password: 11111 
```
这是登录Dashboard，admin使用的密码，你可以根据自己需要进行修改。

#### 2. 编辑 /etc/kolla/globals.yml 自定义openstack中部署事项
```diff
[root@server162 ~]# vim /etc/kolla/globals.yml      # 配置openstack安装中的参数
- 15 #kolla_base_distro: "centos"                  # 选择下载的镜像为基于centos版本的镜像
+ 15 kolla_base_distro: "centos"

- 18 #kolla_install_type: "binary"                 # 去了前面的#号，使用yum安装二进制包安装
+ 18 kolla_install_type: "binary"                  # 源码安装，指的是使用git clone源码安装

- 21 #openstack_release: ""                        # 指定安装pike版本的openstack，       
+ 21 openstack_release: "pike"                     # 后期下载的openstack相关的docker镜像的tag标记也都为pike

  23 # Location of configuration overrides
  24 #node_custom_config: "/etc/kolla/config"      # 配置文件的位置

- 31 kolla_internal_vip_address: "10.10.10.254"    # 我们没有启用高可用，所以这里的IP可以和ens33一样
+ 31 kolla_internal_vip_address: "192.168.0.162"   # 也可以独立写一个和ens33同网段的IP
                                                   # 如果配置了高可用，这里要使用一个没被占用的IP,
                                                   # 这个IP是搭建HA高可用的浮动IP,
                                                   # 此IP将由keepalived管理以提供高可用性,
                                                   # 应设置为和network_interface ens33 同一个网段的地址

- 73 #network_interface: "eth0"                    # Kolla-Ansible需要设置一些网络选项。
+ 73 network_interface: "ens33"                    # 这是openstack内部多个管理类型网络的默认接口

- 88 #neutron_external_interface: "eth1"           # 所需的第二个接口专用于Neutron外部（或公共）网络，可以是vlan或flat
+ 88 neutron_external_interface: "ens34"           # 此接口应在没有IP地址的情况下处于活动状态
                                                   # 如果不是，openstack云平台中的云主机实例将无法访问外部网络
                                                   # 只要网卡启动着，就可以了，不要给IP，有IP时br-ex桥接就不成功了

  135 #enable_cinder: "no"                         # 先不开启cinder

- 151 #enable_haproxy: "yes"
+ 151 enable_haproxy: "no"                         # 去了前面的#号，改yes为no，关闭高可用

```
## 2.5 开始基于kolla-ansible安装openstack私有云
### 1. 生成SSH Key，并授信本节点：
```
[root@server162 ~]# ssh-keygen
[root@server162 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.162
```
### 2. 配置单节点主机清单文件
```
[root@server162 kolla]# vim /etc/kolla/all-in-one           # 把localhost替换成server162

:1,$s/localhost/server162/

:1,$s/ansible_connection=local//   

# 将以下[]中的组件，都安装到server162这台机器上

[control]
server162     

[network]
server162     

[compute]
server162     

[storage]
server162     

[deployment]
server162       
```
### 3. 开始部署OpenStack
以下命令中的```-i /etc/kolla/all-in-one```可以省略
#### i. 对主机进行预部署检查
```
[root@server162 ~]# kolla-ansible -i /etc/kolla/all-in-one prechecks
```
有可能会报错，不用管，继续下面步骤
#### ii. 拉取镜像
```
[root@server162 ~]# kolla-ansible -i /etc/kolla/all-in-one  pull
```
```
[root@server162 ~]# docker images                 # 查看下载到的镜像
REPOSITORY              TAG       IMAGE ID            CREATED             SIZE
kolla/centos-binary-cron   pike      659fa47c7d43        About an hour ago   455MB
... ...

[root@server162 ~]# docker images | wc -l          # 整个过程，会下载了32个镜像
30
```
#### iii. 最后进入实际的OpenStack部署
```
[root@server162 ~]# kolla-ansible -i /etc/kolla/all-in-one deploy 
```
因为前面已经下载的镜像，所以这时，安装会快一些。如果前面没有下载镜像，那么这时，还会边下载各种openstack相关的镜像边部署docker实例。

#### iv. 验证部署
```
[root@server162 ~]# kolla-ansible -i /etc/kolla/all-in-one  post-deploy
```
```
PLAY RECAP ***********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0 
```
这样就创建 /etc/kolla/admin-openrc.sh 文件
```
[root@server162 ~]# ll /etc/kolla/admin-openrc.sh
-rw-r--r-- 1 root root 323 Mar 22 14:40 /etc/kolla/admin-openrc.sh

[root@server162 ~]# cat /etc/kolla/admin-openrc.sh                   # 查看openstack登录帐号
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_TENANT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=11111
export OS_AUTH_URL=http://192.168.0.162:35357/v3
export OS_INTERFACE=internal
export OS_IDENTITY_API_VERSION=3
export OS_REGION_NAME=RegionOne
```

![](https://i.loli.net/2019/03/22/5c9484c33d1a3.png)

![](https://i.loli.net/2019/03/21/5c93b3f3ac779.png)
