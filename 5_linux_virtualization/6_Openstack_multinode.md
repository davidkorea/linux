### Error when install kolla-ansible by pip, need to get by clone from github -> [Issue](https://github.com/davidkorea/linux_study/blob/master/5_linux_virtualization/6_Openstack_multinode.md#issue-%E6%8C%89%E7%85%A7%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3pip-install-kolla-ansible-%E5%85%A8%E6%98%AF%E9%94%99)
# 使用Kolla部署Pike版本的OpenStack多节点云平台

1. 准备 openstack 多结点实验环境
2. 安装 kolla-ansible
3. 自定义 kolla-ansible 安装 openstack 的相关配置文件
4. 开始基亍 kolla-ansible 安装 openstack 私有云
5. 实戓-通过命令行来创建自己的网络拓扑图

# 1. 准备 openstack 多结点实验环境

## 1.1 准备硬件环境
新创建 3 台虚拟机，分别作为 controller节点，compute节点，storage节点。其中controller节点 2 张网卡，compute、storage 节点各 1 张网卡。操作系统为 centos7.5. 注：controller 节点 也作为安装结点

|主机名| IP |角色| 内存| 网卡类型|硬盘|
|-|-|-|-|-|-|
|server162| 192.168.0.162| controller| 8G| ens33 和 ens34 都桥接|1 \* 100G|
|client163| 192.168.0.163| compute| 4G| ens33 桥接| 1 \* 20G |
|client164| 192.168.0.164| storage| 4G| ens33 桥接| 2 \* 20G, 作为cinder的lvm后端 |

## 1.2 linux系统环境配置 - All Nodes
### 1.2.1 基本环境
#### 1.（all）关闭Selinux和防火墙
```
[root@server162 ~]# vim /etc/selinux/config
SELINUX=disabled
[root@server162 ~]# reboot   #如果原来的系统开着selinux，那么需要重启，才能关闭selinux  
```
#### 2.（all）关闭Firewalld
```
[root@server162 ~]# systemctl stop firewalld
[root@server162 ~]# systemctl disable firewalld
[root@server162 ~]# systemctl status firewalld
```
#### 3.（all）安装 Epel源
```
[root@server162 ~]# yum install epel-release -y
```
#### 4.（all）配置 Hostname
```
[root@server162 ~]# cat /etc/hostname
server162
```
```
[root@client163 ~]# cat /etc/hostname
client163
```
```
[root@client164 ~]# cat /etc/hostname
client164
```
#### 5.（all）配置/etc/hosts
hosts 文件中的短主机名，给 rabbitmq 使用的, rabbitmq 服务会使用短主机域名
```
[root@server162 ~]# cat /etc/hosts     # 添加以下两行
192.168.0.162 server162.com server162
192.168.0.163 client163.com client163
192.168.0.164 client164.com client164
```
```
[root@server162 ~]# scp /etc/hosts 192.168.0.163:/etc/
[root@server162 ~]# scp /etc/hosts 192.168.0.164:/etc/
```

#### 6.（all）同步时间
```
[root@server162 ~]# yum install ntp
[root@server162 ~]# systemctl enable ntpd.service
[root@server162 ~]# systemctl start ntpd.service
```
#### 7.（all）配置 pip 镜像源，方便快速下载python库（这一步很重要）
并没有做
```
[root@server162 ~]# mkdir ~/.pip
[root@server162 ~]# vim  ~/.pip/pip.conf  #写入下以内容

[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
```
#### 8. ens33（all）, ens34（controller ONLY）
- ens33
```
[root@server162 ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
  BOOTPROTO="none"        # 添加双网卡，此处自动被更改为dhcp，所以手动改回none会static
  IPADDR=192.168.0.162
  GATEWAY=192.168.0.1
  DNS1=168.126.63.1
  DNS2=164.124.101.2
```
- ens34
```
[root@server162 kolla]# vim /etc/sysconfig/network-scripts/ifcfg-ens34
  TYPE=Ethernet
  BOOTPROTO=none
  NAME=ens34
  DEVICE=ens34
  ONBOOT=yes
````
### 1.2.2 docker环境 - All Nodes

#### 1.（all）安装基础包
```
[root@server162 ~]# yum install python-devel libffi-devel gcc openssl-devel git python-pip -y
[root@server162 ~]# pip install -U pip      # 升级一下pip，不然后，后期安装会报警告
[root@server162 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2   # 安装必要的一些系统工具
```
#### 2.（all）添加docker yum源并安装docker
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
#### 3.（all）设置docker volume卷挂载方式
```
[root@server162 ~]# mkdir /etc/systemd/system/docker.service.d
[root@server162 ~]# tee /etc/systemd/system/docker.service.d/kolla.conf << 'EOF'
[Service]
MountFlags=shared
EOF
```
注：加上MountFlags=shared后，当docker宿主机新增分区时，docker服务不用重启。如果不加docker服务服务重启，docker中的实例才可以使用新加的磁盘或分区。 添加这个参考后，后期在openstack中使用cinder存储服务时，新加磁盘比较方便。
#### 4. （all）指定docker 镜像加速器 （很重要，不然后期从国外下载docker镜像会直接报错，而且速度慢） 
```
[root@server162 ~]# mkdir /etc/docker/             # 安装docker之后会自动创建该目录，没有的话需要手动创建
[root@server162 ~]# vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://e9yneuy4.mirror.aliyuncs.com"]  
}
```
```
[root@server162 ~]# systemctl daemon-reload        # 修改了启动脚本，需要执行。否则之后precheck会报错
[root@server162 ~]# systemctl enable docker && systemctl restart docker && systemctl status docker
```
#### 5. （cinder ONLY）storage配置存储信息
```
[root@client163 ~]# ls /dev/sdb*
/dev/sdb
[root@client163 ~]# pvcreate /dev/sdb
Physical volume "/dev/sdb" successfully created.

[root@client163 ~]# vgcreate cinder-volumes /dev/sdb  	  # 创建名字为cinder-volumes的卷组，给后期cinder使用
Volume group "cinder-volumes" successfully created

[root@client163 ~]# systemctl enable lvm2-lvmetad.service
[root@client163 ~]# vgs
VG #PV #LV #SN Attr VSize VFree
cinder-volumes 1 0 0 wz--n- <20.00g <20.00g
```

到此 3 台机器的基础软件包环境已经安装好。

# 2. 安装 kolla-ansible
## 2.1 安装 ansible
```
[root@server162 ~]# yum install ansible -y
```
## 2.2 安装 kolla-ansible

> [官方文档 ](https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html)推荐使用```pip install kolla-ansible```来安装，但是，对于pike OpenStack，千万不要pip安装。报错一大堆，已知bug，并没被解决。还是安装下面git clone的方法进行配置，与all-in-one操作相同。
>
> [Issue: 按照官方文档pip install kolla-ansible 全是错](https://github.com/davidkorea/linux_study/blob/master/5_linux_virtualization/6_Openstack_multinode.md#issue-%E6%8C%89%E7%85%A7%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3pip-install-kolla-ansible-%E5%85%A8%E6%98%AF%E9%94%99)

#### 1. 下载kolla-ansible的代码
```
[root@server162 ~]# cd /root
[root@server162 ~]# git clone http://git.trystack.cn/openstack/kolla-ansible -b stable/pike    # 下载pike版本
```
#### 2. 安装kolla-ansible需要依赖包
```
[root@server162 ~]# cd /root/kolla-ansible
[root@server162 kolla-ansible]# pip install .    #这里 . 代表当前目录 安装目录下所有requirements.txt文件
```
#### 3. 复制kolla-ansible的相关配置文件
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
**这一步超级重要！！！** 如果是在**虚拟机里装kolla，希望虚拟机中再嵌套启动虚拟机**，需要把virt_type=qemu，默认是kvm。
```
[root@server162 kolla-ansible]# mkdir -p /etc/kolla/config/nova
[root@server162 kolla-ansible]# cat << EOF > /etc/kolla/config/nova/nova-compute.conf
[libvirt]
virt_type=qemu
cpu_mode=none
EOF
```

# 3. kolla-ansible自定义安装openstack的相关配置文件
## 3.1 自动生成openstack各服务的密码文件
```diff
[root@server162 kolla-ansible]# kolla-genpwd         # /etc/kolla/passwords.yml自动为该文件生产随机密码

[root@server162 ~]# vim /etc/kolla/passwords.yml

- 158 keystone_admin_password: HsPbEQHxTqmewKYNoRPpIOyQNdEYpHy36OX67TG3
+ 158 keystone_admin_password: 11111 
```
这是登录Dashboard，admin使用的密码，你可以根据自己需要进行修改。

## 3.2 编辑 /etc/kolla/globals.yml 自定义openstack中部署事项

```diff
[root@server162 ~]# vim /etc/kolla/globals.yml     # 配置openstack安装中的参数
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

- 135 #enable_cinder: "no"                         # 开启cinder
+ 135 enable_cinder: "yes"

- 140 #enable_cinder_backend_lvm: "no"
+ 140 enable_cinder_backend_lvm: "yes"		   # cinder 后端用 lvm			

- 151 #enable_haproxy: "yes"
+ 151 enable_haproxy: "no"                         # 去了前面的#号，改yes为no，关闭高可用

- 294 #cinder_volume_group: "cinder-volumes"
+ 294 cinder_volume_group: "cinder-volumes"	   # 取消前面的#号，这个卷组的名字是在 client163 上创建的

```
## 3.3 配置 multinode 多结点清单文件

此处为git clone获得的清单文件，与直接pip install获得的清单文件稍有不同

```
  2 # additional groups are for more control of the environment.
  3 [control]
  4 # These hostname must be resolvable from your deployment host
  5 server162
  6 
  7 # The above can also be specified as follows:
  8 #control[01:03]     ansible_user=kolla
  9 
 10 # The network nodes are where your l3-agent and loadbalancers will run
 11 # This can be the same as a host in the control group
 12 [network]
 13 server162
 14 
 15 [compute]
 16 client164
 17 
 18 [monitoring]
 19 server162
 20 
 21 # When compute nodes and control nodes use different interfaces,
 22 # you can specify "api_interface" and other interfaces like below:
 23 #compute01 neutron_external_interface=eth0 api_interface=em1 storage_i    nterface=em1 tunnel_interface=em1
 24 
 25 [storage]
 26 client163
 27 
 28 [deployment]
 29 server162
 30 
```

# 4. 开始基于kolla-ansible安装openstack私有云

[Issue: 按照官方文档pip install kolla-ansible 全是错](https://github.com/davidkorea/linux_study/blob/master/5_linux_virtualization/6_Openstack_multinode.md#issue-%E6%8C%89%E7%85%A7%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3pip-install-kolla-ansible-%E5%85%A8%E6%98%AF%E9%94%99)

#### 1. 生成 SSH Key，并授信所有节点
```
[root@server162 ~]# ssh-keygen
[root@server162 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@server162
[root@server162 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@client163
[root@server162 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@client164
```
注：ssh-copy-id 复制公钥时，后面要写主机名，不要写 IP。因为后期 ansible 自劢安装节点，主机清单中写的是主机名不是 IP。

#### 2. 预部署检查
```
[root@server162 kolla]# kolla-ansible -i /etc/kolla/multinode prechecks
```

#### 3. 部署
```
[root@server162 ~]# kolla-ansible -i /etc/kolla/multinode deploy 
```
因为此时边下载各种 openstack 相关的镜像并部署 docker 实例，会比较慢。等待 30 分钟左右。就可以了。配置了 docker镜像加速结点，会比较快

#### 4. 验证部署
执行验证部署后，才会创建 /etc/kolla/admin-openrc.sh 文件
```
[root@server162 ~]# kolla-ansible -i /etc/kolla/multinode post-deploy
Post-Deploying Playbooks : ansible-playbook -i /etc/kolla/multinode -e @/etc/kolla/globals.yml -e @/etc/kolla/passwords.yml -e CONFIG_DIR=/etc/kolla  /usr/share/kolla-ansible/ansible/post-deploy.yml 

PLAY [Creating admin openrc file on the deploy node] ***************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [template] ****************************************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0   

[root@server162 ~]# ls /etc/kolla/admin-openrc.sh 
/etc/kolla/admin-openrc.sh
[root@server162 ~]# cat !$
cat /etc/kolla/admin-openrc.sh
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

http://192.168.0.162/auth/login/?next=/home/ 访问成功

# 5. init-runonce 创建测试云主机demo1
## 5.1 安装 OpenStack client 端
方便后期使用命令行操作 openstack
```
[root@server162 ~]# pip install python-openstackclient python-glanceclient python-neutronclient
```
```
[root@server162 ~]# pip install ipaddress --ignore-installed ipaddress 
[root@server162 ~]# pip install python-openstackclient python-glanceclient
python-neutronclient
```
```
[root@server162 ~]# pip install PyYAML --ignore-installed PyYAML
[root@server162 ~]# pip install python-openstackclient python-glanceclient
python-neutronclient
```
```
[root@server162 ~]# pip install pyinotify --ignore-installed pyinotify 
[root@server162 ~]# pip install python-openstackclient python-glanceclient
python-neutronclient
```

## 5.2 使用 init-runonce 脚本创建openstack demo项目
#### 1. 修改 init-runonce
init-runonce 是在 openstack 中快速创建一个于项目例子的脚本
```diff
[root@server162 ~]# vim /usr/share/kolla-ansible/init-runonce

- 12 EXT_NET_CIDR='10.0.2.0/24'
- 13 EXT_NET_RANGE='start=10.0.2.150,end=10.0.2.199' 
- 14 EXT_NET_GATEWAY='10.0.2.1'
 
+ EXT_NET_CIDR='192.168.0.0/24' 
+ EXT_NET_RANGE='start=192.168.0.10,end=192.168.0.20' 
+ EXT_NET_GATEWAY='192.168.0.1'
```
注：192.168.0.0 的网络，就是上面 ens34 接入的局域网中的地址，这个网络是通过局域网络中的路由器访问互联网。配置好这个，装完虚拟机就可以直接 ping 通

#### 2. 开始创建demo

> [Issue：ImportError: cannot import name decorate](https://github.com/davidkorea/linux_study/blob/master/5_linux_virtualization/5_Openstack_all_in_one.md#issueimporterror-cannot-import-name-decorate)
> 
> fixed: ```pip install -U decorator```

```
[root@server162 ~]# source /etc/kolla/admin-openrc.sh  # 先加载这个文件，把环境变量加入系统中，才有权限执行下面的命令
[root@server162 ~]# cd /usr/share/kolla-ansible 
[root@server162 kolla-ansible]# ./init-runonce         # 最后弹出以下命令，复制后直接执行即可

Done.

To deploy a demo instance, run:

openstack server create \
    --image cirros \
    --flavor m1.tiny \
    --key-name mykey \
    --nic net-id=a0cbe0d4-3f7d-4275-856b-5d810c7a1297 \
    demo1
    
[root@server162 kolla-ansible]# openstack server create \
>     --image cirros \
>     --flavor m1.tiny \
>     --key-name mykey \
>     --nic net-id=a0cbe0d4-3f7d-4275-856b-5d810c7a1297 \
>     demo1
+-------------------------------------+-----------------------------------------------+
| Field                               | Value                                         |
+-------------------------------------+-----------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                        |
| OS-EXT-AZ:availability_zone         |                                               |
| OS-EXT-SRV-ATTR:host                | None                                          |
| OS-EXT-SRV-ATTR:hypervisor_hostname | None                                          |
| OS-EXT-SRV-ATTR:instance_name       |                                               |
| OS-EXT-STS:power_state              | NOSTATE                                       |
| OS-EXT-STS:task_state               | scheduling                                    |
| OS-EXT-STS:vm_state                 | building                                      |
| OS-SRV-USG:launched_at              | None                                          |
| OS-SRV-USG:terminated_at            | None                                          |
| accessIPv4                          |                                               |
| accessIPv6                          |                                               |
| addresses                           |                                               |
| adminPass                           | 5d8WtK6wUYVi                                  |
| config_drive                        |                                               |
| created                             | 2019-03-26T09:13:37Z                          |
| flavor                              | m1.tiny (1)                                   |
| hostId                              |                                               |
| id                                  | fbc4742b-ad4c-4797-8e17-25923a6ed101          |
| image                               | cirros (3c23cb2a-9951-4cb6-866e-696c663e6c3e) |
| key_name                            | mykey                                         |
| name                                | demo1                                         |
| progress                            | 0                                             |
| project_id                          | 4ff1fb42b3904f0b8020b6512ef14ae8              |
| properties                          |                                               |
| security_groups                     | name='default'                                |
| status                              | BUILD                                         |
| updated                             | 2019-03-26T09:13:37Z                          |
| user_id                             | e41eb00d7d1145ebaf4d00de0a4a7422              |
| volumes_attached                    |                                               |
+-------------------------------------+-----------------------------------------------+
```
## 5.3 查看已创建openstack demo项目信息和云主机网络连通性
#### 1. command查看网络信息
```
[root@server162 ~]# source /etc/kolla/admin-openrc.sh 		# 要读一下这个环境变量配置文件

[root@server162 ~]# openstack router list 			# 查看路由信息
[root@server162 ~]# openstack router show demo-router 		# 查看 demo-router 路由信
[root@server162 ~]# openstack network list 			# 查看网络信息
[root@server162 ~]# openstack server show demo1 		# 查看名字为 demo1 的虚拟机信息
```
分配浮动ip，测试远程连接虚拟机，成功
```
[root@server162 ~]# ssh 192.168.0.13
The authenticity of host '192.168.0.13 (192.168.0.13)' can't be established.
RSA key fingerprint is SHA256:xy5AkUHiLMOb2cCdrV3XpLF34sMUAvNqB+0XfPJwuHA.
RSA key fingerprint is MD5:ed:c1:ad:a8:69:a3:4f:5c:a2:75:8f:5f:b0:7d:e3:df.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.0.13' (RSA) to the list of known hosts.
Please login as 'cirros' user, not as root

Connection to 192.168.0.13 closed.
[root@server162 ~]# ssh cirros@192.168.0.13
$ pwd
/home/cirros
```

# 6. 实戓-通过命令行来创建自己的网络拓扑图

先删除已创建的demo1云主机，管理员菜单下删除路由，以及所有网络。顺序一定是先删除路由，才能删除网络，否则先删除网络会报错删不掉


运行source openers.sh 脚本，添加可用openstack环境变量
```
[root@server162 ~]# source /etc/kolla/admin-openrc.sh 
```
## 6.1 创建网络
#### 1. 创建外网
```
[root@server162 ~]# openstack network create --external --provider-physical-network physnet1 --provider-network-type flat public_network
```
```
[root@server162 ~]# openstack subnet create --no-dhcp --allocation-pool 'start=192.168.0.10,end=192.168.0.20' --network public_network --subnet-range 192.168.0.0/24 --gateway 192.168.0.1 public_subnet
```
#### 2. 创建内网
```
[root@server162 ~]# openstack network create --provider-network-type vxlan demo_network
```
```
[root@server162 ~]# openstack subnet create --subnet-range 10.0.0.0/24 --network demo_network --gateway 10.0.0.1 --dns-nameserver 8.8.8.8 demo_subnet
```
#### 3. 内网，外网之间创建路由
```
[root@server162 ~]# openstack router create demo_router
```
```
[root@server162 ~]# openstack router add subnet demo_router demo_subnet
[root@server162 ~]# openstack router set --external-gateway public_network demo_router
```






















# [Issue] 按照官方文档pip install kolla-ansible 全是错
1. kolla-ansible -i ./multinode bootstrap-servers
```
PLAY RECAP *********************************************************************
client163                  : ok=37   changed=6    unreachable=0    failed=0   
client164                  : ok=37   changed=6    unreachable=0    failed=0   
server162                  : ok=38   changed=18   unreachable=0    failed=0   
```
2. kolla-ansible -i /etc/kolla/multinode prechecks
```
PLAY RECAP *********************************************************************
client163                  : ok=3    changed=0    unreachable=0    failed=1   
client164                  : ok=7    changed=0    unreachable=0    failed=1   
server162                  : ok=67   changed=0    unreachable=0    failed=0  
```
3. kolla-ansible -i ./multinode deploy
```
TASK [mariadb : include_tasks] *************************************************
included: /usr/share/kolla-ansible/ansible/roles/mariadb/tasks/bootstrap_cluster.yml for server162

TASK [mariadb : Running MariaDB bootstrap container] ***************************
fatal: [server162]: FAILED! => {"changed": true, "msg": "Container exited with non-zero return code 1"}

RUNNING HANDLER [mariadb : restart slave mariadb] ******************************

RUNNING HANDLER [mariadb : restart master mariadb] *****************************
	to retry, use: --limit @/usr/share/kolla-ansible/ansible/site.retry

PLAY RECAP *********************************************************************
client163                  : ok=29   changed=20   unreachable=0    failed=0   
client164                  : ok=29   changed=20   unreachable=0    failed=0   
server162                  : ok=53   changed=30   unreachable=0    failed=1   
```
reboot and try again
```
RUNNING HANDLER [mariadb : Waiting for master mariadb] *************************
FAILED - RETRYING: Waiting for master mariadb (10 retries left).
FAILED - RETRYING: Waiting for master mariadb (9 retries left).
FAILED - RETRYING: Waiting for master mariadb (8 retries left).

FAILED - RETRYING: Waiting for master mariadb (2 retries left).
FAILED - RETRYING: Waiting for master mariadb (1 retries left).
fatal: [server162]: FAILED! => {"attempts": 10, "changed": false, "module_stderr": "Shared connection to server162 closed.\r\n",
"module_stdout": "Traceback (most recent call last):\r\n  File \"/root/.ansible/tmp/ansible-tmp-1553493174.26-
227571964728856/AnsiballZ_wait_for.py\", line 113, in <module>\r\n    _ansiballz_main()\r\n  File \"/root/.ansible/tmp/ansible-tmp-
1553493174.26-227571964728856/AnsiballZ_wait_for.py\", line 105, in _ansiballz_main\r\n    invoke_module(zipped_mod, temp_path,
ANSIBALLZ_PARAMS)\r\n  File \"/root/.ansible/tmp/ansible-tmp-1553493174.26-227571964728856/AnsiballZ_wait_for.py\", line 48, in
invoke_module\r\n    imp.load_module('__main__', mod, module, MOD_DESC)\r\n  File \"/tmp/ansible_wait_for_payload_xQGt1V/__main__.py\",
line 629, in <module>\r\n  File \"/tmp/ansible_wait_for_payload_xQGt1V/__main__.py\", line 558, in main\r\nsocket.error: [Errno 104]
Connection reset by peer\r\n", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}

NO MORE HOSTS LEFT *************************************************************
	to retry, use: --limit @/usr/share/kolla-ansible/ansible/site.retry

PLAY RECAP *********************************************************************
client163                  : ok=24   changed=0    unreachable=0    failed=0   
client164                  : ok=24   changed=0    unreachable=0    failed=0   
server162                  : ok=48   changed=2    unreachable=0    failed=1   
```

