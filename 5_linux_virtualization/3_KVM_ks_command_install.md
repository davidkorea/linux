# 字符界面安装 KVM-阿里云主机使用方法 

1. 在命令行安装 KVM 虚拟机 
2. 在命令行无人执守安装 KVM 虚拟机 
3. 阿里云主机使用方法 


```
virt-install --name centos7_ks --ram 1024 --vcpus=1 --disk path=/var/lib/libvirt/images/centos7_ks.qcow2,size=13 --accelerate --location=http://192.168.0.162/centos7/ --network bridge=br0 -x "ks=http://192.168.0.162/ks.cfg"
```
