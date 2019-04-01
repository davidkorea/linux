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
## 1.2 nodes
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

## 2.1 所有节点准备镜像 docker.io/nginx:latest

## 2.2 创建 kubectl run
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

## 2.3 删除kubectl delete
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


# 3. kubectl create加载yaml文件生成deployment

使用 kubectl run 在设定很复杂的需求时，需要非常长的一条诧句，也很容易出错，也没法保存。 所以更多场景下会使用 yaml 戒者 json 文件

上传镜像至所有node，```docker pull docker.io/mysql/mysql-server ```

## 3.1 生成 mysql-deployment.yaml 文件

```ymal
[root@server162 ~]# vim mysql-deployment.yaml 
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: mysql
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: mysql
    spec:
      containers:
      - name: mysql
        image: docker.io/mysql/mysql-server
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3306
          protocol: TCP
        env:
          - name: MYSQL_ROOT_PASSWORD
            value: "hello123"
```









































