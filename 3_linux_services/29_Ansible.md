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









