# Networking

- VLAN 
  - VLAN is a networking technology that enables a single switch to act as if it was multiple independent switches. Specifically, two hosts that are connected to the same switch but on different VLANs do not see each other’s traffic. 
  - OpenStack is able to take advantage of VLANs to isolate the traffic of different tenants, even if the **tenants happen to have instances running on the same compute host**. Each VLAN has an associated numerical ID, between 1 and 4095. We say “VLAN 15” to refer to the VLAN with numerical ID of 15.
- two syntaxes for expressing a netmask
  - dotted quad, 192.168.1.5, 255.255.255.0
  - classless inter-domain routing (CIDR), 192.168.1.5/24

## Classic with Open vSwitch

- OpenStack services - controller node
  - **Operational SQL server with neutron database and appropriate configuration in the neutron.conf file.**
    - [Install and configure controller node](https://docs.openstack.org/neutron/pike/install/controller-install-rdo.html)
    - Configure networking options
      - ```yum install openstack-neutron openstack-neutron-ml2 python-neutronclientwhich -y```
      
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
## Controller - node
**Operational SQL server with neutron database and appropriate configuration in the neutron.conf file.**
- [create a database, service credentials, and API endpoints](https://docs.openstack.org/neutron/pike/install/controller-install-rdo.html)
- Configure networking options
  - ```yum install openstack-neutron openstack-neutron-ml2 python-neutronclient which -y```
  - vim /etc/neutron/neutron.conf
    ```
    [DEFAULT]
    verbose = True
    core_plugin = ml2
    service_plugins = router
    allow_overlapping_ips = true
    transport_url = rabbit://openstack:RABBIT_PASS@controller
    auth_strategy = keystone
    notify_nova_on_port_status_changes = true
    notify_nova_on_port_data_changes = true
    
    [database]
    connection = mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron
    
    [keystone_authtoken]
    auth_uri = http://controller:5000
    auth_url = http://controller:35357
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = NEUTRON_PASS
    
    [nova]
    auth_url = http://controller:35357
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = nova
    password = NOVA_PASS
    
    [oslo_concurrency]
    lock_path = /var/lib/neutron/tmp
    ```
  - vim /etc/neutron/plugins/ml2/ml2_conf.ini 
    ```
    [ml2]
    type_drivers = flat,vlan,gre,vxlan
    tenant_network_types = vlan,gre,vxlan
    mechanism_drivers = openvswitch,l2population
    extension_drivers = port_security

    [ml2_type_flat]
    flat_networks = external

    [ml2_type_vlan]
    network_vlan_ranges = external,vlan:1:1000

    [ml2_type_gre]
    tunnel_id_ranges = 1:1000

    [ml2_type_vxlan]
    vni_ranges = 1:1000

    [securitygroup]
    enable_ipset = True
    ```
  - Start the following services: Server

- vim /etc/nova/nova.conf
  ```
  [neutron]
  url = http://controller:9696
  auth_url = http://controller:35357
  auth_type = password
  project_domain_name = default
  user_domain_name = default
  region_name = RegionOne
  project_name = service
  username = neutron
  password = NEUTRON_PASS
  service_metadata_proxy = true
  metadata_proxy_shared_secret = METADATA_SECRET
  ```
- ```ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini```
- ```su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron```初始化数据库
- ```systemctl restart openstack-nova-api.service```
- ```systemctl enable neutron-server.service```
- ```systemctl start neutron-server.service```
- 验证
  ```
  . admin-openrc
  neutron ext-list
  ```
  
  
  
## Network node
- vim /etc/neutron/neutron.conf
  ```
  [DEFAULT]
  verbose = True
  ...??
  ```
- vim /etc/neutron/plugins/ml2/openvswitch_agent.ini
  ```
  [ovs]
  local_ip = TUNNEL_INTERFACE_IP_ADDRESS(172.16.251.10)
  bridge_mappings = vlan:br-vlan,external:br-ex

  [agent]
  tunnel_types = gre,vxlan
  l2_population = True
  prevent_arp_spoofing = True

  [securitygroup]
  firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
  enable_security_group = True
  ```
  - Replace ```TUNNEL_INTERFACE_IP_ADDRESS``` with the IP address of the interface that handles GRE/VXLAN project networks.

- vim /etc/neutron/l3_agent.ini
  ```
  [DEFAULT]
  verbose = True
  interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
  use_namespaces = True
  external_network_bridge =
  ```
- vim /etc/neutron/dhcp_agent.ini
  ```
  [DEFAULT]
  verbose = True
  interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
  dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
  enable_isolated_metadata = True
  ```
- vim /etc/neutron/metadata_agent.ini
  ```
  [DEFAULT]
  verbose = True
  nova_metadata_ip = controller
  metadata_proxy_shared_secret = METADATA_SECRET(11111)
  ```

















































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
