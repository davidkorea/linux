# 搭建 Kubernetes web管理界面
1. 部署 Kubernetes Dashboard web 界面
2. 在 kubernetes 上面的集群上搭建基于 redis 和 docker 的留言簿案例

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
