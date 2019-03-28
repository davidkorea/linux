# 部署 docker 容器虚拟化平台
1. Docker 概述
2. 部署 docker 容器虚拟化平台
3. docker 平台基本使用方法
4. docker 镜像制作和发布方法
5. Container 容器端口映射

# 1. Dcoker 概述
## 1.1 Dcoker 概述
Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙盒机制，相互之间不会有任何接口(类似 iPhone 的 app)。几乎没有性能开销,可以徆容易地在机器和数据中心中运行。最 重要的是,他们不依赖于任何语言、框架戒包装系统。

Docker 是 dotCloud 公司开源的一个基于 LXC 的高级容器引擎，源代码托管在 Github 上, 基于 go 语言并遵从 Apache2.0 协议开源。Docker 让开发者可以打包他们的应用以及依赖包到一个可移植的 container 中，然后发布到任何流 行的 Linux 机器上。

Docker 核心技术
1. Namespace — 实现 Container 的进程、网络、消息、文件系统和主机名的隔离。
2. Cgroup — 实现对资源的配额和度量。Cgroup 的配额，可以挃定实例使用的 cpu 个数，内存大小等。就像vmware 虚拟机中的硬件配置参数。

- 扩展:沙盒
  - 沙盒也叫沙箱，英文 sandbox。在计算机领域挃一种虚拟技术，且多用于计算机安全技术。安全软件 可以先让它在沙盒中运行，如果含有恶意行为，则禁止程序的进一步运行，而这丌会对系统造成任何危害。
  
- 扩展:LXC
  - LXC 为 Linux Container 的简写。Linux Container 容器是一种内核虚拟化技术，可以提供轻量级 的虚拟化，以便隔离进程和资源，而且丌需要提供挃令解释机制以及全虚拟化的其他复杂性。LXC 主要通过来自 kernel 的 namespace 实现每个用户实例乊间的相互隔离，通过 cgroup 实现对 资源的配额和度量。

![](https://i.loli.net/2019/03/27/5c9b87c0bbb78.png)

![](https://i.loli.net/2019/03/27/5c9b880c7c834.png)

- 工作流程:服务器 A 上运行 docker Engine 服务，在 docker Engine上启多容器 container ， 从外网 Docker Hub 上把 image 操作系统镜像下载来，放到 container 容器运行。这样一个容器的实例 就运行起来了。最后，通过 Docker client 对 docker 容器虚拟化平台进行控制。

- Image 和 Container 的关系:image 可以理解为一个系统镜像，Container 是 Image 在运行时的 一个状态。
- 如果拿虚拟机作一个比喻的话，**Image 就是关机状态下的磁盘文件**，**Container就是虚拟机运行时的磁盘文件，包括内存数据**。

## 1.2 docker 特性
- 文件系统隔离:每个进程容器运行在一个完全独立的根文件系统里
- 资源隔离:系统资源，像 CPU 和内存等可以分配到丌同的容器中，使用 cgroup
- 网络隔离:每个进程容器运行在自己的网络空间，虚拟接口和 IP 地址
- 日志记彔:Docker 将会收集和记彔每个进程容器的标准流(stdout/stderr/stdin)，用于实时检索戒批量检索。
- 变更管理:容器文件系统的变更可以提交到新的镜像中，并可重复使用以创建更多的容器。无需使用模板或手动配置
- 日志记彔:Docker 将会收集和记彔每个进程容器的标准流(stdout/stderr/stdin)，用于实时检索
- 交互式 shell:Docker 可以分配一个虚拟终端并关联到任何容器的标准输入上，例如运行一个一次性交互 shell

缺点局限性:
- Docker 用于应用程序时是最有用的，但并丌包含数据。日志，跟踪和数据库等通常应放在 Docker 容器外。 
- 一个容器的镜像通常都徆小，不适合存大量数据，存储可以通过外部挂载的方式使用。比如使用: NFS，ipsan，MFS 等, -v 映射磁盘分区
- 一句话:docker 叧用于计算，存储交给别人。oracle 不适合使用 docker 来运行，太大了，存储的数据太多。
# 2. 部署docker容器虚拟化平台
## 2.1 安装docker
- ```yum install -y docker```
- ```systemctl start docker && systenctl enable docker && systemctl status docker```
- ```docker version```
```
[root@server15 ~]# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.13.1
... ...
```
## 2.2 下载 docker 镜像
#### 1. 官方下载
- 搜索镜像
```
[root@server15 ~]# docker search centos
INDEX      NAME                               DESCRIPTION                    STARS  OFFICIAL  AUTOMATED
docker.io  docker.io/centos                   The official build of CentOS.  5273   [OK]       
docker.io  docker.io/ansible/centos7-ansible  Ansible on Centos7             121              [OK]
... ...
```
- 下载pull镜像
```
[root@server15 ~]# docker pull docker.io/centos 

Using default tag: latest
Trying to pull repository docker.io/library/centos ... 
latest: Pulling from docker.io/library/centos
8ba884070f61: Pull complete 
Digest: sha256:8d487d68857f5bc9595793279b33d082b03713341ddec91054382641d14db861
Status: Downloaded newer image for docker.io/centos:latest```
```
- 查看本地已有镜像
```
[root@server15 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    latest              9f38484d220f        12 days ago         202 MB
```
#### 2. 把提前下载好的镜像导入docker image
- 把本地 docker.io-centos.tar 镜像上传到 linux 上
- 参数: -i " docker.io-centos.tar " 指定载入的镜像

```[root@server15 ~]# docker load -i /root/docker.io-centos.tar```

#### 3. 下载其他站点的镜像
```[root@server15 ~]# docker pull hub.c.163.com/library/tomcat:latest ```

## 2.3 开启劢网络转发功能
- 开启劢网络转发功能，默认会自劢开启
```
[root@server15 ~]# cat /proc/sys/net/ipv4/ip_forward
1
```
- 手动开启
```
[root@server15 ~]# vim /etc/sysctl.conf 
net.ipv4.ip_forward = 1                 # 输入

[root@server15 ~]# sysctl -p            # 生效 
net.ipv4.ip_forward = 1

[root@server15 ~]# cat /proc/sys/net/ipv4/ip_forward 
1
```

# 3. docker 平台基本使用方法

## 3.1 运行一个docker实例
运行一个 container 并加载镜像 centos，运行起来这个实例后，在实例中执行 /bin/bash

- docker 常用参数:
  - ```run -it```
    - ```-i``` 以交互模式运行容器，通常与 -t 同时使用;
    - ```-t``` 为容器重新分配一个伪输入终端，通常与 -i 同时使用

```
[root@server15 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    latest              9f38484d220f        12 days ago         202 MB

[root@server15 ~]# docker run -it docker.io/centos:latest /bin/bash
[root@c21529149f63 /]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
[root@c21529149f63 /]# exit
exit
```
```
[root@server15 ~]# docker ps -a
CONTAINER ID  IMAGE                    COMMAND     CREATED             STATUS                     PORTS  NAMES
c21529149f63  docker.io/centos:latest  "/bin/bash" About a minute ago  Exited (0) 59 seconds ago         hardcore_jennings
``` 

## 3.2 后台运行一个docker实例
在 container 中启动一个长久运行的进程，不断向 stdin 输出 hello world 。模拟一个后台运行的服务

- docker run 常用参数:
  - ```-d``` 后台运行容器，并返回容器 ID
  - ```-c``` 后面跟待完成的命令

```
[root@server15 ~]# docker run -d docker.io/centos:latest /bin/bash -c "while true;do echo hello world; sleep 1;done"
072cf4be4881083f806f8b448ab1dd302a18b1c402ecf76d84cffe101dd0a521

[root@server15 ~]# docker logs 072cf4be4881
hello world
hello world
hello world
hello world
... ...
```
因为是run -d 后台执行，所以需要手动关闭docker
```
[root@server15 ~]# docker kill 072cf4be4881     # 或者 docker stop 072cf4be4881
```

## 3.3 删除一个docker实例
```
[root@server15 ~]# docker rm 562a6c6eda2a
Error response from daemon: You cannot remove a running container 562a6c6eda2a8d1ab3311d5411190f0e7fb774432edd4d0b06255c20f8ef976c. 
Stop the container before attempting removal or use -f

[root@server15 ~]# docker rm -f 562a6c6eda2a
```
## 3.4 删除全部已有实例
```
[root@server162 ~]# docker ps -a -q               # 过滤出全部容器id
1ea49e301c43
8b2b3a8b89fe
5a8e334f4547
6293ac9d8cbd
abfc748b2421
18ff9dc2f10a
5a209db13389
[root@server162 ~]# docker rm $(docker ps -a -q)
1ea49e301c43
8b2b3a8b89fe
5a8e334f4547
6293ac9d8cbd
abfc748b2421
18ff9dc2f10a
5a209db13389
[root@server162 ~]# docker ps -a
CONTAINER ID      IMAGE      COMMAND      CREATED     STATUS      PORTS      NAMES
```
## 3.5 删除一个docker image镜像
```docker rmi IMAGEID```
```
[root@server162 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    latest              4ef1e5c4b172        17 hours ago        318 MB
docker.io/centos    <none>              9f38484d220f        13 days ago         202 MB

[root@server162 ~]# docker rmi 4ef1e5c4b172
Untagged: docker.io/centos:latest
Deleted: sha256:4ef1e5c4b17240411af513a37bf7eb0aac7a64a1183d5885b46b61e8780dd3b9
Deleted: sha256:bdd846fc53d84188a6af2507b46bbd7e219a04ab963f8571cf089f71351cf25a
Deleted: sha256:98424b457ac5255b8ba10ebdc3fb85080281364031aa4d9085a1c9e802c68895
Deleted: sha256:1dcb115a137dc4a44736046bb61621b4a41824c989bc7de1c85b828ec9b09fbe
Deleted: sha256:7d05b419a39b25e92bacbb253926e08ca2c5a7ce216c84129e7394424a2d10fe
Deleted: sha256:08e6c6eaf3b53eab6e5bc7fcaa37456612c084377de55b947d8dc6fc1143af1a
Deleted: sha256:a43a310f927513c6c18c9e853aa3e3dec24f2de0f3810671fac0d0010402dac3

[root@server162 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    <none>              9f38484d220f        13 days ago         202 MB
```

# 4. docker镜像制作及发布

- 方法 1: docker commit  # 保存 container 的当前状态到 image 后，然后生成对应的 image 
- 方法 2: docker build   # 使用 Dockerfile 文件自劢化制作 image

## 4.1 docker commit
1. 启动一个docker实例，安装apache
```
[root@server15 ~]# docker run -it docker.io/centos:latest /bin/bash
[root@681cded12b1f /]# yum install -y httpd
Loaded plugins: fastestmirror, ovl
Determining fastest mirrors
 * base: ftp.neowiz.com
 * extras: ftp.neowiz.com
 * updates: data.aonenetworks.kr
 ... ...
 Complete!
 
[root@681cded12b1f /]# exit
```
2. 查看刚关闭动docker实例动id
```
[root@server15 ~]# docker ps -a
CONTAINER ID        IMAGE                  
681cded12b1f        docker.io/centos:latest     
```
3. 根据id，commit一个新动镜像
```docker commit 容器id 新生成的镜像名称:tag```
```
[root@server15 ~]# docker commit 681cded12b1f docker.io/centos:apache
sha256:fb5cf00cfd4c5bf4e6c45a480ff3c33876046bcd17572d45dfa89eeb5dd2bf0a
[root@server15 ~]# docker images
REPOSITORY          TAG         IMAGE ID            CREATED             SIZE
docker.io/centos    apache      fb5cf00cfd4c        6 seconds ago       318 MB
docker.io/centos    latest      9f38484d220f        12 days ago         202 MB
```
4. 测试新创建的镜像可用
```
[root@server15 ~]# docker run -it docker.io/centos:apache /bin/bash
[root@826378b093df /]# rpm -qa httpd
httpd-2.4.6-88.el7.centos.x86_64
```
## 4.2 docker build by Dockerfile
Dockerfile 有点像源码编译时./configure 后产生的 Makefile
```
[root@server15 ~]# mkdir /docker-build
[root@server15 ~]# cd /docker-build/
[root@server15 docker-build]# touch Dockerfile
[root@server15 docker-build]# vim Dockerfile 
  FROM docker.io/centos:latest                  # FROM 基于哪个镜像
  MAINTAINER david                              # MAINTAINER 镜像创建者  
  RUN yum install -y httpd                      # RUN 安装软件用
  ADD start.sh /usr/local/bin/start.sh          # ADD 将文件<src>拷贝到新产生的镜像的文件系统对应的路径<dest>
  ADD index.html /var/www/html/index.html       # 所有拷贝到新镜像中的文件和文件夹权限为0755,uid和gid为0

[root@server15 docker-build]# echo "/usr/bin/httpd -DFOREGROUND" > start.sh   # 相当于执行了systemctl start httpd
[root@server15 docker-build]# chmod +x start.sh 
[root@server15 docker-build]# echo "docker build image test" > index.html
```

语法: ```docker build -t 新生成的镜像名:tag Dockerfile文件所在路径```

```
[root@server15 docker-build]# docker build -t docker.io/centos:httpd ./
Sending build context to Docker daemon 4.096 kB
Step 1/5 : FROM docker.io/centos:latest
 ---> 9f38484d220f
Step 2/5 : MAINTAINER david
 ---> Running in 621396f530ff
 ---> 5369cc9c74cd
Removing intermediate container 621396f530ff
Step 3/5 : RUN yum install -y httpd
 ---> Running in 9c2b1baaff9e

Loaded plugins: fastestmirror, ovl
Determining fastest mirrors
 * base: mirror.kakao.com
 * extras: mirror.kakao.com
 * updates: mirror.kakao.com
 ... ...
Complete!
 ---> b5574c71a17a
Removing intermediate container 9c2b1baaff9e
Step 4/5 : ADD start.sh /usr/local/bin/start.sh
 ---> a614a36d4a6b
Removing intermediate container 418be398d33d
Step 5/5 : ADD index.html /var/www/html/index.html
 ---> 675757eae28c
Removing intermediate container 2c39911488dc
Successfully built 675757eae28c
```
查看已创建出新的镜像 docker.io/centos：httpd
```
[root@server15 docker-build]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    httpd               675757eae28c        28 seconds ago      318 MB
docker.io/centos    apache              fb5cf00cfd4c        17 minutes ago      318 MB
docker.io/centos    latest              9f38484d220f        12 days ago 
```
## 4.3 Docker Image 的发布

- 方法 1：Save Image To tar包
- 方法 2：Push Image To Docker Hub

#### 1. 报错docker image为tar
```docker save -o 保存后的名字.tar 需保存的镜像名:tag ```
```
[root@server162 docker-build]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              httpd               fecffc9b8493        36 seconds ago      318 MB
docker.io/centos    latest              9f38484d220f        13 days ago         202 MB
[root@server162 docker-build]# docker save -o docker-centos-httpd.tar centos:httpd 

[root@server162 docker-build]# ll docker-centos-httpd.tar 
-rw------- 1 root root 326070272 3月  28 10:30 docker-centos-httpd.tar
```
导出后的tar，再导入为docker image
```
[root@server162 docker-build]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              httpd               fecffc9b8493        7 minutes ago       318 MB
docker.io/centos    latest              9f38484d220f        13 days ago         202 MB
[root@server162 docker-build]# docker rmi centos:httpd                                # 先删除已存在的http镜像
Untagged: centos:httpd
Deleted: sha256:fecffc9b84934dddb9ac0a10dc0e7deb2907dc046dd04b1d065b83291b6fab5f
Deleted: sha256:ca0307f983100b0efbdedc052e480c95a863e480e57f0f30bf8702f7363456c9
Deleted: sha256:5bbb2864e30b14d1a492fd8595c4fb3f553cacf4a79c8612294c1551b1c2da0f
Deleted: sha256:223625e510bb2a0a45f6ea43e1af5cb79b6e0d4483d0f24621408956d23ae69b
Deleted: sha256:a5de51627549729321de8ac9704b380043c826e6e1f1f9becef7573b2ee5fc50
Deleted: sha256:a35e841920ed4ef53bab979161bcb1aec6ea5fd2b1c8f057e1b3472556cfabdd
Deleted: sha256:daeef8bb05be6fcfc08405da81b0d4eaf81753ca4162f38d4e1a718fdb1f1a82
[root@server162 docker-build]# docker images                                          # 删除成功
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/centos    latest              9f38484d220f        13 days ago         202 MB

[root@server162 docker-build]# docker load -i docker-centos-httpd.tar                 # 导人tar为docker image
5b9e4ba712e0: Loading layer 116.6 MB/116.6 MB
cc20756b914f: Loading layer 3.584 kB/3.584 kB
0254331b8688: Loading layer 3.584 kB/3.584 kB
Loaded image: centos:httpd

[root@server162 docker-build]# docker images                                          # 导入成功
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              httpd               fecffc9b8493        8 minutes ago       318 MB
docker.io/centos    latest              9f38484d220f        13 days ago         202 MB
```
#### 2. push dockerhub
```
[root@server162 ~]# docker login -u ... -p ...
Login Succeeded

[root@server162 ~]# docker push centos:httpd 
Error response from daemon: You cannot push a "root" repository. Please rename your repository to docker.io/<user>/<repo> (ex: docker.io/a406622768/centos)

[root@server162 ~]# docker tag centos:httpd docker.io/a406622768/centos       # tag用于重命名
[root@server162 ~]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
centos                        httpd               fecffc9b8493        17 minutes ago      318 MB
docker.io/a406622768/centos   latest              fecffc9b8493        17 minutes ago      318 MB
docker.io/centos              latest              9f38484d220f        13 days ago         202 MB

[root@server162 ~]# docker push docker.io/a406622768/centos
The push refers to a repository [docker.io/a406622768/centos]
0254331b8688: Pushed 
cc20756b914f: Pushed 
5b9e4ba712e0: Pushed 
d69483a6face: Mounted from library/centos 
latest: digest: sha256:765082fc00ec6d56fbacb498cb3e19498224050b25700bf0f65d96a5953b0d57 size: 1155
```
