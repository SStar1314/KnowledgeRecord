---
title: Linux Networking Internals 4 - Network Device Statistics
tags: Linux Networking Internals
categories: Linux
---

## Network Device Statistics
netdev_rx_stat, its elements are of type netif_rx_stats        <include/linux/netdevice.h>

```bash
struct netif_rx_stats netdev_rx_stat[NR_CPUS];

struct netif_rx_stats
{
        unsigned total;
        unsigned dropped;
        unsigned time_squeeze;
        unsigned throttled;
        unsigned fastroute_hit;
        unsigned fastroute_success;
        unsigned fastroute_defer;
        unsigned fastroute_deferred_out;
        unsigned fastroute_latency_reduction;
        unsigned cpu_collision;
} __ _  _cacheline_aligned;
```

![](/images/Linux_network_internal_netdevice_Statistics.png)
