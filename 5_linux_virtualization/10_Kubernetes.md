# 搭建 Kubernetes 容器集群管理系统

- master - 192.168.0.15
- node1  - 192.168.0.16
- node2  - 192.168.0.17
关闭防火墙

1. master + etcd

```
yum install -y kubernetes etcd flannel ntp
```

2. node1 + node2
```
yum install kubernetes flannel ntp -y
```
