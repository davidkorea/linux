# 使用自动化运维工具Ansible集中化管理服务器
1. ansible概述和运行机制
2. 实战-安装并配置Ansible管理两个节点
3. ansible常见模块高级使用方法 
4. 实战-使用Playbook批量部署多台LAMP环境

# 1.Aansible概述和运行机制
Ansible是一款为类Unix系统开发的自由开源的配置和自动化工具。它用Python写成，类似于saltstack和Puppet，但是有一个不同和优点是我们不需要在节点中安装任何客户端。它使用SSH来和节点进行通信。Ansible基于 Python paramiko 开发，分布式，无需客户端，轻量级，配置语法使用 YMAL 及 Jinja2模板语言，更强的远程命令执行操作

Ansible 在管理节点将 Ansible 模块通过 SSH 协议推送到被管理端执行，执行完之后自动删除，可以使用 SVN 等来管理自定义模块及编排。

![](https://i.loli.net/2019/03/20/5c924ad896cc5.png)

上图可以看到 Ansible 的组成由 5 个部分组成：
- Ansible ：     ansible核心
- Modules ：    包括 Ansible 自带的核心模块及自定义模块
- Plugins ：      完成模块功能的补充，包括连接插件、邮件插件等
- Playbooks ：   剧本；定义 Ansible 多任务配置文件，由Ansible 自动执行
- Inventory ：    定义 Ansible 管理主机的清单  [ˈɪnvəntri] 清单

# 2. 实战-安装并配置Ansible管理两个节点

ansible服务端: xuegod63 	 192.168.1.63
ansible节点1: xuegod63    192.168.1.63
ansible节点2: xuegod62  	192.168.1.62

### 2.1 服务端安装Ansible
Ansible默认不在yum仓库中，因此我们需要使用下面的命令启用epel仓库。

```
[root@localhost ~]# yum install epel-release -y

[root@localhost ~]# yum install ansible -y 

[root@localhost ~]# ansible --version
ansible 2.7.8
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Oct 30 2018, 23:45:53) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
```
