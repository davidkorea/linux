# 搭建 Kubernetes web管理界面
1. 部署 Kubernetes Dashboard web 界面
2. 在 kubernetes 上面的集群上搭建基于 redis 和 docker 的留言簿案例

# 0. 相关概念

- service 概述：service 是 pod 的路由代理抽象，用于解决 pod 乊间的服务发现问题。因为 pod 的
运行状态可劢态变化(比如切换机器了、缩容过程中被终止了等)，所以访问端丌能以写死 IP 的方式去
访问该 pod 提供的服务。 service 的引入旨在保证 pod 的劢态变化对访问端透明，访问端只需要知
道 service 的地址，由 service 来提供代理。

- service 的三种端口
  - port ： service 暴露在 cluster ip 上的端口，port 是提供给集群内部客户访问 service 的入口。
  - nodePort ：nodePort 是 k8s 提供给集群外部客户访问 service 入口的一种方式。
  - targetPort ：targetPort 是 pod 中容器实例上的端口，从 port 和 nodePort 上到来的数据最终经过 kube-proxy 流入到后端 pod 的 targetPort 上迚入容器。

  port 和 nodePort 都是 service 的端口，前者暴露给集群内客户访问服务，后者暴露给集群外客户访问服务。从这两个端口到来的数据都需要经过反向代理 kube-proxy 流入后端 pod 的 targetPod，从而到达 pod 上的容器内。

- replicationController：是 pod 的复制抽象，用于解决 pod 的扩容缩容问题。通常，分布式应用
为 了 性 能 戒 高可 用 性 的考 虑 ， 需要 复 制 多份 资 源 ，并 且 根 据负 载 情 况劢 态 伸 缩。 通 过
replicationController，我仧可以指定一个应用需要几份复制，Kubernetes 将为每份复制创建一
个 pod，并且保证实际运行 pod 数量总是不该复制数量相等(例如，当前某个 pod 宕机时，自劢创
建新的 pod 来替换)。

总结：service 和 replicationController 只是建立在 pod 乊上的抽象，最终是要作用于 pod 的，
那么它仧如何跟 pod 联系起来呢？这就要引入 label 的概念：label 其实很好理解，就是为 pod 加
上可用于搜索戒关联的一组 key/value 标签，而 service 和 replicationController 正是通过 label
来不 pod 关联的。如下图所示，有三个 pod 都有 label 为"app=backend"，创建 service 和
replicationController 时可以指定同样的 label:"app=backend"，再通过 label selector 机制，
就将它仧不这三个 pod 关联起来了。例如，当有其他 frontend pod 访问该 service 时，自劢会转
发到其中的一个 backend pod。
# 1. 部署 Kubernetes Dashboard web 界面
## 1.1 创建 dashboard-deployment.yaml 配置文件
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
# Keep the name in sync with image version and
# gce/coreos/kube-manifests/addons/dashboard counterparts
  name: kubernetes-dashboard-latest
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
        version: latest
        kubernetes.io/cluster-service: "true"
    spec:
      containers:
      - name: kubernetes-dashboard
        image: docker.io/bestwu/kubernetes-dashboard-amd64:v1.6.3             # 一定要使用这个image，否则报错heapster
        imagePullPolicy: IfNotPresent
        resources:
          # keep request = limit to keep this container in guaranteed class
          limits:
            cpu: 100m
            memory: 50Mi
          requests:
            cpu: 100m
            memory: 50Mi
        ports:
        - containerPort: 9090
        args:
        - --apiserver-host=http://192.168.0.162:8080
        livenessProbe:
          httpGet:
            path: /
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
```
## 1.2 创建dashboard-service.yaml 文件
```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    k8s-app: kubernetes-dashboard
  ports:
  - port: 80
    targetPort: 9090
```
## 1.3 准备 kubernetes 相关的镜像
- 在官方的 dashboard-deployment.yaml 中 定 义 了 dashboard 所 用 的 镜 像 ：gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1，
- 启动 k8s 的 pod 还需要一个额外的镜像：registry.access.redhat.com/rhel7/pod-infrastructure:latest，

可以使用 docker 自带的源先下载下来，每个node保存一份镜像

```
docker.io/a406622768/pod-infrastructure       latest      1158bd68df6d        19 months ago       209 MB
docker.io/bestwu/kubernetes-dashboard-amd64   v1.6.3      691a82db1ecd        20 months ago       139 MB
```

## 1.4 创建 dashboard 的 deployment 和 service

```
[root@server162 ~]# kubectl create -f dashboard-deployment.yaml
deployment "kubernetes-dashboard-latest" created
[root@server162 ~]# kubectl create -f dashboard-service.yaml
service "kubernetes-dashboard" created
```
查看deployment，pods，service，需要参数--all-namespaces 
```
[root@server162 ~]# kubectl get deployment --all-namespaces 
NAMESPACE     NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   kubernetes-dashboard-latest   1         1         1            1           3h
```

```
[root@server162 ~]# kubectl get pod --all-namespaces 
NAMESPACE     NAME                                           READY     STATUS    RESTARTS   AGE
kube-system   kubernetes-dashboard-latest-3739580684-z19p4   1/1       Running   0          3h
```
```
[root@server162 ~]# kubectl get svc --all-namespaces 
NAMESPACE     NAME                   CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
default       kubernetes             10.254.0.1       <none>        443/TCP        23h
kube-system   kubernetes-dashboard   10.254.142.63    <none>        80/TCP         3h
```

删除deployment，pods，service的时候也需要参数-n 来制定namespace，否则删除失败，默认-n是default
```
[root@server162 ~]# kubectl delete svc kubernetes-dashboard -n kube-system
```




