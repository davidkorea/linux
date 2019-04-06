# kvm-openstack-docker-kubernetes

# Visualization
- 模拟器emulization。可以虚拟和宿主机不同的系统
  - QEMU，PearPC，Bochs等技术
  - 需要将不同架构的虚拟机的指令，经过转换，在转换为宿主机支持的指令。因此，指令集的转换效率低
- 完全虚拟化 full vitualization, native virtualizatiom. 虚拟机需要和宿主机的架构完全相同。如果宿主机是x86 64bit，那么虚拟机也必须相同。
  - 需要VMM虚拟机监视器来监控虚拟机的指令，在宿主机转换后，再将结果返回给虚拟机
  - 硬件辅助虚拟化。CPU多来一个环，0，1，2，3的基础上又增加了环-1
