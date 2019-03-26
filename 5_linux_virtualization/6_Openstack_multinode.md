### Error when install kolla-ansible by pip, need to get by clone from github
# 使用Kolla部署Pike版本的OpenStack多节点云平台

1. 准备 openstack 多结点实验环境
2. 安装 kolla-ansible
3. 自定义 kolla-ansible 安装 openstack 的相关配置文件
4. 开始基亍 kolla-ansible 安装 openstack 私有于
5. OpenStack 使用方法
6. 查看创建好的 openstack 项目中的信息和于主机网络连

# 1. 准备 openstack 多结点实验环境

## 1.1 准备硬件环境
新创建 3 台虚拟机，分别作为 controller节点，compute节点，storage节点。其中controller节点 2 张网卡，compute、storage 节点各 1 张网卡。操作系统为 centos7.5. 注：controller 节点 也作为安装结点
|主机名| IP |角色| 内存| 网卡类型|
|-|-|-|-|-|
|xuegod63| 192.168.1.63| controller 节点| 8G| ens33 和 ens38 都桥接|
|xuegod64| 192.168.1.64| compute 节点| 4| ens33| 桥接|
|xuegod62| 192.168.1.62| storage 节点| 4G| ens33| 桥接|








- 编辑 /etc/kolla/globals.yml 自定义openstack中部署事项
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

- 135 #enable_cinder: "no"                         # 开启cinder
+ 135 enable_cinder: "yes"

- 140 #enable_cinder_backend_lvm: "no"
+ 140 enable_cinder_backend_lvm: "yes"		   # cinder 后端用 lvm			

- 151 #enable_haproxy: "yes"
+ 151 enable_haproxy: "no"                         # 去了前面的#号，改yes为no，关闭高可用

- 294 #cinder_volume_group: "cinder-volumes"
+ 294 cinder_volume_group: "cinder-volumes"	   # 取消前面的#号，这个卷组的名字是在 client163 上创建的

```

-  [root@server162 kolla]# vim multinode 

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






















1. kolla-ansible -i /etc/kolla/multinode prechecks
```
PLAY RECAP *********************************************************************
client163                  : ok=3    changed=0    unreachable=0    failed=1   
client164                  : ok=7    changed=0    unreachable=0    failed=1   
server162                  : ok=67   changed=0    unreachable=0    failed=0  
```
2. [root@server162 kolla]# kolla-ansible -i ./multinode deploy

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
fatal: [server162]: FAILED! => {"attempts": 10, "changed": false, "module_stderr": "Shared connection to server162 closed.\r\n", "module_stdout": "Traceback (most recent call last):\r\n  File \"/root/.ansible/tmp/ansible-tmp-1553493174.26-227571964728856/AnsiballZ_wait_for.py\", line 113, in <module>\r\n    _ansiballz_main()\r\n  File \"/root/.ansible/tmp/ansible-tmp-1553493174.26-227571964728856/AnsiballZ_wait_for.py\", line 105, in _ansiballz_main\r\n    invoke_module(zipped_mod, temp_path, ANSIBALLZ_PARAMS)\r\n  File \"/root/.ansible/tmp/ansible-tmp-1553493174.26-227571964728856/AnsiballZ_wait_for.py\", line 48, in invoke_module\r\n    imp.load_module('__main__', mod, module, MOD_DESC)\r\n  File \"/tmp/ansible_wait_for_payload_xQGt1V/__main__.py\", line 629, in <module>\r\n  File \"/tmp/ansible_wait_for_payload_xQGt1V/__main__.py\", line 558, in main\r\nsocket.error: [Errno 104] Connection reset by peer\r\n", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}

NO MORE HOSTS LEFT *************************************************************
	to retry, use: --limit @/usr/share/kolla-ansible/ansible/site.retry

PLAY RECAP *********************************************************************
client163                  : ok=24   changed=0    unreachable=0    failed=0   
client164                  : ok=24   changed=0    unreachable=0    failed=0   
server162                  : ok=48   changed=2    unreachable=0    failed=1   
```
3. kolla-ansible -i ./multinode bootstrap-servers

```
PLAY RECAP *********************************************************************
client163                  : ok=37   changed=6    unreachable=0    failed=0   
client164                  : ok=37   changed=6    unreachable=0    failed=0   
server162                  : ok=38   changed=18   unreachable=0    failed=0   
```
