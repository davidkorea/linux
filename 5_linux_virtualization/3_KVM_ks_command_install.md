# 字符界面安装 KVM-阿里云主机使用方法 

1. 在命令行安装 KVM 虚拟机 
2. 在命令行无人执守安装 KVM 虚拟机 
3. 阿里云主机使用方法 

- 报错空间不足
```shell
virt-install --name centos7_ks --ram 1024 --vcpus=1 --disk path=/var/lib/libvirt/images/centos7_ks.qcow2,size=13 --accelerate --location=http://192.168.0.162/centos7/ --network bridge=br0 -x "ks=http://192.168.0.162/ks.cfg"
```


- 正常安装，扩大内存至2G，硬盘位20G
```shell
virt-install --name kvm_centos7_ks --ram 2048 --vcpus=1 --disk path=/var/lib/libvirt/images/kvm_centos7_ks.qcow2,size=20 --accelerate --location=http://192.168.0.163/centos7/ --network bridge=br0 -x "ks=http://192.168.0.163/ks.cfg"
```
使用的ks，cfg文件如下
```shell
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Install OS instead of upgrade
install
# Keyboard layouts
keyboard 'us'
# Root password
rootpw --iscrypted $1$ylWwkW5v$Zpe1fUpFalYQ79qQ4VhxO1
# Use network installation
url --url="http://192.168.0.163/centos7"
# System language
lang en_US
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use graphical install
graphical
firstboot --disable
# SELinux configuration
selinux --disabled

# Firewall configuration
firewall --disabled
# Reboot after installation
reboot
# System timezone
timezone Asia/Shanghai
# System bootloader configuration
bootloader --location=mbr
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part /boot --fstype="xfs" --size=300
part swap --fstype="swap" --size=2000
part / --fstype="xfs" --grow --size=1

%packages
@additional-devel
@development
@platform-devel

%end


```
