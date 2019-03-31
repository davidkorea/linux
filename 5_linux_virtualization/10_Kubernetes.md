# 搭建 Kubernetes 容器集群管理系统

- master - 192.168.0.15
- node1  - 192.168.0.16
- node2  - 192.168.0.17

- 关闭防火墙
- hostname
```
[root@k8s-master ~]# cat /etc/hostname 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.15 k8s-master
192.168.0.15 k8s-etcd
192.168.0.16 k8s-node1
192.168.0.17 k8s-node2
```



# 1. master + etcd
```
yum install -y kubernetes etcd flannel ntp
```
## 1.1 etcd

```diff

   3 ETCD_DATA_DIR="/var/lib/etcd/default.etcd"

-  6 ETCD_LISTEN_CLIENT_URLS="http://localhost:2379"
+  6 ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://192.168.0.15:2379"

   9 ETCD_NAME="default"      # default可以改成ectd

- 21 ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"
+ 21 ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.15:2379"
```
```
[root@k8s-master ~]# grep -v ^# /etc/etcd/etcd.conf 
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://192.168.0.15:2379"
ETCD_NAME="default"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.15:2379"
```
- ```ETCD_NAME="etcd"``` etcd 节点名称，如果 etcd 集群只有一台 etcd，这一项可以注释丌用配置，默认名称为 default，这个名字后面会用到。
- ```ETCD_DATA_DIR="/var/lib/etcd/default.etcd"```etcd 存储数据的目录 
- ```ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://192.168.1.63:2379" ```etcd 对外服务监听地址，一般指定 2379 端口，如果为 0.0.0.0 将会监听所有接口 
- ```ETCD_ARGS=""```需要额外添加的参数，可以自己添加，etcd 的所有参数可以通过 etcd -h 查看。

```
[root@k8s-master ~]# systemctl start etcd
[root@k8s-master ~]# systemctl enable etcd
```
```
[root@k8s-master ~]# netstat -anutp | grep 2379
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      14664/etcd          
tcp        0      0 192.168.0.15:2379       0.0.0.0:*               LISTEN      14664/etcd          
tcp        0      0 192.168.0.15:2379       192.168.0.15:59138      ESTABLISHED 14664/etcd          
tcp        0      0 127.0.0.1:47070         127.0.0.1:2379          ESTABLISHED 14664/etcd          
tcp        0      0 192.168.0.15:59138      192.168.0.15:2379       ESTABLISHED 14664/etcd          
tcp        0      0 127.0.0.1:2379          127.0.0.1:47070         ESTABLISHED 14664/etcd  
```











# 2. node1 + node2
```
yum install kubernetes flannel ntp -y
```
