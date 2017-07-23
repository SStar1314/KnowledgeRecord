---
title: 容器技术 lxc
tags: lxc
categories: 容器技术
---

### LXC
LXC is a userspace interface for the Linux kernel containment features.Through a powerful API and simple tools, it lets Linux users easily create and manage system or application containers.

### online try
online try: https://linuxcontainers.org/lxd/try-it/

```bash
root@tryit-vast:/proc# lxc launch ubuntu:16.04 first  

root@tryit-vast:/proc# lxc launch ubuntu:16.04 first                                                          
Creating first   
Retrieving image: rootfs: 100% (139.92MB/s)        
Starting first            
root@tryit-vast:/proc#
```
```bash
root@tryit-vast:/proc# lxc list                                                         
+-------+---------+---------------------+----------------------------------------------+------------+-----------+               
| NAME  |  STATE  |        IPV4         |                     IPV6                     |    TYPE    | SNAPSHOTS |                                                           
+-------+---------+---------------------+----------------------------------------------+------------+-----------+               
| first | RUNNING | 10.59.94.107 (eth0) | 2001:470:b368:1070:216:3eff:fee3:b5ae (eth0) | PERSISTENT | 0         |                                                           
+-------+---------+---------------------+----------------------------------------------+------------+-----------+   
```
```bash
root@tryit-vast:/proc# lxc  info  first

root@tryit-vast:/proc# lxc config show first

root@tryit-vast:/proc# lxc remote list           
+-----------------+------------------------------------------+---------------+--------+--------+                                
|      NAME       |                   URL                    |   PROTOCOL    | PUBLIC | STATIC |
+-----------------+------------------------------------------+---------------+--------+--------+                                
| images          | https://images.linuxcontainers.org       | simplestreams | YES    | NO     |
+-----------------+------------------------------------------+---------------+--------+--------+                                
| local (default) | unix://                                  | lxd           | NO     | YES    |
+-----------------+------------------------------------------+---------------+--------+--------+                                
| ubuntu          | https://cloud-images.ubuntu.com/releases | simplestreams | YES    | YES    |
+-----------------+------------------------------------------+---------------+--------+--------+                                
| ubuntu-daily    | https://cloud-images.ubuntu.com/daily    | simplestreams | YES    | YES    |
+-----------------+------------------------------------------+---------------+--------+--------+       
```
