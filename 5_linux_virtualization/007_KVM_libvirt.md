
# libvirt virsh

QEMU opens the device fle (/dev/kvm) exposed by the KVM kernel module and
executes ioctls() on it. Please refer to the next section on KVM to know more
about these ioctls(). To conclude, KVM makes use of QEMU to become a complete
hypervisor, and KVM is an accelerator or enabler of the hardware virtualization
extensions (VMX or SVM) provided by the processor to be tightly coupled with the
CPU architecture. Indirectly, this conveys that virtual systems also have to use the
same architecture to make use of hardware virtualization extensions/capabilities.
Once it is enabled, it will defnitely give better performance than other techniques
such as binary translation.
