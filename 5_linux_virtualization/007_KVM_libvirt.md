
# libvirt virsh

QEMU opens the device fle (/dev/kvm) exposed by the KVM kernel module and
executes ioctls() on it. To conclude, KVM makes use of QEMU to become a complete
hypervisor, and KVM is an accelerator or enabler of the hardware virtualization
extensions (VMX or SVM) provided by the processor to be tightly coupled with the
CPU architecture. Indirectly, this conveys that virtual systems also have to use the
same architecture to make use of hardware virtualization extensions/capabilities.
Once it is enabled, it will defnitely give better performance than other techniques
such as binary translation.

Once again, let me say that KVM is not a hypervisor! Are you lost? OK, then let me
rephrase that. The KVM or kernel-based virtual machine is not a full hypervisor;
however, with the help of QEMU and emulators (a slightly modifed QEMU for I/O
device emulation and BIOS), it can become one. KVM needs hardware virtualizationcapable processors to operate. Using these capabilities, KVM turns the standard
Linux kernel into a hypervisor. When KVM runs virtual machines, every VM is
a normal Linux process, which can obviously be scheduled to run on a CPU by
the host kernel as with any other process present in the host kernel.

KVM enables virtualization and readies your server or workstation to host the
virtual machines. In technical terms, KVM is a set of kernel modules for an x86
architecture hardware with virtualization extensions; when loaded, it converts a
Linux server into a virtualization server (hypervisor). The loadable modules are
kvm.ko, which provides the core virtualization capabilities and a processor-specifc
module, kvm-intel.ko or kvm-amd.ko.

Quick Emulator (QEMU) is an open source machine emulator. This emulator will
help you to run the operating systems that are designed to run one architecture on
top of another one. For example, Qemu can run an OS created on the ARM platform
on the x86 platform; however, there is a catch here. Since QEMU uses dynamic
translation, which is a technique used to execute virtual machine instructions
on the host machine, the VMs run slow.

If QEMU is slow, how can it run blazing fast KVM-based virtual machines at a near
native speed? KVM developers thought about the problem and modifed QEMU as
a solution. This modifed QEMU is called qemu-kvm, which can interact with KVM
modules directly and safely execute instructions from the VM directly on the CPU
without using dynamic translations. In short, we use qemu-kvm binary to run the
KVM-based virtual machines.

It is getting more and more confusing, right? If qemu-kvm can run a virtual machine,
then why do you need to use libvirt. The answer is simple, libvirt manages
qemu-kvm and qemu-kvm runs the KVM virtual machines.

Libvirt will take care of the storage, networking, and virtual hardware requirements
to start a virtual machine along with VM lifecycle management.


- ```yum install qemu-kvm libvirt virt-install virt-manager virt-install -y```
- ```systemctl enable libvirtd && systemctl start libvirtd```





