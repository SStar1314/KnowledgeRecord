---
title: Linux Networking Internals 5 - Network Device Notification Chains
tags: Linux Networking Internals
categories: Linux
---

## Network Device Notification Chains
publish  -vs-   subscribe

notifier_block 结构体
```bash
struct notifier_block
{
    int (*notifier_call)(struct notifier_block *self, unsigned long, void *);
    struct notifier_block *next;
    int priority;
};
```

注册函数：  notifier_chain_register

网络相关的 chain： inetaddr_chain , inet6addr_chain , and netdev_chain.

![](/images/Linux_network_internal_netdevice_notification_chain.png)

### Notifying Events on a Chain:
函数 notifier_call_chain    <kernel/sys.c>
```bash
int notifier_call_chain(struct notifier_block **n, unsigned long val, void *v)
{
    int ret = NOTIFY_DONE;
    struct notifier_block *nb = *n;

    while (nb)
    {
        ret = nb->notifier_call(nb, val, v);
        if (ret & NOTIFY_STOP_MASK)
        {
            return ret;
        }
        nb = nb->next;
    }
    return ret;
}
```
该函数返回值 类型 位于  include/linux/notifier.h 文件中。

### Notification  Chains  for the Networking Subsystems:
inetaddr_chain        netdev_chain

inetaddr_chain: sends notifications about insertion/removal/change of an IPv4 address on a local interface.
netdev_chain: sends notifications about the registration status of network devices.

```bash
int register_netdevice_notifier(struct notifier_block *nb)
{
        return notifier_chain_register(&netdev_chain, nb);
}
```
Common names for wrappers include [un]register_xxx_notifier, xxx_[un]register_notifier, and xxx_[un]register.
