# openstack-allinone-使用方法

1. 安装 OpenStack 客户端并创建一个cirros demo1云主机
2. 查看创建好的 openstack 项目中的信息和云主机网络连通性
3. openstack web 界面使用方法



# 1. 安装 OpenStack 客户端并创建一个cirros demo1云主机
## 1.1 安装 OpenStack client 端
安装 OpenStack client 端，方便后期使用命令行操作 openstack

#### 1. pip install python-openstackclient
```
[root@server15 ~]# pip install python-openstackclient 
Found existing installation: PyYAML 3.10
Cannot uninstall 'PyYAML'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.

[root@server15 ~]# pip install PyYAML --ignore-installed PyYAML

[root@server15 ~]# pip install python-openstackclient        # 再次安装
Found existing installation: ipaddress 1.0.16
Cannot uninstall 'ipaddress'. It is a distutils installed project and thus we cannot
accurately determine which files belong to it which would lead to only a partial uninstall. 

[root@server15 ~]# pip install ipaddress --ignore-installed ipaddress 

[root@server15 ~]# pip install python-openstackclient        # 再次安装 ok
```
#### 2. pip install python-neutronclient
安装 openstack 网络相关的命令
```
[root@server15 ~]# pip install python-neutronclient 
[root@server15 ~]# pip install pyinotify --ignore-installed pyinotify 
[root@server15 ~]# pip install python-neutronclient         # 再次安装成功。
```
## 1.2 使用 init-runonce 脚本创建一个 openstack 云项目
#### 1. 修改 init-runonce 脚本
修改 init-runonce 脚本，指定浮劢 IP 地址范围，浮劢 IP 就是云主机的公网 IP。init-runonce 是在 openstack 中快速创建一个云项目例子的脚本。

```diff
- 12 EXT_NET_CIDR='10.0.2.0/24'
- 13 EXT_NET_RANGE='start=10.0.2.150,end=10.0.2.199' 
- 14 EXT_NET_GATEWAY='10.0.2.1'
 
+ EXT_NET_CIDR='192.168.0.0/24' 
+ EXT_NET_RANGE='start=192.168.0.100,end=192.168.0.200' 
+ EXT_NET_GATEWAY='192.168.0.1'
```
#### 2. 使用 init-runonce 脚本创建一个 openstack 云项目
```
[root@server15 ~]# source /etc/kolla/admin-openrc.sh  # 先加载这个文件，把文件中的环境变量加入系统中，才有权限执行下面的命令
[root@server15 ~]# cd /usr/share/kolla-ansible 
[root@server15 kolla-ansible]# ./init-runonce         # 最后弹出以下命令，复制后直接执行即可

Done.

To deploy a demo instance, run:

openstack server create \
    --image cirros \
    --flavor m1.tiny \
    --key-name mykey \
    --nic net-id=3b04f928-a1bb-4e6f-8fd8-6c7f31261ca5 \
    demo1

[root@server15 kolla-ansible]# openstack server create \   
>     --image cirros \
>     --flavor m1.tiny \
>     --key-name mykey \
>     --nic net-id=3b04f928-a1bb-4e6f-8fd8-6c7f31261ca5 \
>     demo1

```





















# 1. 
```
pip install python-openstackclient 
 
Cannot uninstall 'ipaddress'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.
[root@server162 ~]# echo $?
1

[root@server162 ~]# pip install ipaddress --ignore-installed ipaddress
DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that date. A future version of pip will drop support for Python 2.7.
Collecting ipaddress
  Using cached https://files.pythonhosted.org/packages/fc/d0/7fc3a811e011d4b388be48a0e381db8d990042df54aa4ef4599a31d39853/ipaddress-1.0.22-py2.py3-none-any.whl
Installing collected packages: ipaddress
Successfully installed ipaddress-1.0.22
```
# 2. 
```
[root@xuegod63 kolla-ansible]# ./init-runonce
。。。 。。。
    from decorator import decorate
ImportError: cannot import name decorate

Done.

To deploy a demo instance, run:

openstack server create \
    --image cirros \
    --flavor m1.tiny \
    --key-name mykey \
    --nic net-id= \
    demo1
```
```
pip install decorate        # no use
pip install decorator       # iinstalled already, no use

pip install -U decorator    # Fixed！！！！！ 
# 或者
pip install --ignore-installed  python-openstackclient 
```
