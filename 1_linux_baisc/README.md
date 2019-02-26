# linux - centos7 baiscs

21. 系统进程管理ps, top, uptime, df,
- ```ps -aux```, 用BSD的格式来显示进程
	- USER: 启动这些进程的用户
	- PID: 进程的ID
	- %CPU 进程占用的CPU百分比； 
	- %MEM 占用内存的百分比； 
	- VSZ：进程占用的虚拟内存大小（单位：KB） 
	- RSS：进程占用的物理内存大小（单位：KB） 
	- STAT：该程序目前的状态，Linux进程有5种基本状态：
	- R ：该程序目前正在运作，或者是可被运作；
		- S ：该程序目前正在睡眠当中 (可说是 idle(空闲，空转) 状态啦！)，但可被某些讯号(signal) 唤醒。
		- T ：该程序目前正在侦测或者是停止了；
		- Z ：该程序应该已经终止，但是其父程序却无法正常的终止他，造成 zombie (疆尸) 程序的状态
		- D  不可中断状态
		- <: 表示进程运行在高优先级上
		- N: 表示进程运行在低优先级上
		- L: 表示进程有页面锁定在内存中
		- s: 表示进程是控制进程
		- l: 表示进程是多线程的
		- +: 表示当前进程运行在前台
	- START：该 process 被触发启动的时间；
	- TIME ：该 process 实际使用 CPU 运作的时间。
	- COMMAND：该程序的实际指令。[xxxx] 使用方括号括起来的进程是内核态的进程，没有括起来的是用户态进程。
- ```ps -ef```, 用标准的格式显示进程
	- UID: 启动这些进程的用户
	- PID: 进程的ID
	- PPID: 父进程的进程号
	- C: 进程生命周期中的CPU利用率
	- STIME: 进程启动时的系统时间
	- TTY: 表明进程在哪个终端设备上运行。如果显示?表示与终端无关，这种进程一般是内核态进程。tty1-tty6 是本机上面的登入者程序，若为 pts/0 则表示运行在虚拟终端上的进程。
	- TIME: 运行进程一共累计占用的CPU时间
	- CMD: 启动的程序名称
- uptime查看系统负载
	```
	15:10:19 up  1:28,  1 user,  load average: 0.00, 0.01, 0.05
	```
	- load average系统负载，即任务队列的平均长度。 三个数值分别为  1分钟、5分钟、15分钟前到现在的平均值。经验：单核心，1分钟的系统平均负载不要超过3，就可以，这是个经验值


-----

20. 文件的归档和压缩
- tar
	- c	create创建文件
	- x	-extract [ˈekstrækt]  提取 解压还原文件
	- v	--verbose显示执行详细过程
	- f	--file指定备份文件
	- t	--list 列出压缩包中包括哪些文件，不解包，查看包中的内容
	- C （大写）--directory   指定解压位置
- tar归档
	- ```tar cvf share.tar ./windowsshare219```压缩
	- ```tar xvf share.tar -C ./windowsshare219/```解压，-C制定解压位置
	- ```tar tvf share.tar```不解压查看内容
- tar压缩
	- z, --gzip   以gzip方式压缩  扩展名： tar.gz
	- j ：        以bz2方式压缩的  扩展名：tar.bz2
	- J ：        以xz 方式压缩   扩展名：tar.xz
- ```tar zcvf gzshare.tar.gz ./windowsshare219/```
- ```tar zxvf gzshare.tar.gz -C ./b```

- zip, unzip
	- ```zip pswd.zip /etc/passwd```
	- ```unzip pswd.zip -d ./share```, -d制定解压路径

- 查看文件类型
	- ```file /etc/passwd```, /etc/passwd: ASCII text

- 按一定规则排序查看文件
	- ```ll -tr```, -t按照时间，-r逆序查看文件。用于找到新创建的文件，显示于最后一行，方便查看。
	- ```ll -Srh```, -S文件大小，-r逆序，-h文件大小按照kb，mb显示


























-----

20. Centos7 挂载 windows 共享文件夹
```
[root@localhost ~]# mkdir windowsshare219
[root@localhost ~]# mount -t cifs -o username=xerox,password=PASSWORD1! //192.168.0.219/file ./windowsshare219
[root@localhost ~]# cd windowsshare219/
[root@localhost windowsshare219]# ls
extundelete-0.2.4.tar.bz2       nginx-1.12.2.tar.gz
lrzsz-0.12.20-27.1.el6.src.rpm  说明.txt
```

-----

19. 软件包的的安装与管理 rpm, yum
- 软件包的类型
	- rpm二进制包，已经使用GCC编译后的
	- tar源码包，需要编译
	- RPM概述：RPM是RPM Package Manager（RPM软件包管理器）的缩写，这一文件格式名称虽然打上了RedHat的标志，但是其原始设计理念是开放式的，现在包括OpenLinux、SUSE以及Turbo Linux等Linux的分发版本都有采用，可以算是公认的行业标准了

- rpm -ivh, ``` rpm -ivh /mnt/Packages/zsh-5.0.2-28.el7.x86_64.rpm```
	-i  是install的意思， 安装软件包
	-v  显示附加信息，提供更多详细信息
	-V  校验，对已经安装的软件进行校验
	-h  --hash  安装时输出####标记
- 从网上下载直接安装centos epel扩展源
	- ```rpm -ivh http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm ```，安装centos epel扩展yum源。 注：epel源是对centos7系统中自带的 base源的扩展。
- rpm查询功能
	- 用法：rpm -q（query） 常与下面参数组合使用
		-a（all）  查询所有已安装的软件包
		-f（file）系统文件名（查询系统文件所属哪个软件包），反向查询
		-i  显示已经安装的rpm软件包信息，后面直接跟包名
		-l（list）  查询软件包中文件安装的位置
		-p  查询未安装软件包的相关信息，后面要跟软件的命名
		-R 查询软件包的依赖性

	- ```rpm -q zsh```，查询指定的包是否安装
	- ```rpm -qa```，查询所有已安装包
	- ```rpm -qa |  grep vim```，查询所有已安装包中带vim关键字的包
	- ```which find```，查看find命令的路径/usr/bin/find
	- ```rpm  -qf /usr/bin/find```，查询文件或命令属于哪个安装包
	- ```rpm -qi lrzsz```，查询已经安装的rpm包的详细信息或作用  rpm -qi  rpm包名
	- ```rpm -qpi /mnt/Packages/php-mysql-5.4.16-42.el7.x86_64.rpm ```，针对没有安装的RPM包，要加参数：-p

- 查看软件包内容是否被修改```rpm -V包名```，```rpm -Vf 文件路径```
	```
	[root@localhost ~]# rpm -Vf /usr/bin/find   #检查具体文件
	[root@localhost ~]# echo aaa >> /usr/bin/find
	[root@localhost ~]# rpm -Vf /usr/bin/find
	S.5....T.    /usr/bin/find
	```
	如果出现的全是点，表示测试通过,出现下面的字符代表某测试的失败:
	- 5 — MD5 校验和是否改变，你也看成文件内容是否改变
	- S — 文件长度，大小是否改变
	- L — 符号链接，文件路径是否改变
	- T — 文件修改日期是否改变
	- D — 设备
	- U — 用户，文件的属主
	- G — 用户组
	- M — 模式 (包含许可和文件类型)
	- ? — 不可读文件

- rpm包卸载和升级
	- 卸载rpm  -e（erase） 包名，``` rpm -e zsh```
	- 升级```rpm -Uvh /mnt/Packages/lrzsz-0.12.20-36.el7.x86_64.rpm```，因为升级时会有一些依赖包要解决。 所以一般我们使用yum update 包  来升级。

- yum
	-  配置本地yum源，光盘镜像
		- ```mount /dev/cdrom /mnt/ ```
		- ```vim /etc/yum.repos.d/centos7.repo```   #必须以.repo结尾，插入以下内容
			```
			[centos7]	#yum源名称，在本服务器上唯一的，用来区分不同的yum源			
			name= CentOS7	#对yum源描述信息
			baseurl=file:///mnt	#yum源的路径,提供方式包括FTP(ftp://...)、HTTP(http://...),
						# 本地(file:///... 光盘挂载目录所在的位置）
			enabled=1	#为1，表示启用yum源；0为禁用
			gpgcheck=0	#为1，使用公钥检验rpm包的正确性；0为不校验
			gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7  #指定进行rpm校验的公钥文件地址
			```
	- 配置网络yum源
		- ```wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo```, wget下载文件 ，-O 将wget下载的文件，保存到指定的位置，保存时可以重新起一个名字，或者直接写一个要保存的路径，这样还用原来的文件名。

			```
			[root@localhost ~]# wget -O /etc/yum.repos.d/CentOS-7-aliyun.repo http://mirrors.aliyun.com/repo/Centos-7.repo
			--2019-02-26 10:50:31--  http://mirrors.aliyun.com/repo/Centos-7.repo
			Resolving mirrors.aliyun.com (mirrors.aliyun.com)... vim ^H^H^H47.246.59.226, 47.246.59.233, 47.246.29.16, ...
			Connecting to mirrors.aliyun.com (mirrors.aliyun.com)|47.246.59.226|:80... connected.
			HTTP request sent, awaiting response... ^H200 OK
			Length: 2523 (2.5K) [application/octet-stream]
			Saving to: '/etc/yum.repos.d/CentOS-7-aliyun.repo'

			100%[==============================================>] 2,523       --.-K/s   in 0s      

			2019-02-26 10:50:41 (70.9 MB/s) - '/etc/yum.repos.d/CentOS-7-aliyun.repo' saved [2523/2523]

			[root@localhost ~]# vim /etc/yum.repos.d/CentOS-7-aliyun.repo 
			
			"""
			vim界面中执行:!cat /etc/centos-release，找到CentOS Linux release 7.6.1810 (Core) 
			$releasever = 7.6.1810
			$basearch = x86_64
			
			"""
			```
		- ```yun clean all```
		- ```yum list```
			```
			[root@localhost ~]# yum clean all
			Failed to set locale, defaulting to C
			Loaded plugins: fastestmirror, langpacks
			Repository base is listed more than once in the configuration
			Repository updates is listed more than once in the configuration
			Repository extras is listed more than once in the configuration
			Repository centosplus is listed more than once in the configuration
			Cleaning repos: base extras updates
			Cleaning up list of fastest mirrors
			[root@localhost ~]# yum list
			Failed to set locale, defaulting to C
			Loaded plugins: fastestmirror, langpacks
			Repository base is listed more than once in the configuration
			Repository updates is listed more than once in the configuration
			Repository extras is listed more than once in the configuration
			Repository centosplus is listed more than once in the configuration
			Determining fastest mirrors
			 * base: mirrors.aliyun.com
			 * extras: mirrors.aliyun.com
			 * updates: mirrors.aliyun.com
			 
			...
			...
			```
- yum常用操作：
	- yum install -y httpd   #安装软件包， -y 直接安装，不在询问安装过程中的yes/no，全部默认同意安装 
	- yum -y update    #升级软件包，改变软件设置和系统设置,系统版本内核都升级
	- yum -y upgrade   #升级软件包，不改变软件设置和系统设置，系统版本升级，内核不改变
	- yum -y update  # 不加任何包，表示整个系统进行升级
	- yum info  httpd    #查询rpm包作用  
	- yum provides /usr/bin/find  #查看命令是哪个软件包安装的  
	- yum -y remove  包名    #卸载包 
	- yum search keyword   #按关键字搜索软件包

- tar源码包安装, 源码安装nginx
	- 准备环境
		- ```yum -y install gcc gcc-c++ make zlib-devel pcre pcre-devel openssl-devel```
	- 源码编译3把斧：./configure, make, make install
		- ```./configure --prefix=/usr/bin/local/nginx```, prefix指定安装路径
		- ```make  -j 4```, 按Makefile文件编译，可以使用-j 4指定4核心CPU编译，提升速度
		- ```make install```, 按Makefile定义的文件路径安装
		- ```make clean```, 清除上次的make命令所产生的object和Makefile文件。使用场景：当需要重新执行configure时，需要执行make clean

		```
		[root@localhost nginx-1.12.2]# ./configure --prefix=/usr/local/nginx
		checking for OS
		 + Linux 3.10.0-957.el7.x86_64 x86_64
		checking for C compiler ... found
		 + using GNU C compiler
		 + gcc version: 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC) 
		checking for gcc -pipe switch ... found
		checking for -Wl,-E switch ... found
		...
		Configuration summary
		  + using system PCRE library
		  + OpenSSL library is not used
		  + using system zlib library

		  nginx path prefix: "/usr/local/nginx"
		  nginx binary file: "/usr/local/nginx/sbin/nginx"
		  nginx modules path: "/usr/local/nginx/modules"
		  nginx configuration prefix: "/usr/local/nginx/conf"
		  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
		  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
		  nginx error log file: "/usr/local/nginx/logs/error.log"
		  nginx http access log file: "/usr/local/nginx/logs/access.log"
		  nginx http client request body temporary files: "client_body_temp"
		  nginx http proxy temporary files: "proxy_temp"
		  nginx http fastcgi temporary files: "fastcgi_temp"
		  nginx http uwsgi temporary files: "uwsgi_temp"
		  nginx http scgi temporary files: "scgi_temp"

		[root@localhost nginx-1.12.2]# make -j 4
		make -f objs/Makefile
		make[1]: Entering directory `/root/nginx-1.12.2'
		cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event 
		-I src/event/modules -I src/os/unix -I objs \		
		make[1]: Leaving directory `/root/nginx-1.12.2'
		
		[root@localhost nginx-1.12.2]# make install
		make -f objs/Makefile install
		make[1]: Entering directory `/root/nginx-1.12.2'
		test -d '/usr/local/nginx' || mkdir -p '/usr/local/nginx'		
		make[1]: Leaving directory `/root/nginx-1.12.2'

		```
		- ```make uninstall``` ,有时删除不干净，所以建议大家安装时，在configure步骤添加一个： --prefix  参数。这样删除或备份时，直接对删除--prefix指定的安装目录操作就可以了。





















-----

18. 文件权限管理 rwx（UGO）

- 文件类型
	- p表示命名管道文件
	- d表示目录文件
	- l表示符号连接文件
	- -表示普通文件
	- s表示socket套接口文件，比如我们启用mysql时，会产生一个mysql.sock文件
	- c表示字符设备文件，例： 虚拟控制台 或tty0
	- b表示块设备文件   例： sda， cdrom
	
- 更改文件的属主和属组 ```chown user:group filename```，可以单独使用```chown user filename```，或```chown :group filename```

- 修改权限```chmod g+x  1.txt```, ```chmod a=rwx  1.txt```
	- 符号, +  添加权限，-  减少权限，=  直接给定一个权限
	
	|命令|说明|
	|-|-|
	|u-w|user拥有者，减去写入权限|
	|g+x|group组，赋予执行权限|
	|o=r|other其他人，赋予读取权限|
	|a+x|all所有人，赋予执行权限|

	

	- 八进制 ```chmod 755 dd.txt```	
	
	|权限|二进制值|八进制值|描述|
	|-|-|-|-|
	|---|000|0|没有任何权限|
	|--x|001|1|只有执行权限|
	|-w-|010|2|只有写入权限|
	|-wx|011|3|有写入和执行权限|
	|r--|100|4|只有读取权限|
	|r-x|101|5|有读取和执行权限|
	|rw-|110|6|有读取和写入权限|
	|rwx|111|7|有全部权限|
	
	- 补码
	umask命令允许你设定文件创建时的缺省模式，对应每一类用户(文件属主、同组用户、其他用户)存在一个相应的umask值中的数字。文件默认权限＝666 ，目录默认权限＝777。我们一般在```/etc/profile、$ [HOME]/.bash_profile```或$```[HOME]/.profile```中设置umask值。永久生效，编辑用户的配置文件```vim .bash_profile ```

- 文件的特殊权限：suid sgid sticky和文件扩展权限ACL
	- SUID（set uid设置用户ID）：限定：只能设置在二进制可执行程序上面，对目录设置无效。功能：程序运行时的权限从执行者变更成程序所有者的权限
		```
		[root@localhost ~]# ll /usr/bin/passwd 
		-rwsr-xr-x. 1 root root 27832 6月  10 2014 /usr/bin/passwd # s: SUID 用户修改密码是会临时获得所有权
		```
	- SGID：限定：既可以给二进制可执行程序设置，也可以对目录设置。功能：在设置了SGID权限的目录下建立文件时，新创建的文件的所属组会继承上级目录的所属组
	- Stickybit：粘滞位权限是针对目录的，对文件无效，也叫防删除位。目录下创建的文件只有root、文件创建者、目录所有者才能删除。
		```
		[root@localhost ~]# ll -d /tmp/ # -d 仅查看目录情况，不查看目录下的内容
		drwxrwxrwt. 18 root root 4096 2月  26 09:42 /tmp/ # t 粘滞位
		```
	- 操作命令
	
	|SUID|SGID|Stickybit|
	|-|-|-|
	|u+s或u=4|g+s或g=2|o+t或o=1|
	
	- 文件扩展权限ACL(access control list)
		- 设置用户xerox对文件1.txt拥有的rwx权限 ，xerox不属于a.txt的所属主和组，xerox是other。怎么做？
		- ```getfacl 2.txt```
		- ```setfacl -m u:xerox:rwx 2.txt```, ```setfacl -m d:u:xerox:rwx /tmp/test```对目录有效，此目录下新建的目录或文件都继承此acl权限

			```
			[root@localhost ~]# getfacl 2.txt
			# file: 2.txt
			# owner: root
			# group: root
			user::rw-
			group::r--
			other::r--
			
			[root@localhost ~]# setfacl -m u:xerox:rwx 2.txt # 只对other中的xerox用户赋予rwx权限
			
			[root@localhost ~]# getfacl 2.txt 
			# file: 2.txt
			# owner: root
			# group: root
			user::rw-
			user:xerox:rwx
			group::r--
			mask::rwx
			other::r--
			```
- 创建一个让root都无法删除的文件
	- ```chattr```, ```lsattr```

	```
	[root@localhost ~]# touch hack.sh
	[root@localhost ~]# ll hack.sh 
	-rw-r--r--. 1 root root 0 Feb 26 09:57 hack.sh
	[root@localhost ~]# chattr +i hack.sh 
	[root@localhost ~]# rm -rf hack.sh 
	rm: cannot remove 'hack.sh': Operation not permitted
	[root@localhost ~]# ll hack.sh # 普通命令查看不到扩展权限
	-rw-r--r--. 1 root root 0 Feb 26 09:57 hack.sh
	[root@localhost ~]# lsattr hack.sh #只有专用命令可以查看到扩展权限
	----i----------- hack.sh
	```
	
	- 从REHL6 开始，新增加文件系统扩展属性：命令：chattr  参数：  a  只能追加内容   ；  i  不能被修改
		- +a: 只能追加内容  如： echo aaa  >> hack.sh
		- +i：即Immutable，系统不允许对这个文件进行任何的修改。如果目录具有这个属性，那么任何的进程只能修改目录之下的文件，不允许建立和删除文件。注：immutable  [ɪˈmju:təbl]  不可改变的  ；  Append [əˈpend] 追加
		- -i ：移除i参数。  -a :移除a参数









-----

17. 用户管理, **直接修改 vim /etc/passwd**

- Linux用户三种角色
	- 超级用户： root  拥有对系统的最高的管理权限  ID=0
	- 普通用户：包括系统用户，本地用户

	|user|centos7|centos6|
	|-|-|-|
	|系统用户 |UID:1-999(centos7版本)| 1-499(centos6版本)|
	|本地用户 |UID:1000+| 500+|
	
	- 虚拟用户：伪用户，一般不会用来登录系统的，它主要是用于维持某个服务的正常运行.如：ftp，apache
- 用户配置文件

|名 称|帐号信息|说 明|
|-|-|-|
|用户配置文件|/etc/passwd|记录了每个用户的一些基本属性，并且对所有用户可读，每一行记录对应一个用户，每行记录通过冒号进行分隔|
|用户组文件|/etc/group|用户组的所有信息存放地儿，并且组名不能重复|
|用户对应的密码信息|/etc/shadow|因为passwd文件对所有用户是可读的，为安全起见把密码从passwd中分离出来放入这个单独的文件，该文件只有root用户拥有读权限，从而保证密码安全性|


- 创建用户 ```useradd -d "用户主目录路径，家目录" -u “UID” -g "初始组" -G "附加组" -s "登陆的shell” 用户名```
	- 创建一个新用户```useradd david```
		```
		[root@localhost ~]# useradd david
		[root@localhost ~]# tail -1 /etc/passwd
		david:x:1001:1001::/home/david:/bin/bash
		```
		- david：用户名
		- x：密码占位符
		- 1001：用户的UID，它都是用数字来表示的
		- 1001：用户所属组的GID，它都是用数字来表示的
		- [BLANK]：用户描述信息：对用户的功能或其它来进行一个简要的描述
		- /home/david：用户主目录（shell提示符中“~”代表的那个）
		- /bin/bash：用户登录系统后使用的shell
	- 指定用户id ```useradd -u 1111 frank```
		```
		[root@localhost ~]# useradd -u 1111 frank
		[root@localhost ~]# id frank
		uid=1111(frank) gid=1111(frank) groups=1111(frank)
		```
	- 指定用户主目录
		```
		[root@xuegod63 ~]# useradd  -d /opt/frank frank
		[root@xuegod63 ~]# tail -1 /etc/passwd
		frank:x:1102:1102::/opt/frank:/bin/bash
		```
	- 指定用户组
		```
		useradd  -g david frank
		[root@xuegod63 ~]# id frank
		uid=1104(frank) gid=1103(david) 组=1103(david)
		```
- 删除用户
	- ```userdel -r frank```, -r 删除的时候，会同时删除用户的家目录和/var/mail下的目录

- 更改用户 ```usermod```
	- ```usermod -m -d /mnt/frank frank```, -m选项会自动创建新目录并且移动内容到新目录里面

	|参数|说明|
	|-|-|
	|-u|	UID|
	|-d|	宿主目录|
	|-g|	起始组	#只能有一个|
	|-G|	附加组	#可以有多个|
	|-s|	登录shell|
	|-L | 锁定|

-----

16. vim
- i 光标签插入， a 光标后插入
- I 行首插入， A 行尾插入
- o 下一行（新起一行）插入， O 上一行（新起一行）插入
- x 向后删除一个字符,等同于delete, X 向前删除一个字符     	
- u 撤销一步,每按一次就撤销一次
- ctrl+r  还原撤销过的操作，将做过的撤销操作再还原回去
- r 替换
- 0 和home键表示切换到行首， $和end键表示切换到行尾
- gg 快速定位到文档的首行 ,  G定位到未行
- 3gg 或者3G，快速定位到第3行
- /string(字符串)，找到或定位你要找的单词或内容，如果相符内容比较多，我们可以通过N、n来进行向上向下查找。/^d，^意思表示以什么开头，查找以字母d开头的内容。/t$，$意思表示以什么结尾，查找以字母t结尾的内容
- yy 复制整行。复制N行： Nyy  ，比如： 2yy ，表示复制2行
- dd（删除，以行为单位，删除当前光标所在行）,删除N行： Ndd  ，比如： 2dd ，表示删除2行
- p 粘贴


- 多行注释
	- ctrl+v 进入列编辑模式
	- 向下或向上移动光标，把需要注释、编辑的行的开头选中起来
	- 然后按大写的I
	- 再插入注释符或者你需要插入的符号,比如"#"
	- 再按Esc,就会全部注释或添加了
- 文本替换,  范围（其中%所有内容）   s分隔符 旧的内容 分隔符 新的内容  （分隔符可以自定义）
	- :1,3 s/bin/xuegod    替换第1到3行中出现的第一个bin进行替换为xuegod
	- :1,3 s/bin/xuegod/g  替换第1到3行中查找到所有的bin进行替换为xuegod
	- :3 s/xue/aaaaa     #只把第3行中内容替换了
	- :% s/do/xuegod/g  	将文本中所有的do替换成xuegod
	- :% s/do/xuegod/gi	将文本中所有的do替换成xuegod, 并且忽略do的大小写
	- :% s@a@b@g	   将文本中所有的a替换成b
	

- :!ifconfig, 调用系统命令
- :r /etc/hosts, 读取其他文件。（把其他文件中的内容追加到当前文档中）
- :set nu  设置行号
- :set nonu 取消设置行号
- :noh   取消高亮显示
- vim -o /etc/passwd /etc/hosts 上下分开
- vim -O /etc/passwd /etc/hosts 左右分开，ctrl+ww  在两文档之间进行切换编辑
- vimdiff /etc/passwd mima.txt 查看2个文档的不同，高亮表示

-----
15. xfs文件系统的备份和恢复，xfsdump xfsrestore

XFS提供了 xfsdump 和 xfsrestore 工具协助备份XFS文件系统中的数据。xfsdump 按inode顺序备份一个XFS文件系统。centos7选择xfs格式作为默认文件系统，而且不再使用以前的ext，仍然支持ext4，xfs专为大数据产生，每个单个文件系统最大可以支持8eb，单个文件可以支持16tb，不仅数据量大，而且扩展性高。还可以通过xfsdump，xfsrestore来备份和恢复。

与传统的UNIX文件系统不同，XFS不需要在备份前被卸载；对使用中的XFS文件系统做备份就可以保证镜像的一致性。XFS的备份和恢复的过程是可以被中断然后继续，无须冻结文件系统。xfsdump 甚至提供了高性能的多线程备份操作——它把一次dump拆分成多个数据流，每个数据流可以被发往不同的目的地。

xfsdump的备份级别有以下两种，默认为0（即完全备份）

|级别|说明|
|-|-|
|0 级别| 完全备份，每次都把指定的备份目录完整的复制一遍，不管目录下的文件有没有变化|
|1 到9级别|增量备份，每次将之前（第一次、第二次、直到前一次）做过备份之后有变化的文件进行备份|

fdisk挂载新硬盘，vm中创建新硬盘。安装系统时已默认挂载sda，新硬盘识别为sdb

- 创建新分区partition
  - ```fdisk sdb```
  - ```n```, enter, enter
  - ```+1G```
  - ```w```,write for create a new partition. 
  - ```fdisk /dev/sdb``` -> ```p```, 查看此硬盘下已有分区partition
    ```
    [root@localhost b]# fdisk /dev/sdb
    Welcome to fdisk (util-linux 2.23.2).

    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.


    Command (m for help): n
    Partition type:
       p   primary (2 primary, 0 extended, 2 free)
       e   extended
    Select (default p): 
    Using default response p
    Partition number (3,4, default 3): 
    First sector (4196352-41943039, default 4196352): 
    Using default value 4196352
    Last sector, +sectors or +size{K,M,G} (4196352-41943039, default 41943039): +1G
    Partition 3 of type Linux and of size 1 GiB is set

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.
    ```
  - 查看计算机已有硬盘分区
    - ```df -h```
      ```
      [root@localhost sdb2]# df -h
      Filesystem      Size  Used Avail Use% Mounted on
      /dev/sda2        10G  4.1G  6.0G  41% /
      devtmpfs        3.9G     0  3.9G   0% /dev
      tmpfs           3.9G     0  3.9G   0% /dev/shm
      tmpfs           3.9G   13M  3.9G   1% /run
      tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
      /dev/sda1       197M  141M   56M  72% /boot
      tmpfs           798M  8.0K  798M   1% /run/user/42
      tmpfs           798M     0  798M   0% /run/user/0
      /dev/sdb3      1014M   33M  982M   4% /root/sdb3
      ```
   
- 使用新分区前，先进行格式化
  - ```mkfs.xfs /dev/sdb3```
    ```
    [root@localhost ~]# mkfs.xfs /dev/sdb3
    meta-data=/dev/sdb3              isize=512    agcount=4, agsize=65536 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=1        finobt=0, sparse=0
    data     =                       bsize=4096   blocks=262144, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
    log      =internal log           bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0
    ```
- 挂载新分区至制定目录下，新分区可以正常使用
  - ```mkdir ./sdb3```
  - ```mount /dev/sdb3 /sdb3```

再新分区sdb2中创建一些文件，下面操作将sdb2进行备份。

- xfsdump -f 备份存放位置 要备份路径或设备文件  
  - ```xfsdump -f ```
    ```
    [root@localhost ~]# xfsdump -f /opt/sdb3_dump /sdb3
    xfsdump: using file dump (drive_simple) strategy
    xfsdump: version 3.1.7 (dump format 3.0) - type ^C for status and control

     ============================= dump label dialog ==============================

    please enter label for this dump session (timeout in 300 sec)
     -> dump_sdb3
    session label entered: "dump_sdb3"

     --------------------------------- end dialog ---------------------------------

    xfsdump: level 0 dump of localhost.localdomain:/sdb3
    xfsdump: dump date: Mon Feb 25 11:23:45 2019
    xfsdump: session id: 05b2aeb0-8426-4850-8078-e09cdbf50212
    xfsdump: session label: "dump_sdb3"
    xfsdump: ino map phase 1: constructing initial dump list
    xfsdump: ino map phase 2: skipping (no pruning necessary)
    xfsdump: ino map phase 3: skipping (only one dump stream)
    xfsdump: ino map construction complete
    xfsdump: estimated dump size: 20800 bytes
    xfsdump: /var/lib/xfsdump/inventory created

     ============================= media label dialog =============================

    please enter label for media in drive 0 (timeout in 300 sec)
     -> media
    media label entered: "media"

     --------------------------------- end dialog ---------------------------------

    xfsdump: creating dump session media file 0 (media 0, file 0)
    xfsdump: dumping ino map
    xfsdump: dumping directories
    xfsdump: dumping non-directory files
    xfsdump: ending media file
    xfsdump: media file size 21016 bytes
    xfsdump: dump size (non-dir files) : 0 bytes
    xfsdump: dump complete: 13 seconds elapsed
    xfsdump: Dump Summary:
    xfsdump:   stream 0 /opt/sdb3_dump OK (success)
    xfsdump: Dump Status: SUCCESS
    ```
  - ```[root@xuegod63 sdb1]# xfsdump -f /opt/dump_passwd /sdb1 -L dump_passwd -M media1```,指定备份时免交互操作，方便后期做定时备份, -L  ：xfsdump  纪录每次备份的 session 标头，这里可以填写针对此文件系统的简易说明。 -M  ：xfsdump 可以纪录储存媒体的标头，这里可以填写此媒体的简易说明

- 查看备份信息与内容
  - ```xfsdump -I```，大写字母i
    ```
    [root@localhost ~]# xfsdump -I
    file system 0:
      fs id:		f0071f64-9967-4848-a3af-08232e5ee4aa
      session 0:
        mount point:	localhost.localdomain:/sdb3
        device:		localhost.localdomain:/dev/sdb3
        time:		Mon Feb 25 11:23:45 2019
        session label:	"dump_sdb3"
        session id:	05b2aeb0-8426-4850-8078-e09cdbf50212
        level:		0
        resumed:	NO
        subtree:	NO
        streams:	1
        stream 0:
          pathname:	/opt/sdb3_dump
          start:		ino 0 offset 0
          end:		ino 1 offset 0
          interrupted:	NO
          media files:	1
          media file 0:
            mfile index:	0
            mfile type:	data
            mfile size:	21016
            mfile start:	ino 0 offset 0
            mfile end:	ino 1 offset 0
            media label:	"media"
            media id:	8297b6b5-d9f8-4c54-bc33-a1c97eb7c92d
    xfsdump: Dump Status: SUCCESS    
    ```
  - ```ls /var/lib/xfsdump/inventory/```，可以查到上述fs id f0071f64-9967-4848-a3af-08232e5ee4aa.InvIndex

- 恢复备份
  - 先删除原有文件
    - ```cd sdb3```
    - ```rm -rf ./*```
  - xfsrestore -f 指定备份文件的存放位置 指定存放恢复后的文件的路径
    - ```xfsrestore -f /opt/sdb3_dump /sdb3```
      ```
      [root@localhost ~]# xfsrestore -f /opt/sdb3_dump /sdb3
      xfsrestore: using file dump (drive_simple) strategy
      xfsrestore: version 3.1.7 (dump format 3.0) - type ^C for status and control
      xfsrestore: searching media for dump
      xfsrestore: examining media file 0
      xfsrestore: dump description: 
      xfsrestore: hostname: localhost.localdomain
      xfsrestore: mount point: /sdb3
      xfsrestore: volume: /dev/sdb3
      xfsrestore: session time: Mon Feb 25 11:23:45 2019
      xfsrestore: level: 0
      xfsrestore: session label: "dump_sdb3"
      xfsrestore: media label: "media"
      xfsrestore: file system id: f0071f64-9967-4848-a3af-08232e5ee4aa
      xfsrestore: session id: 05b2aeb0-8426-4850-8078-e09cdbf50212
      xfsrestore: media id: 8297b6b5-d9f8-4c54-bc33-a1c97eb7c92d
      xfsrestore: using online session inventory
      xfsrestore: searching media for directory dump
      xfsrestore: reading directories
      xfsrestore: 1 directories and 0 entries processed
      xfsrestore: directory post-processing
      xfsrestore: restore complete: 0 seconds elapsed
      xfsrestore: Restore Summary:
      xfsrestore:   stream 0 /opt/sdb3_dump OK (success)
      xfsrestore: Restore Status: SUCCESS

      
      ```

-----

14. 查看文件
  - ```cat /etc/passwd```
  - ```more /etc/passwd```, 按下回车刷新一行，按下空格刷新一屏，输入q键退出。不支持后退
  - ```less /etc/passwd```, 支持前后翻滚，既可以向上翻页（pageup按键），也可以向下翻页（pagedown按键）
  - ```head -n 3 /etc/passwd```, 只看前三行 默认不加-n时显示前十行
  - ```tail -n 3 /var/log/secure```, 查看最后3行记录
  - ```tail -f /var/log/secure```, 动态查看文件内容, 不关闭
  
13. ```mv a.txt dir1/abc.txt```, 在移动文件的时候支持改名操作,a->abc

12. cp 源文件/目录  目录文件/目录
- ```cp -r /boot/grup /opt```, -R/r：递归处理，将指定目录下的所有文件与子目录一并处理

11. ```rm -rf```, -f 强制删除没有提示, -r 删除目录


10. mkdir
- ```mkdir -p /tmp/a/b/c```,在创建一个目录的时候，如果这个目录的上一级不存在的话，要加参数-p = parent父目录

9. touch
- ```touch file{1..8}.txt```, file1, file2, ...file8


8. 查看文件状态
- ```ll /etc/passwd```, -rw-r--r--. 1 root root 2307 Feb 22 13:21 /etc/passwd
- ```stat /etc/passwd```

  ```
      File: '/etc/passwd'
    Size: 2307      	Blocks: 8          IO Block: 4096   regular file
  Device: 802h/2050d	Inode: 8443595     Links: 1
  Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
  Context: system_u:object_r:passwd_file_t:s0
  Access: 2019-02-24 13:30:01.853163925 +0800 # 访问时间：atime  查看内容  cat  a.txt
  Modify: 2019-02-22 13:21:28.562334692 +0800 # 修改时间：mtime  修改内容  vim a.txt
  Change: 2019-02-22 13:21:28.563334692 +0800 # 改变时间：ctime  文件属性，比如权限change time, chmod +x a.sh
   Birth: -
  ```







1. ```systemctl status NetworkManager```, #查看networkmanager服务是是否启动

2. RHEL/CENTOS Linux网络相关的配置文件/etc/sysconfig/network-scripts

  - ```ls /etc/sysconfig/network-scripts/ifcfg-ens33```, #IP地址，子网掩码等配置文件

  - ```ls /etc/sysconfig/network-scripts/ifcfg-lo```, #网卡回环地址

  - ```cat /etc/resolv.conf```, #DNS配置文件

  - ```cat /etc/hosts```, #设置主机和IP绑定信息

  - ```cat /etc/hostname```, #设置主机名

3. 修改ip地址
  - 方法1: ```nmtui```, 使用nmtui文本框方式修改IP，重启网卡服务生效：```systemctl restart network``` ---重启服务
  - 方法2: ```vim /etc/sysconfig/network-scripts/ifcfg-ens33```

4. 关闭防火墙并设置开机开不启动
  - ```systemctl status firewalld.service```    #查看firewalld状态
  - ```systemctl stop firewalld```       #关闭
  - ```systemctl start firewalld```       #开启
  - ```systemctl disable firewalld```     #开机自动关闭   //RHLE7
  - ```chkconfig --list|grep network```    #查看开机是否启动   //RHLE6
  - ```systemctl enable firewalld```     #开机自动启动

5. 设置系统光盘开机自动挂载
  - ```vim  /etc/fstab```  #在文档最后，添加以以下内容：
/dev/cdrom 			      /mnt			  iso9660 defaults        0 0
  - ```mount -a```
mount: /dev/sr0 写保护，将以只读方式挂载
  - ```ls /mnt/```   #可以查看到此目录下有内容，说明挂载成功
CentOS_BuildTag  GPL       LiveOS    RPM-GPG-KEY-CentOS-7

6. 配置本地YUM源
yum的一切配置信息都储存在一个叫yum.repos.d的配置文件中，通常位于/etc/yum.repos.d目录下 
  - ```rm -rf  /etc/yum.repos.d/*```, 删除原有的文件, 也可以不删除 
  - ```vim  CentOS7.repo```  #创建一个新的yum源配置文件，yum源配置文件的结尾必须是.repo, 写入以下内容
        
        [CentOS7]        # --->yum的ID，必须唯一 
        name=CentOS-server    #  ----->描述信息
        baseurl=file:///mnt    # -------> /mnt表示的是光盘的挂载点  . file:后面有3个///
        enabled=1   # ------>启用
        gpgcheck=0   # ---->取消验证

  - ```yum clean all```			#清空并生成缓存列表,清空yum缓存
  - ```yum list```						#生成缓存列表
  - ```yum -y install httpd``` # 验证一下

## 2019-02-25

7. install tree package
  - ```mount /dev/sr0 /media/```, # 挂在磁盘与media目录，mount: /dev/sr0 写保护，将以只读方式挂载， 也可以挂在于/mnt，此目录默认为空
  - ``` rpm -ivh /media/Packages/tree-1.6.0-10.el7.x86_64.rpm```
  
8. centos 各目录介绍
   
|目录|说明|
|-|-|
|/|root，处于linux系统树形结构的最顶端，它是linux文件系统的入口，所有的目录、文件、设备都在 / 之下|
|/bin|bin是Binary的缩写。常用的二进制命令目录。比如 ls、cp、mkdir、cut等；和/usr/bin类似，一些用户级gnu工具|
|/boot|存放的系统启动相关的文件，例如：kernel.grub(引导装载程序)|
|/dev | dev是Device的缩写。设备文件目录，比如声卡、磁盘...在Linux中一切都被看做文件。终端设备、磁盘等等都被看做文件。设备文件:/dev/sda,/dev/sda1,/dev/tty1,/dev/tty2,/dev/pts/1, /dev/zero, /dev/null, /dev/cdrom |
|/etc|常用系统及二进制安装包配置文件默认路径和服务器启动命令目录。passwd 用户信息文件，shadow  用户密码文件，group 存储用户组信息，fstab 系统开机启动自动挂载分区列表，hosts 设定用户自己的IP与主机名对应的信息。|
|/home|普通用户的家目录默认存放目录 |
|/lib|库文件存放目录,函数库目录|
|/mnt, /media|一般用来临时挂载存储设备的挂载目录，比如有cdrom、U盘等目录,在CENTOS7中会挂载到/run下面,/run目录默认不为空|
|/opt|option表示的是可选择的意思，有些软件包也会被安装在这里 |
|/proc|操作系统运行时，进程（正在运行中的程序）信息及内核信息（比如cpu、硬盘分区、内存信息等）存放在这里。/proc目录是伪装的文件系统proc的挂载目录，proc并不是真正的文件系统。因此，这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。也就是说，这个目录的内容不在硬盘上而是在内存里.查看CPU信息 ```cat /proc/cpuinfo``` |
|/sys|系统目录，存放硬件信息的相关文件|
|/run|运行目录，存放的是系统运行时的数据，比如进程的PID文件|
|/srv|服务目录，存放的是我们本地服务的相关文件|
|/sbin|大多数涉及系统管理的命令都存放在该目录中，它是超级权限用户root的可执行命令存放地，普通用户无权限执行这个目录下的命令，凡是目录sbin中包含的命令都是root权限才能执行的  |
|/tmp|该目录用于存放临时文件，有时用户运行程序的时候，会产生一些临时文件。/tmp就是用来存放临时文件的。/var/tmp目录和该目录的作用是相似的。不能存放重要数据，它的权限比较特殊 ```ls –ld /tmp # -d只看目录，不看下面发文件``` -> ```drwxrwxrwt 10 root root 12288 Oct 3 20:45 /tmp/``` →粘滞位（sticky bit）目录的sticky位表示这个目录里的文件只能被owner和root删除|
|/var|系统运行和软件运行时产生的日志信息，该目录的内容是经常变动的，存放的是一些变化的文件。比如/var下有/var/log目录用来存放系统日志的目录，还有mail、/var/spool/cron   |
|/usr|存放应用程序和文件，/usr/bin 普通用户使用的应用程序，/usr/sbin 管理员使用的应用程序，/usr/lib 库文件Glibc(32位)，/usr/lib64 库文件Glibc|
|/lib, /lib64 都在/usr目录下| 这个目录里存放着系统最基本的动态链接共享库，包含许多被/bin/和/sbin/中的程序使用的库文件，目录/usr/lib/中含有更多用于用户程序的库文件。作用类似于windows里的DLL文件，几乎所有的应用程序都需要用到这些共享库。注：lib***.a是静态库，lib***.so是动态库。静态库在编译时被加载到二进制文件中，动态库在运行时加载到进程的内存空间中，简单的说：这些库是为了让你的程序能够正常编译运行的，其实类似于WIN中.dll文件，几乎所有的应用程序都需要用到这些共享库|
|/lost+found 只在centos6中有|默认为空，被FSCK（file system check用来检查和维护不一致的文件系统。若系统掉电或磁盘发生问题，可利用fsck命令对文件系统进行检查）用来放置零散文件（没有名称的文件） 当系统非法关机后，这里就会存放一些文件。在centos6版本下，每个分区的挂载点下会有些目录|

