# 使用Kolla部署Pike版本的OpenStack-allinone云平台

1. openstack 概述
5. Kolla 概述和 openstack 所有结点 linux 系统初始配置
6. 安装 kolla-ansible
7. 自定义 kolla-ansible 安装 openstack 的相关配置文件
8. 开始基于 kolla-ansible 安装 openstack 私有云

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

[](https://i.loli.net/2019/03/22/5c9459a84104b.png)

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


# 2.  Kolla概述 和 openstack all-in-one初始配置

kolla是openstack下面用于自动化部署的一个项目，它基于docker和ansible来实现，docker主要负责镜像制作，容器管理。而ansible主要负责环境的部署和管理。

Kolla实际上是分为两大块的，一部分，Kolla提供了生产环境级别的镜像，涵盖了Openstack用到的各个服务，另一部分是自动化的部署，也就是上面说的ansible部分。最开始两个部分是在一个项目中的（也就是Kolla），从O版本开始将两个部分独立开来，Kolla项目用来构建所有服务的镜像，Kolla-ansible用来执行自动化部署。

## 5.1   linux系统环境配置

#### 1. 关闭Selinux和防火墙
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
[root@xuegod63 ~]# yum install ntp
[root@xuegod63 ~]# systemctl enable ntpd.service
[root@xuegod63 ~]# systemctl start ntpd.service
```
#### 7.配置 pip 镜像源，方便快速下载python库（这一步很重要）
并没有做
```
[root@xuegod63 ~]# mkdir ~/.pip
[root@xuegod63 ~]# vim  ~/.pip/pip.conf  #写入下以内容

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























[](https://i.loli.net/2019/03/21/5c93b3f3ac779.png)
