
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
