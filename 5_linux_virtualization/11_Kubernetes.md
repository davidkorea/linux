
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
