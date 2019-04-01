# 使用kubectl命令管理Kubernetes容器平台

1. kubectl 创建和删除一个 pod 相关操作
3. yaml 诧法规则
4. kubectl create 加载 yaml 文件生成 deployment 设备资源
5. kubectl 其他常用命令和参数说明
6. 使用 kubectl 管理集群中 deployment 资源和 service 服务


# 1. 环境准备

### 1.1 master
#### 1. 启动服务
```systemctl restart etcd kube-apiserver kube-controller-manager kube-scheduler kube-proxy flanneld```
重启kube-proxy，以防止docker0和flannel0不在同一网段
### 1.2 nodes
#### 1. 启动服务
```systemctl restart flanneld kube-proxy kubelet docker```

#### 2. 所有node - 准备pod基础镜像
```
[root@client164 ~]# docker images
REPOSITORY                                TAG                 IMAGE ID            CREATED             SIZE
docker.io/a406622768/pod-infrastructure   latest              1158bd68df6d        19 months ago       209 MB
```
- 因为使用自己的pod image, docker.io/a406622768/pod-infrastructure:latest ，需要修改node的kubelet配置文件，否则一直会创建中ContainerCreating
```diff

- 17 KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
+ 17 KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=docker.io/a406622768/pod-infrastructure:latest" 
```
- 或者将自己的镜像重命名为registry.access.redhat.com/rhel7/pod-infrastructure:latest

#### 3. 其他应用镜像
和pod infrastructure镜像一样。所有node，都必须有一份

# 2. kubectl命令创建和删除 deployment，pod 
kubectl 是一个用亍操作 kubernetes 集群的命令行接口，通过利用 kubectl 各种功能

- 192.168.0.162 master
- 192.168.0.163 node1
- 192.168.0.164 node2

### 2.1 所有节点准备镜像 docker.io/nginx:latest

### 2.2 创建 kubectl run
```
[root@server162 ~]# kubectl run nginx --image=docker.io/nginx --replicas=1 --port=9000
deployment "nginx" created
```
kubectl run 之后，kubernetes 创建了一个 deployment。查看 Deployment
```
[root@server162 ~]# kubectl get deployment 
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     1         1         1            1           10s
```
查看生成的 pod，kubernetes 将容器运行在 pod 中以方便实施卷和网络共享等管理
```
[root@server162 ~]# kubectl get pod -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP            NODE
nginx-2187705812-60s2p   1/1       Running   0          2m        10.255.65.2   node1
```
- pods 常见的状态：
  - ContainerCreating #容器创建
  - ImagePullBackOff #从后端把镜像拉取到本地。如果这里 pod 没有正常运行，都是因为 docker hub 没有连接上，导致镜像没有下载成功，这时，可以在 node 节点上把相关镜像手动上传，或把 docker 源换成阿里的
  - terminating ['tɜ:mɪneɪtɪŋ] #终止 。当删除 pod 时的状态
  - Running 正常运行状态

### 2.3 删除kubectl delete
#### 1. kubectl delete pod
```
[root@server162 ~]# kubectl delete pod nginx-2187705812-60s2p
pod "nginx-2187705812-60s2p" deleted

[root@server162 ~]# kubectl get pod -o wide
NAME                     READY     STATUS              RESTARTS   AGE       IP        NODE
nginx-2187705812-lpvdl   0/1       ContainerCreating   0          2s        <none>    node1
```
可以看到刚刚生成的 nginx pod 正在结束(Terminating）,随之一个新的 nginx pod 正在创建，这是正是 replicas 为 1 的作用，平台会一直保证有一个副本在运行。
#### 2. kubectl delete deployment
```
[root@server162 ~]# kubectl delete deployment nginx 
deployment "nginx" deleted
[root@server162 ~]# kubectl get pod -o wide
No resources found.
[root@server162 ~]# kubectl get deployment 
No resources found.
```
直接删除 pod 触发了 replicas 的确保机制，所以直接删除 deployment，同时也删除掉了pod


# 3. 加载yaml文件生成deployment

使用 kubectl run 在设定很复杂的需求时，需要非常长的一条诧句，也很容易出错，也没法保存。 所以更多场景下会使用 yaml 戒者 json 文件

上传镜像至所有node，```docker pull docker.io/nginx```

## 3.1 生成 nginx-deployment.yaml资源 和 nginx-svc.yaml服务配置文件
#### 1. nginx-deployment.yaml 
```
[root@server162 ~]# vim nginx-deployment.yaml 
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nginx
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: nginx
    spec:
      containers:
      - name: nginx
        image:  docker.io/nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          protocol: TCP
```
#### 2. nginx-svc.yaml 
```
[root@server162 ~]# vim nginx-svc.yaml 
kind: Service
apiVersion: v1
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  type: NodePort
  ports:
  - protocol: TCP
    nodePort: 31001   # 用户可以通过node节点上这个端口访问nginx，物理机的公网上的端口
    targetPort: 80    # 指定 nginx docker 容器的端口
    port: 80          # 指定 pod 端口
  selector:
    name: nginx
```
- 服务端口的定义：
  - nodePort: 31001
  - port: 80 # pod 端口号定义
  - targetPort: 80 #指定是 nginx docker 容器的端口

## 3.2 使用yaml创建 deployment 和 serveice
```
[root@server162 ~]# kubectl create -f nginx-deployment.yaml         # 创建deployment，相当于创建设备硬件资源	
deployment "nginx" created

[root@server162 ~]# kubectl create -f nginx-svc.yaml                # 创建service
service "nginx" created

[root@server162 ~]# kubectl get pod -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP            NODE
nginx-1011335894-vfpqt   1/1       Running   0          19s       10.255.65.2   node1
[root@server162 ~]# kubectl get svc

NAME         CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   10.254.0.1     <none>        443/TCP        54s
nginx        10.254.86.32   <nodes>       80:31001/TCP   18s
```

> **排错**：之前测试的时候已经用来31001端口。但是删除的时候只是删除了deployment，service并没有删除
> ```
> [root@server162 ~]# kubectl create -f nginx-svc.yaml 
> The Service "nginx" is invalid: spec.ports[0].nodePort: Invalid value: 31001: provided port is already allocated
> ```
> ```[root@server162 ~]# kubectl delete svc nginx ```，删除即可


> **排错**：docker0 和 flannel0 不在同一网段
> ```
>  [root@k8s-master ~]# ifconfig 
>  docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
>          inet 10.255.31.1  netmask 255.255.255.0  broadcast 0.0.0.0
>
>  ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
>          inet 192.168.0.15  netmask 255.255.255.0  broadcast 192.168.0.255
>
>  flannel0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1472
>          inet 10.255.63.0  netmask 255.255.0.0  destination 10.255.63.0
>  ```
> 检查了所有配置文件包括flannel和etcd以及所有kubernetes，全部正确。最后master重启服务kube_proxy后正常.
> ```
>  [root@k8s-master flannel]# systemctl restart flanneld kube-proxy kubelet docker
>  [root@k8s-master flannel]# ifconfig 
>  docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
>          inet 10.255.63.1  netmask 255.255.255.0  broadcast 0.0.0.0
>
>  flannel0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1472
>          inet 10.255.63.0  netmask 255.255.0.0  destination 10.255.63.0
>  ```


**注：如果只是创建 deployment设备硬件资源，没有对应 service 服务，不能直接在外网迚行访问 Nginx服务**

此时访问任何一个node都可以正常打开nginx页面http://192.168.0.163:31001/ ， http://192.168.0.164:31001/

尽管nginx的pod是在node1运行的，但我们去访问任意 node，都可以正常访问 ginx的。已经做了负载均衡，所有node可以成功访问 web 服务

测试更改nginx的主页/usr/share/nginx/html/index.html
```
[root@k8s-master ~]# kubectl exec -it nginx-1011335894-k3qzs bash
/index.html 011335894-k3qzs:/# echo "hello kubernetes!" >> /usr/share/nginx/html/
```
![](https://i.loli.net/2019/04/01/5ca2312c45c7f.png)
## 3.3 kubectl命令

#### 0. 创建测试环境
- ```docker pull docker.io/mysql/mysql-server ```
- 生成 mysql-deployment.yaml 文件
```ymal
[root@server162 ~]# vim mysql-deployment.yaml 
kind: Deployment                             # 使用deployment创建一个pod，旧版本可以使用kind: ReplicationController
apiVersion: extensions/v1beta1
metadata:
  name: mysql                                # deployment 的名称，全尿唯一
spec:
  replicas: 1                                # Pod副本期待数量=1 表示运行一个pod，里面一个容器
  template:                                  # 根据此模板创建 Pod 的副本（实例）
    metadata:
      labels:                                # 符合目标的Pod拥有此标签。默认和 name 的值一样
        name: mysql                          # 定义 Pod 的名字是 mysql
    spec:                                    # Pod 中容器的定义部分
      containers:
      - name: mysql                          # 容器的名称
        image: docker.io/mysql/mysql-server  # 容器对应的 Docker Image 镜像
        imagePullPolicy: IfNotPresent        # 如果本地存在镜像就优先使用本地镜像
        ports:
        - containerPort: 3306                # 容器暴露的端口号
          protocol: TCP
        env:                                 # 注入到容器的环境变量
          - name: MYSQL_ROOT_PASSWORD        # 设置 mysql root 的密码
            value: "hello123"
```
- imagePullPolicy:
  - 默认值为：imagePullPolicy: Always 一直从外网，下载镜像，不使用本地的
  - IfNotPresent ：如果本地存在镜像就优先使用本地镜像。 这样可以直接使用本地镜像
  - Never：不再去拉取镜像了，使用本地的，如果本地不存在就报异常了

- 使用 mysql-deployment.yaml创建mysql资源
```kubectl create -f mysql-deployment.yaml```


#### 1. ```kubectl get``` 命令
- kubectl get deployments (缩写 deploy)
- kubectl get events (缩写 ev)
- kubectl get namespaces (缩写 ns)
- kubectl get nodes (缩写 no)
- kubectl get pods (缩写 po)
- kubectl get replicasets (缩写 rs)
- kubectl get replicationcontrollers (缩写 rc)
- kubectl get services (缩写 svc)
#### 2. ```kubectl describe``` 命令
- kubectl describe pod pod的名字
- kubectl describe node node的名字
- kubectl describe deployment deployment的名字
#### 3. 其他命令
- kubectl logs 取得 pod 中容器的 log 信息
- kubectl exec -it 在 pod 中执行一条命令
  ```
  [root@server162 ~]# kubectl exec -it mysql-2261771434-d1m98 bash
  bash-4.2# cat /etc/my.cnf
  # For advice on how to change settings please see
  # http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html
  [mysqld]
  ... ... 
  ```    
- kubectl cp 从容器拷出 或 向容器拷入文件。**容器内需要安装tar命令**
  ```
  [root@server162 ~]# kubectl cp mysql-2261771434-d1m98:/etc/my.cnf /tmp/my.cnf     # container -> physical
  error: unexpected EOF
  
  [root@server162 ~]# kubectl exec -it mysql-2261771434-d1m98 bash
  bash-4.2# yum install -y tar
  
  [root@server162 ~]# kubectl cp mysql-2261771434-d1m98:/etc/my.cnf /tmp/my.cnf
  tar: Removing leading `/' from member names
  [root@server162 ~]# ll /tmp/
  总用量 4
  -rw-r--r-- 1 root root 1239 4月   1 16:28 my.cnf
  ```
  ```
  [root@server162 ~]# kubectl cp /etc/hosts mysql-2261771434-d1m98:/tmp/hosts       # physical -> container
  [root@server162 ~]# kubectl exec -it mysql-2261771434-d1m98 bash
  bash-4.2# ls /tmp/
  hosts
  ```
- kubectl attach  实时查看运行中的容器消息


# 4. 使用 kubectl 管理service 服务
### 1. 查看service的详细信息
```
[root@k8s-master ~]# kubectl get service -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: 2019-04-01T15:04:33Z
    labels:
      component: apiserver
      provider: kubernetes
    name: kubernetes
    namespace: default
    resourceVersion: "26711"
    selfLink: /api/v1/namespaces/default/services/kubernetes
    uid: 74eefc46-548f-11e9-86e4-000c2954fad5
  spec:
    clusterIP: 10.254.0.1
    ports:
    - name: https
      port: 443
      protocol: TCP
      targetPort: 6443
    sessionAffinity: ClientIP
    type: ClusterIP
  status:
    loadBalancer: {}
- apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: 2019-04-01T15:08:08Z
    labels:
      name: nginx
    name: nginx
    namespace: default
    resourceVersion: "27089"
    selfLink: /api/v1/namespaces/default/services/nginx
    uid: f58cfc87-548f-11e9-86e4-000c2954fad5
  spec:
    clusterIP: 10.254.212.24
    ports:
    - nodePort: 31001
      port: 80
      protocol: TCP
      targetPort: 80
    selector:
      name: nginx
    sessionAffinity: None
    type: NodePort
  status:
    loadBalancer: {}
kind: List
metadata: {}
resourceVersion: ""
selfLink: ""
```
### 2. 修改访问端口31001 为31002
#### i. kubectl edit service
```
[root@k8s-master ~]# kubectl edit service nginx 
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2019-04-01T15:08:08Z
  labels:
    name: nginx
  name: nginx
  namespace: default
  resourceVersion: "27089"
  selfLink: /api/v1/namespaces/default/services/nginx
  uid: f58cfc87-548f-11e9-86e4-000c2954fad5
spec:
  clusterIP: 10.254.212.24
  ports:
  - nodePort: 31001       # 改为31002
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    name: nginx
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```
http://192.168.0.16:31002/，访问成功。edit 编辑修改配置文件时，不需要停止服务。改完后立即生效

```
[root@k8s-master ~]# kubectl get service
NAME         CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   10.254.0.1      <none>        443/TCP        21m
nginx        10.254.212.24   <nodes>       80:31002/TCP   17m
```
#### ii. kubectl apply
apply 命令是用来使用文件或者标准输入来更改配置信息
```
[root@master ~]# vim nginx-svc.yaml
改: nodePort: 31001
为: nodePort: 31004
```
执行 apply 命令，执行设定文件可以在运行状态修改 port 信息
```
[root@master ~]# kubectl apply -f nginx-svc.yaml service "nginx" configured
[root@master ~]# kubectl get svc
 NAME kubernetes nginx 已经改变。
```
#### iii. kubectl replace
replace也可以更改port，我觉着是三个方法中最难用的一个
```
kubectl get service nginx -o yaml > nginx_replace.yaml  # 导出配置文件
kubectl replace -f nginx_replace.yaml                   # 更改好后，使用replace进行更改
``` 
### 3. kubectl patch 换运行镜像

当修改一部分设定时，使用 patch 很方便。比如:给 pod 换个 image 镜像

#### i. 查看当前镜像是否可以解析php
```
[root@k8s-master ~]# kubectl exec -it nginx-1011335894-k3qzs bash
root@nginx-1011335894-k3qzs:/# nginx -v
nginx version: nginx/1.15.10
root@nginx-1011335894-k3qzs:/# php
bash: php: command not found        # 果然不能解析php
```
#### ii. 更换可以解析php的镜像
```
[root@k8s-node1 ~]# docker pull docker.io/richarvey/nginx-php-fpm     # 所有节点下载镜像
[root@k8s-node2 ~]# docker pull docker.io/richarvey/nginx-php-fpm

[root@k8s-master ~]# kubectl patch pod nginx-1011335894-k3qzs -p '{"spec":{"containers":[{"name":"nginx","image":"docker.io/richarvey/nginx-php-fpm:latest"}]}}'

"nginx-1011335894-k3qzs" patched
[root@k8s-master ~]# kubectl exec -it nginx-1011335894-k3qzs bash
bash-4.4# php -v
PHP 7.3.3 (cli) (built: Mar  9 2019 01:07:11) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.3, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.3.3, Copyright (c) 1999-2018, by Zend Technologies
bash-4.4# nginx -v
nginx version: nginx/1.14.2
```
http://192.168.0.16:31002/ 访问php页面成功
![](https://i.loli.net/2019/04/01/5ca2312c5c240.png)


## 4. kubectl scale
scale 命令用亍横向扩展，是 kubernetes 或 swarm 这类容器编辑平台的重要功能之一

- 之前已经设定 nginx 的 replica 副本为 1
```
[root@k8s-master ~]# kubectl get pod -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP            NODE
nginx-1011335894-k3qzs   1/1       Running   1          44m       10.255.16.2   k8s-node2
```
- 执行scale
```
[root@k8s-master ~]# kubectl scale --current-replicas=1 --replicas=3 deployment/nginx
deployment "nginx" scaled
[root@k8s-master ~]# kubectl get pod -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP            NODE
nginx-1011335894-k3qzs   1/1       Running   1          45m       10.255.16.2   k8s-node2
nginx-1011335894-61tdw   1/1       Running   0          3s        10.255.35.2   k8s-node1
nginx-1011335894-xbp7p   1/1       Running   0          3s        10.255.16.3   k8s-node2
nginx-1011335894-k3qzs   1/1       Running   1          45m       10.255.16.2   k8s-node2
```
```
[root@k8s-master ~]# kubectl get deployment 
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     3         3         3            3           47m
```







