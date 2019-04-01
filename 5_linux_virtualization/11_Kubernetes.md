# 使用kubectl命令管理Kubernetes容器平台

1. kubectl 创建和删除一个 pod 相关操作
3. yaml 诧法规则
4. kubectl create 加载 yaml 文件生成 deployment 设备资源
5. kubectl 其他常用命令和参数说明
6. 使用 kubectl 管理集群中 deployment 资源和 service 服务


# 1. 环境准备

## 1.1 master
#### 1. 启动服务
```systemctl restart etcd kube-apiserver kube-controller-manager kube-scheduler flanneld```





# 2. kubectl命令创建和删除一个 pod 
kubectl 是一个用亍操作 kubernetes 集群的命令行接口，通过利用 kubectl 各种功能

- 192.168.0.162 master
- 192.168.0.163 node1
- 192.168.0.164 node2





















1. node节点中ip的设置有这个文件决定
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
2. master run a deployment
node1 and node2 should have 2 images
  - docker.io/nginx
  - pod-infrastucture
master node don't need to have these 2 images

```
[root@k8s-master ~]# docker images
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
a406622768/pod-infrastructure                         latest              1158bd68df6d        19 months ago       209 MB
registry.access.redhat.com/rhel7/pod-infrastructure   latest              1158bd68df6d        19 months ago       209 MB
[root@k8s-master ~]# kubectl run nginx --image=docker.io/nginx --replicas=1 --port=9000
deployment "nginx" created
[root@k8s-master ~]# kubectl get deployment 
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     1         1         1            1           14s

[root@k8s-master ~]# kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
nginx-2187705812-gz0fs   1/1       Running   0          42s
[root@k8s-master ~]# kubectl get pods -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-2187705812-gz0fs   1/1       Running   0          50s       10.255.3.2   k8s-node1
```

如果使用自己的pod image, docker.io/a406622768/pod-infrastructure:latest ，需要修改node的kubelet配置文件
否则一直会创建中
```
[root@server162 ~]# kubectl get pod
NAME                      READY     STATUS              RESTARTS   AGE
nginx-2187705812-ft855    0/1       Completed           0          19m      # 这个状态也不正常，应该是running
nginx2-951959098-vh4ss    0/1       Completed           0          12m
nginx3-1206369853-cks8d   0/1       ContainerCreating   0          2m
```
```diff

- 17 KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
+ 17 KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=docker.io/a406622768/pod-infrastructure:latest" 
```
```
[root@server162 ~]# kubectl get pod
NAME                      READY     STATUS              RESTARTS   AGE
nginx-2187705812-ft855    0/1       Completed           0          22m
nginx2-951959098-vh4ss    0/1       Completed           0          16m
nginx3-1206369853-cks8d   0/1       ContainerCreating   0          6m
[root@server162 ~]# kubectl get pod
NAME                      READY     STATUS    RESTARTS   AGE
nginx-2187705812-ft855    1/1       Running   1          23m
nginx2-951959098-vh4ss    1/1       Running   1          16m
nginx3-1206369853-cks8d   1/1       Running   0          6m
```
3. delete pod
```
[root@server162 ~]# kubectl delete pod nginx3-1206369853-cks8d
pod "nginx3-1206369853-cks8d" deleted
[root@server162 ~]# kubectl get pod
NAME                      READY     STATUS              RESTARTS   AGE
nginx-2187705812-ft855    1/1       Running             1          25m
nginx2-951959098-vh4ss    1/1       Running             1          18m
nginx3-1206369853-z4r5r   0/1       ContainerCreating   0          3s
[root@server162 ~]# kubectl get pod
NAME                      READY     STATUS    RESTARTS   AGE
nginx-2187705812-ft855    1/1       Running   1          25m
nginx2-951959098-vh4ss    1/1       Running   1          18m
nginx3-1206369853-z4r5r   1/1       Running   0          11s
```
4. delete deploymet
```
[root@server162 ~]# kubectl delete deployment nginx3
deployment "nginx3" deleted
[root@server162 ~]# kubectl get deployment 
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     1         1         1            1           26m
nginx2    1         1         1            1           19m
[root@server162 ~]# kubectl get pod
NAME                     READY     STATUS    RESTARTS   AGE
nginx-2187705812-ft855   1/1       Running   1          26m
nginx2-951959098-vh4ss   1/1       Running   1          19m
```
