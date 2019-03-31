
node节点中ip的设置有这个文件决定
```
[root@k8s-node1 ~]# cat /run/flannel/subnet.env 
FLANNEL_NETWORK=10.255.0.0/16
FLANNEL_SUBNET=10.255.3.1/24
FLANNEL_MTU=1472
FLANNEL_IPMASQ=false
```
```
[root@k8s-node2 ~]# cat /run/flannel/subnet.env 
FLANNEL_NETWORK=10.255.0.0/16
FLANNEL_SUBNET=10.255.79.1/24
FLANNEL_MTU=1472
FLANNEL_IPMASQ=false
```
