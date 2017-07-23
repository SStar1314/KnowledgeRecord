---
title: Linux Networking Internals 1 - Critical Data Structures
tags: Linux Networking Internals
categories: Linux
---

## 本系列是阅读《Linux Networking Internals》书籍的总结
《Linux Networking Internals》 是基于Linux解释network的书籍, 文章内容还是不错的.
另推荐一本书《Understanding the Linux Kernel》, 阅读本系列之前更加推荐之前的文章, 关于该文章的记录也会放在博客上.

### Critical Data Structures
*  struct __sk_buff__
存放packet的结构体
*  struct __net_device__
linux kernel中指代网络设备的结构体
*  struct __sock__
存放socket信息的结构体

#### sk_buff
Sock Buffer: __sk_buff__    <include/linux/skbuff.h>
包括以下内容：
1. Layout
2. General
3. Feature-specific
4. Management functions:  skb_put, skb_push, skb_pull, skb_reserve


alloc_skb -> dev_alloc_skb
![](/images/Linux_network_internal_skbuff.png)

kfree_skb -> dev_kfree_skb
![](/images/Linux_netwirk_internal_skbuff_kfree.png)

skb_reserve
![](/images/Linux_network_internal_skbuff_reserve.png)

#### net_device
__net_device__ structure include:
1. Configuration
2. Statistics
3. Device status
4. List management
5. Traffic management
6. Feature specific
7. Generic
8. Function pointers (or VFT)

tips: __ifconfig/route__ 通过 __ioctl__ 系统调用来实现； __ip/IPROUTE2__ 使用Netlink socket 来实现.
