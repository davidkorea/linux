
# libvirt virsh

QEMU opens the device fle (/dev/kvm) exposed by the KVM kernel module and executes ioctls() on it. To conclude, KVM makes use of QEMU to become a complete hypervisor, and KVM is an accelerator or enabler of the hardware virtualization extensions (VMX or SVM) provided by the processor to be tightly coupled with the CPU architecture. Indirectly, this conveys that virtual systems also have to use the same architecture to make use of hardware virtualization extensions/capabilities. Once it is enabled, it will defnitely give better performance than other techniques such as binary translation.

Once again, let me say that KVM is not a hypervisor! Are you lost? OK, then let me rephrase that. The KVM or kernel-based virtual machine is not a full hypervisor; however, with the help of QEMU and emulators (a slightly modifed QEMU for I/O device emulation and BIOS), it can become one. KVM needs hardware virtualizationcapable processors to operate. Using these capabilities,  VM turns the standard Linux kernel into a hypervisor. When KVM runs virtual machines, every VM is a normal Linux process, which can obviously be scheduled to run on a CPU by the host kernel as with any other process present in the host kernel.

KVM enables virtualization and readies your server or workstation to host the virtual machines. In technical terms, KVM is a set of kernel modules for an x86 architecture hardware with virtualization extensions; when loaded, it converts a Linux server into a virtualization server (hypervisor). The loadable modules are kvm.ko, which provides the core virtualization capabilities and a processor-specifc module, kvm-intel.ko or kvm-amd.ko.

Quick Emulator (QEMU) is an open source machine emulator. This emulator will help you to run the operating systems that are designed to run one architecture on top of another one. For example, Qemu can run an OS created on the ARM platform on the x86 platform; however, there is a catch here. Since QEMU uses dynamic translation, which is a technique used to execute virtual machine instructions on the host machine, the VMs run slow.

If QEMU is slow, how can it run blazing fast KVM-based virtual machines at a near native speed? KVM developers thought about the problem and modifed QEMU as a solution. This modifed QEMU is called qemu-kvm, which can interact with KVM modules directly and safely execute instructions from the VM directly on the CPU without using dynamic translations. In short, we use qemu-kvm binary to run the KVM-based virtual machines.

It is getting more and more confusing, right? If qemu-kvm can run a virtual machine, then why do you need to use libvirt. The answer is simple, libvirt manages qemu-kvm and qemu-kvm runs the KVM virtual machines.

Libvirt will take care of the storage, networking, and virtual hardware requirements to start a virtual machine along with VM lifecycle management.


- ```yum install qemu-kvm libvirt virt-install virt-manager virt-install -y```
- ```systemctl enable libvirtd && systemctl start libvirtd```
- ```virt-host-validate```
  - Executing this command as root user will perform sanity checks on KVM capabilities to validate that the host is configured in a
suitable way to run the libvirt hypervisor drivers using KVM virtualization.
  ```
  [root@server162 ~]# virt-host-validate
  QEMU: 正在检查 for hardware virtualization                                 : PASS
  QEMU: 正在检查 if device /dev/kvm exists                                   : PASS
  QEMU: 正在检查 if device /dev/kvm is accessible                            : PASS
  QEMU: 正在检查 if device /dev/vhost-net exists                             : PASS
  QEMU: 正在检查 if device /dev/net/tun exists                               : PASS
  QEMU: 正在检查 for cgroup 'memory' controller support                      : PASS
  QEMU: 正在检查 for cgroup 'memory' controller mount-point                  : PASS
  QEMU: 正在检查 for cgroup 'cpu' controller support                         : PASS
  QEMU: 正在检查 for cgroup 'cpu' controller mount-point                     : PASS
  QEMU: 正在检查 for cgroup 'cpuacct' controller support                     : PASS
  QEMU: 正在检查 for cgroup 'cpuacct' controller mount-point                 : PASS
  QEMU: 正在检查 for cgroup 'cpuset' controller support                      : PASS
  QEMU: 正在检查 for cgroup 'cpuset' controller mount-point                  : PASS
  QEMU: 正在检查 for cgroup 'devices' controller support                     : PASS
  QEMU: 正在检查 for cgroup 'devices' controller mount-point                 : PASS
  QEMU: 正在检查 for cgroup 'blkio' controller support                       : PASS
  QEMU: 正在检查 for cgroup 'blkio' controller mount-point                   : PASS
  QEMU: 正在检查 for device assignment IOMMU support                         : WARN (No ACPI DMAR table found, 
                                    IOMMU either disabled in BIOS or not supported by this hardware platform)
   LXC: 正在检查 用于 Linux >= 2.6.26                                      : PASS
   LXC: 正在检查 for namespace ipc                                           : PASS
   LXC: 正在检查 for namespace mnt                                           : PASS
   LXC: 正在检查 for namespace pid                                           : PASS
   LXC: 正在检查 for namespace uts                                           : PASS
   LXC: 正在检查 for namespace net                                           : PASS
   LXC: 正在检查 for namespace user                                          : PASS
   LXC: 正在检查 for cgroup 'memory' controller support                      : PASS
   LXC: 正在检查 for cgroup 'memory' controller mount-point                  : PASS
   LXC: 正在检查 for cgroup 'cpu' controller support                         : PASS
   LXC: 正在检查 for cgroup 'cpu' controller mount-point                     : PASS
   LXC: 正在检查 for cgroup 'cpuacct' controller support                     : PASS
   LXC: 正在检查 for cgroup 'cpuacct' controller mount-point                 : PASS
   LXC: 正在检查 for cgroup 'cpuset' controller support                      : PASS
   LXC: 正在检查 for cgroup 'cpuset' controller mount-point                  : PASS
   LXC: 正在检查 for cgroup 'devices' controller support                     : PASS
   LXC: 正在检查 for cgroup 'devices' controller mount-point                 : PASS
   LXC: 正在检查 for cgroup 'blkio' controller support                       : PASS
   LXC: 正在检查 for cgroup 'blkio' controller mount-point                   : PASS
   LXC: 正在检查 if device /sys/fs/fuse/connections exists                   : PASS
   ```
- ```virsh nodeinfo```
  - physical node's system resource information
  ```
  [root@server162 ~]# virsh nodeinfo 
  CPU 型号：        x86_64
  CPU：               4
  CPU 频率：        2393 MHz
  CPU socket：        2
  每个 socket 的内核数： 2
  每个内核的线程数： 1
  NUMA 单元：       1
  内存大小：      8388084 KiB
  ```
- ```virsh domcapabilities```
  -  displays an XML document describing the capabilitiesof qemu-kvm with respect to the host and libvirt version.  It will help you determine the type of virtual disks you can use with the virtual machines, the maximum number of vCPUs that can be assigned, and so on.

  - vcpus
    ```xml
    [root@server162 ~]# virsh domcapabilities | grep -i max
    <vcpu max='240'/>
    ```
    - on this host a maximum of 240 vcpus can be defned for a virtual machine
  - disk device
    ```xml
    [root@server162 ~]# virsh domcapabilities | grep diskDevice -A 5
      <enum name='diskDevice'>
        <value>disk</value>
        <value>cdrom</value>
        <value>floppy</value>
        <value>lun</value>
      </enum>
    ```
    -  disk, cdrom, floppy, and lun type devices can be used with the virtual machine on this host
    
    
# 1. virt-manager 
## 1.1 Introducing virt-manager 
**go to Edit | Connection Details**
### 1 Overview

### 2 Virtual Networks
- **NATed virtual network**
A NAT-based virtual network provides outbound network connectivity to the virtual machines. That means the VMs can communicate with the outside network based on the network connectivity available on the host but none of the outside entities will be able to communicate with the VMs. In this setup, the virtual machines and host should be able to communicate with each other through the bridge interface confgured on the host.
- **Routed virtual network**
A routed virtual network allows the connection of virtual machines directly to the physical network. Here VMs will send out packets to the outside network based on the routing rules set on the hypervisor.
- **Isolated virtual network**
As the name implies, this provides a private network between the hypervisor and the virtual machines.
    
```
[root@server162 ~]# virsh net-list --all
 名称               状态     自动开始  持久
----------------------------------------------------------
 default              活动     是           是
```
```
[root@server162 ~]# virsh net-info default
名称：       default
UUID:           ed954729-1466-454a-b3a2-19c4f1778388
活跃：       是
持久：       是
自动启动： 是
桥接：       virbr0
```   
```xml
[root@server162 ~]# virsh net-dumpxml default
<network>
  <name>default</name>
  <uuid>ed954729-1466-454a-b3a2-19c4f1778388</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:99:40:2f'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```
- Virtual network configuration files are stored in /etc/libvirt/qemu/networks/ as XML files. For the default network it is /etc/libvirt/qemu/networks/default.xml
### 3 Storage
The location of this storage pool is in /var/lib/libvirt/images.
    
## 1.2 Creating virtual machines using virt-manager

The following methods are available with virt-manager for Guest OS installation:
#### 1. Local installation media (ISO Image or CD-ROM)
#### 2. Network installation (HTTP, FTP, or NFS)
  - url: https://mirrors.aliyun.com/centos/7.6.1810/os/x86_64/
#### 3. Network boot (PXE)
- PXE Guest installation requires a PXE server running on the same subnet where you wish to create the virtual machine and the host system must have network connectivity to the PXE server.
- The default NATed network created by virt-manager is not compatible with PXE installation, because a virtual machine connected to the NAT does not appear on the network as its own device, and therefore the PXE server can't see it and can't send the required data to perform the installation. 
- select "主机设备 ens33：macvtap"  - "桥接"

#### 4. Importing existing disk images
- import a pre-installed and confgured disk image instead of doing a manual installation
    
    
    
    
    
    
    
    
    
    
    
    
    
    
