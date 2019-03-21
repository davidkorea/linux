# 使用自动化运维工具Ansible集中化管理服务器
1. ansible概述和运行机制
2. 实战-安装并配置Ansible管理两个节点
3. ansible常见模块高级使用方法 
4. 实战-使用Playbook批量部署多台LAMP环境

# 1.Aansible概述和运行机制
Ansible是一款为类Unix系统开发的自由开源的配置和自动化工具。它用Python写成，类似于saltstack和Puppet，但是有一个不同和优点是我们不需要在节点中安装任何客户端。它使用SSH来和节点进行通信。Ansible基于 Python paramiko 开发，分布式，无需客户端，轻量级，配置语法使用 YMAL 及 Jinja2模板语言，更强的远程命令执行操作

Ansible 在管理节点将 Ansible 模块通过 SSH 协议推送到被管理端执行，执行完之后自动删除，可以使用 SVN 等来管理自定义模块及编排。

![](https://i.loli.net/2019/03/20/5c924ad896cc5.png)

上图可以看到 Ansible 的组成由 5 个部分组成：
- Ansible ：     ansible核心
- Modules ：    包括 Ansible 自带的核心模块及自定义模块
- Plugins ：      完成模块功能的补充，包括连接插件、邮件插件等
- Playbooks ：   剧本；定义 Ansible 多任务配置文件，由Ansible 自动执行
- Inventory ：    定义 Ansible 管理主机的清单  [ˈɪnvəntri] 清单

# 2. 实战-安装并配置Ansible管理两个节点

- ansible服务端:  xuegod63 	 192.168.1.63
  - ansible节点1: xuegod63    192.168.1.63
  - ansible节点2: xuegod62  	192.168.1.62

## 2.1 服务端安装Ansible
Ansible默认不在yum仓库中，因此我们需要使用下面的命令启用epel仓库。

```
[root@localhost ~]# yum install epel-release -y

[root@localhost ~]# yum install ansible -y 

[root@localhost ~]# ansible --version
ansible 2.7.8
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Oct 30 2018, 23:45:53) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
```
## 2.2 ansible命令参数
- anisble命令语法： ```ansible [-i 主机文件] [-f 批次] [组名] [-m 模块名称] [-a 模块参数]```
  - ```ansible -i /etc/ansible/hosts 'web-servers'  -m ping```
- ansible详细参数：
  - v,–verbose   #  详细模式，如果命令执行成功，输出详细的结果 (-vv –vvv -vvvv)
  - i PATH, -inventory=PATH      #  指定 host 文件的路径，默认是在 /etc/ansible/hosts. inventory[ˈɪnvəntri]  库存
  - f NUM,-forks=NUM     # NUM 是指定一个整数，默认是 5 ，指定 fork 开启同步进程的个数。
  - m NAME,-module-name=NAME    #   指定使用的 module 名称，默认使用 command模块
  - a,MODULE_ARGS   #指定 module 模块的参数
  - k,-ask-pass           #提示输入 ssh 的密码，而不是使用基于 ssh 的密钥认证
  - sudo          # 指定使用 sudo 获得 root 权限
  - K,-ask-sudo-pass             #提示输入 sudo 密码，与 -sudo 一起使用
  - u USERNAME,-user=USERNAME          # 指定移动端的执行用户
  - C,–check             #测试此命令执行会改变什么内容，不会真正的去执行
- ansible-doc详细参数：
  - ansible-doc -l           #列出所有的模块列表
  - ansible-doc -s 模块名    #查看指定模块的参数 -s, --snippet[ˈsnɪpɪt]片断```ansible-doc -s service```

## 2.3 定义主机清单/etc/ansible/hosts
#### 1. 基于端口，用户，密码定义主机清单
- ansible基于ssh连接-i （inventory）参数后指定的远程主机时，也可以写端口，用户，密码。
- 格式：
  - ansible_ssh_port:指定ssh端口   
  - ansible_ssh_user:指定 ssh 用户 
  - ansible_ssh_pass:指定 ssh 用户登录是认证密码（明文密码不安全）  
  - ansible_sudo_pass:指明 sudo 时候的密码
  - 文件 /etc/ansible/hosts 维护着Ansible中服务器的清单, 在文件最后追加以下内容
    ```
    [root@server15 ~]# vim /etc/ansible/hosts re 
      44 [web-servers]      # 主机组 名
      45 192.168.0.12 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass=11111
    ```
    ```
    [root@server15 ~]# ansible -i /etc/ansible/hosts web-servers -m ping
    192.168.0.12 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
    }
    ```
    - 如果报错
      ```
      "msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled 
      and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to 
      manage this host."
      ```
    - 那么，手动连接一下/etc/ansible/hosts主机清单中的主机，这样就可以在ansible服务器上保存目标主机的fingerprint
       
      ```
      [root@server15 ~]# ssh 192.168.0.12
      The authenticity of host '192.168.0.12 (192.168.0.12)' can't be established.
      ECDSA key fingerprint is SHA256:agsHE/bUbZGaFrQ8tZyxnaiQTg6rYkvGh5+9MjRXLUo.
      ECDSA key fingerprint is MD5:a6:37:b6:d2:00:9e:f1:a0:78:73:8c:48:4e:28:4b:de.
      Are you sure you want to continue connecting (yes/no)? yes
      ```
#### 2. 基于ssh密钥来访问定义主机清单

一般来说，使用明文密码不安全，所以增加主机无密码访问。在Ansible服务端生成密钥，并且复制公钥到所有需要被远程控制到机器。
```
[root@server15 ~]# ssh-keygen                                                 # 一直回车
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:d7FGRNKTCZqCjUfNHgzpONe/SLEsfZ+9BIb6dEiKHUQ root@server15
The key's randomart image is:
+---[RSA 2048]----+
|      o*E o+oo   |
|     =..=o o=    |
|    oo+o+.  o.   |
|    o.oo+  o o   |
|     o oS++ *    |
|      .o=*o= .   |
|      .o+ooo.o.  |
|        .o..o..  |
|          .   .. |
+----[SHA256]-----+

[root@server15 ~]# ssh-copy-id root@192.168.0.15                              # 复制公钥到server15
The authenticity of host '192.168.0.15 (192.168.0.15)' can't be established.
ECDSA key fingerprint is SHA256:7fRdUzAvYC2iJTVAev67cLTPRS0FQ7sy4FiN0g9ToI8.
ECDSA key fingerprint is MD5:e6:f1:11:b9:b5:37:4e:89:df:94:72:63:5d:22:f2:82.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.0.15's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.0.15'"
and check to make sure that only the key(s) you wanted were added.

[root@server15 ~]# ssh-copy-id root@192.168.0.12                              # 复制公钥到client12
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.0.12's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.0.12'"
and check to make sure that only the key(s) you wanted were added.
```
修改host配置文件
```
[root@server15 ~]# vim /etc/ansible/hosts       # 注释掉或者删除掉之前那一行，重新写入需要远程控制到主机ip
 44 [web-servers]
 45 # 192.168.0.12 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass=11111
 46 192.168.0.12
 47 192.168.0.15
```
```
[root@server162 ~]# ansible -m ping web-servers
192.168.0.162 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
192.168.0.163 | UNREACHABLE! => {
    "changed": false, 
    "msg": "Failed to connect to the host via ssh: ssh: connect to host 192.168.0.163 port 22: Connection timed out", 
    "unreachable": true
}
```

## Issue：[两台centos交换了ip，ssh-copy-id 失败](https://github.com/davidkorea/linux_study/issues/1)



## 2.4 在Ansible服务端运行命令
command模块执行shell命令，command作为ansible -m的默认模块，可以运行远程权限范围内的所有shell命令

#### 1. ping模块检查网络连通性
此处 -m 的参数为 ping 模块，不是默认的command模块
- ```[root@server15 ~]# ansible -i /etc/ansible/hosts 'web-servers' -m  ping```
```
[root@server15 ~]# ansible web-servers -m ping      # 不指定，默认使用/etc/ansible/hosts文件
192.168.0.15 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
192.168.0.12 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

#### 2. 检查Ansible节点的运行时间（uptime）

- ```[root@server15 ~]# ansible web-servers -m command -a 'uptime'```
```
[root@server15 ~]# ansible web-servers -a "uptime"      # -m command是默认参数，可以不写
192.168.0.12 | CHANGED | rc=0 >>
 23:05:32 up 35 min,  4 users,  load average: 0.00, 0.01, 0.05

192.168.0.15 | CHANGED | rc=0 >>
 23:05:32 up 28 min,  4 users,  load average: 0.16, 0.07, 0.06
```

#### 3. 检查节点的内核版本
```
[root@server15 ~]# ansible web-servers -a 'uname -r'
192.168.0.15 | CHANGED | rc=0 >>
3.10.0-957.el7.x86_64

192.168.0.12 | CHANGED | rc=0 >>
3.10.0-957.el7.x86_64
```
#### 4. 给节点增加用户
```
[root@server15 ~]# ansible web-servers -a "useradd test_user"
192.168.0.15 | CHANGED | rc=0 >>


192.168.0.12 | CHANGED | rc=0 >>

```
查看添加成功
```
[root@server15 ~]# ansible web-servers -a "tail -1 /etc/passwd"
192.168.0.15 | CHANGED | rc=0 >>
test_user:x:1001:1001::/home/test_user:/bin/bash

192.168.0.12 | CHANGED | rc=0 >>
test_user:x:1001:1001::/home/test_user:/bin/bash
```
#### 5. 将df命令在所有节点执行后，重定向输出到本机的/tmp/command-output.txt文件中

```
[root@server15 ~]# ansible web-servers -a "df -h" > /tmp/dfh.txt
[root@server15 ~]# cat /tmp/dfh.txt 
192.168.0.12 | CHANGED | rc=0 >>
文件系统        容量  已用  可用 已用% 挂载点
/dev/sda2        10G  4.8G  5.3G   48% /
devtmpfs        2.0G     0  2.0G    0% /dev
tmpfs           2.0G     0  2.0G    0% /dev/shm
tmpfs           2.0G   13M  2.0G    1% /run
tmpfs           2.0G     0  2.0G    0% /sys/fs/cgroup
/dev/sda1       197M  141M   56M   72% /boot
tmpfs           394M  4.0K  394M    1% /run/user/42
tmpfs           394M   36K  394M    1% /run/user/0
192.168.0.15 | CHANGED | rc=0 >>
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   78G  4.4G   74G    6% /
devtmpfs                 2.0G     0  2.0G    0% /dev
tmpfs                    2.0G  124K  2.0G    1% /dev/shm
tmpfs                    2.0G   13M  2.0G    1% /run
tmpfs                    2.0G     0  2.0G    0% /sys/fs/cgroup
/dev/sda1                497M  159M  339M   32% /boot
tmpfs                    394M  4.0K  394M    1% /run/user/42
tmpfs                    394M   28K  394M    1% /run/user/0
/dev/sr0                 4.3G  4.3G     0  100% /run/media/root/CentOS 7 x86_64
```

# 3. Ansible常见模块高级使用方法

## 3.1 三个远程命令模块的区别
#### 1. command模块
为ansible默认模块，不指定-m参数时，使用的就是command模块； comand模块比较简单，常见的命令都可以使用，但其命令的执行不是通过shell执行的，所以，像这些 "<", ">", "|", and "&"操作都不可以，当然，也就不支持管道； 缺点：不支持管道，没法批量执行命令。

话虽如此，但这样也成功了
```
[root@server162 ~]# ansible web-servers -a "netstat -anutp | grep httpd"
```
#### 2. shell模块
使用shell模块，在远程命令通过/bin/sh来执行；所以，我们在终端输入的各种命令方式，都可以使用
```
[root@server162 ~]# ansible -i /etc/ansible/hosts  web-servers -m shell -a "free -m"
```
注：但是我们自己定义(alias 自定义变量)在~/.bashrc或~/.bash_profile中的环境变量shell模块由于没有加载，所以无法识别；如果需要使用自定义的环境变量，就需要在最开始，执行加载自定义脚本的语句
- 对shell模块的使用可以分成两块： 
  - 如果待执行的语句少，可以直接写在一句话中
    ```
    [root@server162 ~]# ansible  web-servers -a "source  ~/.bash_profile && df -h | grep sda2"
    192.168.0.162 | FAILED | rc=2 >>
    [Errno 2] No such file or directory
                                                    # 果然，不用shell模块，执行会报错
    192.168.0.163 | FAILED | rc=2 >>
    [Errno 2] No such file or directory

    [root@server162 ~]# ansible  web-servers -m shell -a "source  ~/.bash_profile && df -h | grep sda2"
    192.168.0.162 | FAILED | rc=1 >>
    non-zero return code                  # 162只用来sda1

    192.168.0.163 | CHANGED | rc=0 >>
    /dev/sda2        10G  4.8G  5.3G  48% /
    ```
  - 如果在远程待执行的语句比较多，可写成一个脚本，通过copy模块传到远端，然后再执行；但这样就又涉及到两次ansible调用；对于这种需求，ansible已经为我们考虑到了，script模块就是干这事的
#### 3. scripts模块
使用scripts模块可以在本地写一个脚本，在远程服务器上执行
```
[root@server162 ~]# vim  /etc/ansible/net.sh
  #!/bin/bash
  date
  hostname

[root@server162 ~]# ansible web-servers -m script -a "/etc/ansible/net.sh"
192.168.0.162 | CHANGED => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 192.168.0.162 closed.\r\n", 
    "stderr_lines": [
        "Shared connection to 192.168.0.162 closed."
    ], 
    "stdout": "Thu Mar 21 11:29:53 CST 2019\r\nserver162\r\n", 
    "stdout_lines": [
        "Thu Mar 21 11:29:53 CST 2019", 
        "server162"
    ]
}
192.168.0.163 | CHANGED => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 192.168.0.163 closed.\r\n", 
    "stderr_lines": [
        "Shared connection to 192.168.0.163 closed."
    ], 
    "stdout": "Thu Mar 21 11:29:53 CST 2019\r\nclient163\r\n", 
    "stdout_lines": [
        "Thu Mar 21 11:29:53 CST 2019", 
        "client163"
    ]
}
```
## 3.2 copy模块:实现主控端向目标主机拷贝文件，类似scp功能

```
[root@server162 ~]# ansible web-servers -m copy -a "src=/etc/hosts dest=/tmp owner=root group=root mode=0755"
```
```
[root@client163 tmp]# ll hosts 
-rwxr-xr-x 1 root root 234 3月  21 11:35 hosts
```
## 3.3 file模块设置文件属性
```
[root@server162 ~]# ansible web-servers -m file -a "path=/tmp/hosts mode=0777"
```
```
[root@client163 tmp]# ll hosts 
-rwxrwxrwx 1 root root 234 3月  21 11:35 hosts
```
## 3.4 stat模块获取远程文件信息
```
[root@server162 ~]# ansible web-servers -m stat -a "path=/tmp/hosts"
```
## 3.5 get_url模块实现远程主机下载指定url到本地，支持sha256sum文件校验
```
[root@server162 ~]# ansible web-servers -m get_url -a "url=https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm dest=/tmp/ mode=0440 force=yes"
```
- 如果force=yes，当下载文件时，如果所下的内容和原目录下的文件内容不一样，则替换原文件，如果一样，就不下载了。
## 3.6 yum模块linux平台软件包管理
- yum模块可以提供的status状态： 
  - latest，present，installed  # 这3个代表安装；
  - removed, absent # 这2个是卸载

```
[root@server162 ~]# ansible web-servers -m yum -a "name=php state=latest"
```
## 3.7 cron模块远程主机crontab配置
```
[root@server162 ~]# ansible web-servers -m cron -a "name='list dir' minute='*/30' job='ls /tmp'"
```
```
[root@server162 ~]# crontab -l
#Ansible: list dir
*/30 * * * * ls /tmp
```

## 3.8 service模块远程主机系统服务管理
- service模块常用参数：
  - name参数：此参数用于指定需要操作的服务名称，比如 nginx，httpd。 
  - state参数：此参数用于指定服务的状态，比如，我们想要启动远程主机中的httpd，则可以将 state 的值设置为 started；如果想要停止远程主机中的服务，则可以将 state 的值设置为 stopped。此参数的可用值有 started、stopped、restarted（重启）、reloaded。 
  - enabled参数：此参数用于指定是否将服务设置为开机 启动项，设置为 yes 表示将对应服务设置为开机启动，设置为 no 表示不会开机启动。
  
注：想使用service模块启动服务，被启动的服务，必须可以使用service 命令启动或关闭，只能systemctl的服务不可以

```
[root@server162 ~]# ansible web-servers -m service -a "name=httpd state=restarted"
```

## 3.9 sysctl模块远程主机sysctl配置
?????干什么用
```
[root@server162 ~]# ansible web-servers -m sysctl -a "name=net.ipv4.ip_forward value=1 reload=yes"
192.168.0.162 | CHANGED => {
    "changed": true
}
192.168.0.163 | CHANGED => {
    "changed": true
}
```
## 3.10 user模块远程主机用户管理
```
[root@server162 ~]# ansible web-servers -m user -a "name=xerox state=present"
192.168.0.162 | SUCCESS => {
    "append": false, 
    "changed": false, 
    "comment": "xerox", 
    "group": 1000, 
    "home": "/home/xerox", 
    "move_home": false, 
    "name": "xerox", 
    "shell": "/bin/bash", 
    "state": "present", 
    "uid": 1000
}
192.168.0.163 | SUCCESS => {
    "append": false, 
    "changed": false, 
    "comment": "xerox", 
    "group": 1000, 
    "home": "/home/xerox", 
    "move_home": false, 
    "name": "xerox", 
    "shell": "/bin/bash", 
    "state": "present", 
    "uid": 1000
}
```
# 4. 实战-使用Playbook批量部署多台LAMP环境
## 4.1 playbooks使用步骤
#### 1. 在playbooks 中定义任务
- name： task description     #任务描述信息
- module_name: module_args    #需要使用的模块名字：  模块参数
#### 2. ansible-playbook 执行命令
 ```[root@server162 ~]# ansible-playbook site.yml```
- playbook是由一个或多个"play"组成的列表。play的主要功能在于将事先归为一组的主机装扮成事先通过ansible中的task定义好的角色。
- github上提供了大量的实例供大家参考  https://github.com/ansible/ansible-examples
#### 3. Playbook常用文件夹作用： 
- files：存放需要同步到异地服务器的源码文件及配置文件
- tasks：需要进行的执行的任务
- handlers：当服务的配置文件发生变化时需要进行的操作，比如：重启服务，重新加载配置文件 ['hændləz] 处理程序
- meta：角色定义，可留空
- templates：用于执行lamp安装的模板文件，一般为脚本
- vars：本次安装定义的变量

## 4.2 服务器上测试安装LAMP环境
我们可以在ansible服务器上安装LAMP环境，然后，再将配置文件通过ansible拷贝到远程主机上
#### 1. 安装httpd软件
```
yum install httpd -y
```
#### 2. 安装MySQL服务端和客户端
```
yum install mariadb-server  mariadb  -y
```
```
[root@server162 ~]# mkdir -p /mydata/data

[root@server162 ~]# chown -R mysql:mysql /mydata/
[root@server162 ~]# ll /mydata/
total 0
drwxr-xr-x 2 mysql mysql 6 Mar 21 13:27 data

[root@server162 ~]# vim /etc/my.cnf
  1 [mysqld]
  2 datadir=/mydata/data                   # 更改存放数据的路径

[root@server162 ~]# systemctl start mariadb
您在 /var/spool/mail/root 中有新邮件
[root@server162 ~]# ll /mydata/data/       # 必须先更改存放路径，否则启动服务后自动会生成数据不会到新的目录
total 37852
-rw-rw---- 1 mysql mysql    16384 Mar 21 13:31 aria_log.00000001
-rw-rw---- 1 mysql mysql       52 Mar 21 13:31 aria_log_control
-rw-rw---- 1 mysql mysql  5242880 Mar 21 13:31 ib_logfile0
-rw-rw---- 1 mysql mysql  5242880 Mar 21 13:31 ib_logfile1
-rw-rw---- 1 mysql mysql 18874368 Mar 21 13:31 ibdata1
drwx------ 2 mysql mysql     4096 Mar 21 13:31 mysql
drwx------ 2 mysql mysql     4096 Mar 21 13:31 performance_schema
drwx------ 2 mysql mysql        6 Mar 21 13:31 test
```
#### 3. 安装PHP和php-mysql模块
```
yum install php php-mysql -y
```
#### 4. 提供php的测试页
```
[root@server162 ~]# vim /var/www/html/index.php
<?php
        phpinfo();
?>
```
#### 5. 启动httpd服务，在浏览器中访问http://192.168.0.162/
```
[root@server162 ~]# systemctl restart httpd 
[root@server162 ~]# iptables -F
```
  - http://192.168.0.162/index.php 访问成功

## 4.3 实战playbook
### 4.3.1 Ansible基本设定
#### 1. /etc/ansible/hosts
```
[root@server162 ~]# vim /etc/ansible/hosts 
  [web-servers]
  192.168.0.162
  192.168.0.163
```
#### 2. ssh-keygen
```
[root@server162 ~]# ssh-keygen 
[root@server162 ~]# ssh-copy-id root@192.168.0.162
[root@server162 ~]# ssh-copy-id root@192.168.0.163
```
### 4.3.2 使用playbook构建LAMP任务
#### 1. 创建相关目录文件
```
mkdir -pv /etc/ansible/lamp/roles/{prepare,httpd,mysql,php}/{tasks,files,templates,vars,meta,default,handlers}
```
```
[root@server162 ~]# tree /etc/ansible/lamp/
/etc/ansible/lamp/
`-- roles
    |-- httpd
    |   |-- default
    |   |-- files
    |   |-- handlers
    |   |-- meta
    |   |-- tasks
    |   |-- templates
    |   `-- vars
    |-- mysql
    |   |-- default
    |   |-- files
    |   |-- handlers
    |   |-- meta
    |   |-- tasks
    |   |-- templates
    |   `-- vars
    |-- php
    |   |-- default
    |   |-- files
    |   |-- handlers
    |   |-- meta
    |   |-- tasks
    |   |-- templates
    |   `-- vars
    `-- prepare
        |-- default
        |-- files
        |-- handlers
        |-- meta
        |-- tasks
        |-- templates
        `-- vars

33 directories, 0 files
```
将上一步测试搭建成功的httpd和MySQL的配置文件拷贝到LAMP对应目录下
```
[root@server162 ~]# cp /etc/my.cnf /etc/ansible/lamp/roles/mysql/files/
```
```
/etc/ansible/lamp/
`-- roles
    |-- httpd
    |   |-- default
    |   |-- files
    |   |   `-- httpd.conf
    |   |-- handlers
    |   |-- meta
    |   |-- tasks
    |   |-- templates
    |   `-- vars
    |-- mysql
    |   |-- default
    |   |-- files
    |   |   `-- my.cnf
    |   |-- handlers
    |   |-- meta
    |   |-- tasks
    |   |-- templates
    |   `-- vars
```
#### 2. 写prepare角色的playbooks
```
[root@server162 ~]# vim /etc/ansible/lamp/roles/prepare/tasks/main.yml

- name: delete yum config
  shell: rm -rf /etc/yum.repos.d/*
- name: provide yum repo file
  shell: wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
- name: clean the yum repo
  shell: yum clean all
- name: clean iptables
  shell: iptables -F
```

#### 3. 构建httpd的任务
```
[root@server162 ~]# cp /var/www/html/index.php /etc/ansible/lamp/roles/httpd/files/
[root@server162 ~]# cp /etc/httpd/conf/httpd.conf /etc/ansible/lamp/roles/httpd/files/

[root@server162 ~]# tree /etc/ansible/lamp/roles/
/etc/ansible/lamp/roles/
|-- httpd
|   |-- default
|   |-- files
|   |   |-- httpd.conf
|   |   `-- index.php
|   |-- handlers
|   |-- meta
|   |-- tasks
|   |   `-- main.yml
|   |-- templates
|   `-- var
```
```
[root@server162 ~]# vim /etc/ansible/lamp/roles/httpd/tasks/main.yml
- name: web server install
  yum: name=httpd state=present                  # 安装httpd服务
- name: provide test page
  copy: src=index.php dest=/var/www/html         # 提供测试页，将上面拷贝到files的文件复制到客户端
- name: delete apache config
  shell: rm -rf  /etc/httpd/conf/httpd.conf      # 删除原有的apache配置文件，如果不删除，下面的copy任务是不会执行的
                                                 # 因为当源文件httpd.conf和目标文件一样时，copy命令是不执行的
                                                 # 如果copy命令不执行，那么notify将不调用handler
- name: provide configuration file
  copy: src=httpd.conf dest=/etc/httpd/conf/httpd.conf    # 提供httpd的配置文件
  notify: restart httpd                          # 当前面的copy复制成功后，通过notify通知名字为restart httpd的handlers运行
```

- 扩展：notify和handlers
  - notify： 这个action可用于在每个play的最后被触发，这样可以避免多次有改变发生时，每次都执行指定的操作，取而代之，仅在所有的变化发生完成后一次性地执行指定操作。在notify中列出的操作称为handler，也即notify中调用handler中定义的操作。
  - handlers概述：
    - Handlers 也是一些 task 的列表,通过名字来引用,它们和一般的 task 并没有什么区别。Handlers 是由通知者进行notify, 如果没有被 notify，handlers 不会执行。
    - 不管有多少个通知者进行了notify，等到 play 中的所有 task 执行完成之后,handlers 也只会被执行一次。
    - Handlers 最佳的应用场景是用来重启服务,或者触发系统重启操作.除此以外很少用到了。


