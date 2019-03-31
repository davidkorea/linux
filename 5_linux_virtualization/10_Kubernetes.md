# 搭建 Kubernetes 容器集群管理系统

1. Kubernetes 和相关组件的介绍
2. 配置 etcd 和 master 节点
3. 配置 minion1 节点
4. 配置 minion2 节点
# 1. Kubernetes 和相关组件的介绍

Kubernetes 是 Google 开源的容器集群管理系统，基于 Docker 构建一个容器的调度服务，提供资 源调度、均衡容灾、服务注册、劢态扩缩容等功能套件。 基于容器的云平台。Kubernetes 基于 docker 容器的云平台，简写成: k8s 。 openstack 基于 kvm 虚拟机云平台。

Kubernetes 常见组件介绍
![](https://upload.wikimedia.org/wikipedia/commons/b/be/Kubernetes.png)
## 1.1 master
kubernetes管理结点
#### 1. apiserver 
提供接口服务，用户通过 apiserver 来管理整个容器集群平台。API Server 负责 和 etcd 交互(其他组件丌会直接操作 etcd，只有 API Server 这么做)，整个 kubernetes 集群的 所有的交互都是以 API Server 为核心的。如:1、所有对集群迚行的查询和管理都要通过 API 来迚行 2、 所有模块乊间并丌会互相调用，而是通过和 API Server 打交道来完成自己那部分的工作 、API Server 提供的验证和授权保证了整个集群的安全

#### 2. Replication Controllers 
复制， 保证 pod 的高可用。Replication Controller 是 Kubernetes 系统中最有用的功能，实现复制多个 Pod 副本，往往一 个应用需要多个 Pod 来支撑，并且可以保证其复制的副本数，即使副本所调度分配的宿主机出现异常，通 过 Replication Controller 可以保证在其它宿主机吭用同等数量的 Pod。Replication Controller 可以 通过 repcon 模板来创建多个 Pod 副本，同样也可以直接复制已存在 Pod，需要通过 Label selector 来 关联。

#### 3. scheduler kubernetes 
调度服务

## 1.2 etcd 
etcd 存储 kubernetes 的配置信息， 可以理解为是 k8s 的数据库，存储着 k8s 容器 云平台中所有节点、pods、网络等信息。

## 1.3 minion
真正运行容器 container 的物理机。 kubernets 中需要很多 minion 机器，来提供 运算。minion [ˈmɪniən] 爪牙
#### 1. Kube_proxy 
代理 做端口转发，相当于 LVS-NAT 模式中的负载调度器器。Proxy 解决了同一宿主机，相同服务端口冲突的问题，还提供了对外服务的能力，Proxy 后端使用了 随机、轮循负载均衡算法。
#### 2. Pod 
在 Kubernetes 系统中，调度的最小颗粒丌是单纯的容器，而是抽象成一个 Pod，Pod是一个可以被创建、销毁、调度、管理的最小的部署单元。pod 中可以包括一个戒一组容器。 pod [pɒd] 豆荚
#### 3. container 
容器 ，可以运行服务和程序




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
[root@k8s-master ~]# vim /etc/etcd/etcd.conf 
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
检查 etcd 集群成员列表，这里只有一台
```
[root@k8s-master ~]# etcdctl member list
8e9e05c52164694d: name=default peerURLs=http://localhost:2380 clientURLs=http://192.168.0.15:2379 isLeader=true
```
```
[root@k8s-master ~]# etcdctl cluster-health
member 8e9e05c52164694d is healthy: got healthy result from http://192.168.0.15:2379
cluster is healthy
```
## 1.2 master
```
[root@k8s-master ~]# vim /etc/kubernetes/
apiserver           controller-manager  proxy               
config              kubelet             scheduler  
```
### 1. kubernetes 全局配置文件
```diff
[root@k8s-master ~]# vim /etc/kubernetes/config 
  13 KUBE_LOGTOSTDERR="--logtostderr=true"
  
  16 KUBE_LOG_LEVEL="--v=0"
  
  19 KUBE_ALLOW_PRIV="--allow-privileged=false"
  
- 22 KUBE_MASTER="--master=http://127.0.0.1:8080"
+ 22 KUBE_MASTER="--master=http://192.168.0.15:8080"
 ```

### 2. apiserver
```diff

-  8 KUBE_API_ADDRESS="--insecure-bind-address=127.0.0.1"
+  8 KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
  
- 17 KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"
+ 17 KUBE_ETCD_SERVERS="--etcd-servers=http://192.168.0.15:2379"


  20 KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

- 23 KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
+ 23 KUBE_ADMISSION_CONTROL="--admission-control=AlwaysAdmit"
```
```
[root@k8s-master ~]# grep -v ^# /etc/kubernetes/apiserver 
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
KUBE_ETCD_SERVERS="--etcd-servers=http://192.168.0.15:2379"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
KUBE_ADMISSION_CONTROL="--admission-control=AlwaysAdmit"
KUBE_API_ARGS=""
```
- ```KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"``` #监听的接口，如果配置为 127.0.0.1 则只监听 localhost，配置为 0.0.0.0 会监听所有接口，这里配置为 0.0.0.0
- ```KUBE_ETCD_SERVERS="--etcd-servers=http://192.168.1.63:2379"``` #etcd 服务地址，前面 已经启动了 etcd 服务
- ```KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"``` #kubernetes 可以分配的 ip 的范围，kubernetes 启动的每一个 pod 以及 serveice 都会分配一个 ip 地址，将从这个 范围中分配 IP。
- ```KUBE_ADMISSION_CONTROL="--admission-control=AlwaysAdmit"``` #不做限制，允讲所有节点可以访问 apiserver ，对所有请求开绿灯。
### 3. kube-controller-manager 配置文件

默认不需要修改

```
[root@k8s-master ~]# grep -v ^# /etc/kubernetes/controller-manager 
KUBE_CONTROLLER_MANAGER_ARGS=""
```
### 4. kube-scheduler 配置文件

```diff

- 7 KUBE_SCHEDULER_ARGS=""
- + 7 KUBE_SCHEDULER_ARGS="--address=0.0.0.0"     # 监听地址,默认是 127.0.0.1
+ 7 KUBE_SCHEDULER_ARGS="0.0.0.0"                 # 直接写0.0.0.0，否则scheduler服务启动报错
```
### 5. 配置 etcd，指定容器云中 docker 的 IP 网段

```
[root@k8s-master ~]# etcdctl mkdir /k8s/network
[root@k8s-master ~]# etcdctl set /k8s/network/config '{"Network": "10.255.0.0/16"}'
{"Network": "10.255.0.0/16"}
[root@k8s-master ~]#  etcdctl get /k8s/network/config
{"Network": "10.255.0.0/16"}
[root@k8s-master ~]# systemctl restart flanneld
[root@k8s-master ~]# systemctl status flanneld
```

### 6. 配置一下 flanneld 服务

```diff
[root@k8s-master ~]# vim /etc/sysconfig/flanneld 

-  4 FLANNEL_ETCD_ENDPOINTS="http://127.0.0.1:2379"
+  4 FLANNEL_ETCD_ENDPOINTS="http://192.168.0.15:2379"

-  8 FLANNEL_ETCD_PREFIX="/atomic.io/network"
+  8 FLANNEL_ETCD_PREFIX="/k8s/network"

- 11 #FLANNEL_OPTIONS=""
+ 11 FLANNEL_OPTIONS="--iface=ens33"         # 指定通信的物理网卡 
```



# 2. node1
```
yum install kubernetes flannel ntp -y
```
### 1. kubernetes config
```diff
[root@k8s-node1 ~]# vim /etc/kubernetes/config 
  13 KUBE_LOGTOSTDERR="--logtostderr=true"
  
  16 KUBE_LOG_LEVEL="--v=0"
  
  19 KUBE_ALLOW_PRIV="--allow-privileged=false"
  
- 22 KUBE_MASTER="--master=http://127.0.0.1:8080"
+ 22 KUBE_MASTER="--master=http://192.168.0.15:8080"        # 设置同上master节点
 ```
### 2. proxy 同上，不做任何改动，默认就是监听所有 ip

主要是负责 service 的实现，具体来说，就是实现了内部从 pod 到 service
```
[root@k8s-node1 ~]# cat /etc/kubernetes/proxy 
###
# kubernetes proxy config

# default config should be adequate

# Add your own!
KUBE_PROXY_ARGS=""
```
### 3. kubelet
Kubelet 运行在 minion 节点上。Kubelet 组件管理 Pod、Pod 中容器及容器的镜像和卷等信息
```diff
[root@k8s-node1 ~]# vim /etc/kubernetes/kubelet 
-  5 KUBELET_ADDRESS="--address=127.0.0.1"
+  5 KUBELET_ADDRESS="--address=0.0.0.0"

  10 # You may leave this blank to use the actual hostname
- 11 KUBELET_HOSTNAME="--hostname-override=127.0.0.1"
+ 11 KUBELET_HOSTNAME="--hostname-override=k8s-node1"

  13 # location of the api-server
- 14 KUBELET_API_SERVER="--api-servers=http://127.0.0.1:8080"
+ 14 KUBELET_API_SERVER="--api-servers=http://192.168.0.15:8080"

  16 # pod infrastructure container
  17 KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
 ```
 
- ```KUBELET_ADDRESS="--address=0.0.0.0"``` #默认只监听 127.0.0.1，要改成:0.0.0.0，因为后期要使用 kubectl 进程连接到 kubelet 服务上，来查看 pod 及 pod 中容器的状态。如果是 127 就无法进程连接 kubelet 服务。
- ```KUBELET_HOSTNAME="--hostname-override=node1"``` # minion 的主机名，设置 成和本主机机名一样，便于识别。
- ```KUBELET_API_SERVER="--api-servers=http://192.168.1.63:8080"``` #批定 apiserver 的地址
- ```KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redh at.com/rhel7/pod-infrastructure:latest"``` infrastructure [ˈɪnfrəstrʌktʃə(r)] 基础设施 KUBELET_POD_INFRA_CONTAINER 指定 pod 基础容器镜像地址。这个是一个基础容器，每一个Pod 启动的时候都会启动一个这样的容器。如果你的本地没有这个镜像，kubelet 会连接外网把这个镜像 下载下来。最开始的时候是在 Google 的 registry 上，因此国内因为 GFW 都下载丌了导致 Pod 运行丌 起来。现在每个版本的 Kubernetes 都把这个镜像地址改成红帽的地址了。你也可以提前传到自己的 registry 上，然后再用这个参数指定成自己的镜像链接。
 
### 4. 启动服务
```
[root@k8s-node1 ~]# systemctl restart flanneld kube-proxy kubelet docker
[root@k8s-node1 ~]# systemctl enable flanneld kube-proxy kubelet docker
[root@k8s-node1 ~]# systemctl status flanneld kube-proxy kubelet docker
```
```
[root@k8s-node1 ~]# ifconfig 
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.255.3.1  netmask 255.255.255.0  broadcast 0.0.0.0

flannel0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1472
        inet 10.255.3.0  netmask 255.255.0.0  destination 10.255.3.0
```

```
[root@k8s-node1 ~]# netstat -anutp | grep proxy
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      15630/kube-proxy    
tcp        0      0 192.168.0.16:51182      192.168.0.15:8080       ESTABLISHED 15630/kube-proxy    
tcp        0      0 192.168.0.16:51180      192.168.0.15:8080       ESTABLISHED 15630/kube-proxy    
```
 
# node2

```
[root@k8s-node1 ~]# scp /etc/sysconfig/flanneld 192.168.0.17:/etc/sysconfig/flanneld 
```
```
[root@k8s-node1 ~]# scp /etc/kubernetes/config 192.168.0.17:/etc/kubernetes/
```
```diff
[root@k8s-node1 ~]# scp /etc/kubernetes/kubelet 192.168.0.17:/etc/kubernetes/

[root@k8s-node2 ~]# vim /etc/kubernetes/kubelet 
- 11 KUBELET_HOSTNAME="--hostname-override=k8s-node1"
+ 11 KUBELET_HOSTNAME="--hostname-override=k8s-node2"
```
```
[root@k8s-node2 ~]# systemctl restart flanneld kube-proxy kubelet docker
[root@k8s-node2 ~]# systemctl enable flanneld kube-proxy kubelet docker
[root@k8s-node2 ~]# systemctl status flanneld kube-proxy kubelet docker
```
```
[root@k8s-node2 ~]# ifconfig 
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.255.79.1  netmask 255.255.255.0  broadcast 0.0.0.0

flannel0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1472
        inet 10.255.79.0  netmask 255.255.0.0  destination 10.255.79.0
        
[root@k8s-node2 ~]# ping 10.255.3.1                         # ping node1的docker网段正常
PING 10.255.3.1 (10.255.3.1) 56(84) bytes of data.
64 bytes from 10.255.3.1: icmp_seq=1 ttl=62 time=1.02 ms
64 bytes from 10.255.3.1: icmp_seq=2 ttl=62 time=1.83 ms
```
go back to master 192.168.0.15查看整个集群的运行状态
```
[root@k8s-master ~]# kubectl get nodes
NAME        STATUS    AGE
k8s-node1   Ready     18m
k8s-node2   Ready     1m
```











