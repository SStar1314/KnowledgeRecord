---
title: Linux Networking Internals 2 - 网络相关的 procfs vs sysctl vs sysfs
tags: Linux Networking Internals
categories: Linux
---

## 网络相关的 procfs vs sysctl vs sysfs
__/proc__ 目录 被 proc_mkdir 函数创建。
__/proc/net__  目录 被 __proc_net_fops_create/proc_net_remove__ (调用 create_proc_entry/remove_proc_entry) 创建及删除。  <include/linux/proc_fs.h>
__/proc/sys__ 实际上是内核变量，可以read／write。 /proc/sys 被定义在 __ctl_table__ 结构体中， 通过 __register_sysctl_table__ 和 __unregister_sysctl_table__ 来注册和取消注册。   <kernel/sysctl.c>
某些目录在系统启动的时候就被创建，某些在运行时刻才被添加。

### /proc/net/sys 创建流程图
![](/images/Linux_network_internal_proc_sys_net_creation.png)


### Dispatching ioctl commands
__ioctl__ 是主要管理net_device的调用函数.
![](/images/Linux_network_internal_ioctl_dispatch.png)
