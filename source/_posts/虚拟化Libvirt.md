---
title: 虚拟化Libvirt
tags: Libvirt
categories: 虚拟化
---

### Libvirt
refer :  https://wiki.archlinux.org/index.php/libvirt_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
refer :  https://www.ibm.com/developerworks/cn/linux/l-libvirt/index.html

### Libvirt架构
![](/images/libvirt_1.png)

### Libvirtd
![](/images/libvirt_2.png)

### Libvirt Driver
![](/images/libvirt_3.png)

### libvirt 支持的Driver程序

- Xen	面向 IA-32，IA-64 和 PowerPC 970 架构的虚拟机监控程序
- QEMU	面向各种架构的平台仿真器
- Kernel-based Virtual Machine (KVM)	Linux 平台仿真器
- Linux Containers（LXC）	用于操作系统虚拟化的 Linux（轻量级）容器
- OpenVZ	基于 Linux 内核的操作系统级虚拟化
- VirtualBox	x86 虚拟化虚拟机监控程序
- User Mode Linux	面向各种架构的 Linux 平台仿真器
- Storage	存储池驱动器（本地磁盘，网络磁盘，iSCSI 卷）
