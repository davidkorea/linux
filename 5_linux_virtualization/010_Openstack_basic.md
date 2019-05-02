

- 30-50 servers no need to implement openstack, need technical supports for openstack, devops chanllege
- over 200 servers ok
- 一般来讲，云主机的 内存，cpu都是同一台物理主机提供，而存储一般也是该主机一同来提供。
  - 因为如果使用共享存储，会大大依赖于网络IO，否则延迟会很大
- 但是，一个云主机如果关机，所有cpu，内存，存储资源都将回收，下次有可能再启动在其他物理主机。
  - 所以该虚拟机的磁盘镜像文件，即使存在某个物理服务器上也没有用
  - 所以，应该尽可能不实用磁盘镜像文件的方式来实现存储功能，而是单独使用一个卷存储
  - 这样每次虚拟机开机，都可以让不同的物理节点通过ISCSI去调用该卷存储中的相应存储空间，并复制到本地来运行
