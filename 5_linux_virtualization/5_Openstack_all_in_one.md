# openstack-allinone-使用方法
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
```
