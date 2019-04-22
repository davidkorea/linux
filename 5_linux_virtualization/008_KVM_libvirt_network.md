# KVM_libvirt_network
# 1. Virtual Network

- **TUN**, which stands for "tunnel", simulates a network layer device and it operates at OSI reference model's layer 3 packets, such as IP packets. 
- **TAP**(namely a network tap) simulates a link layer device and it operates at OSI reference model's layer 2 packets, such as Ethernet frames. 
- TUN is used with routing, while TAP is used to create a network bridge.
