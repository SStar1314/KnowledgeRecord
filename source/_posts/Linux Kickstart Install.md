---
title: Linux Kickstart Install(无人值守安装)
tags: Linux
categories: Linux
---

### Linux Install
- Linux dd：  driver disk 安装。
- Linux ks：  kickstart 安装，无人值守安装。

boot:  linux  ks=nfs:192.168.0.254:/var/ftp/pub/ks.cfg

__anaconda__ 是python写的系统安装工具。

Linux 中有 4个主分区 ＋ 逻辑分区(需要将其中一个主分区变为扩展分区)。

Installation过程中可以切换窗口，Ctrl ＋ Alt ＋ F1,2,3,4,5,6,每个窗口分别有不同的作用。

### Linux Kickstart Install
__anaconda-ks.cfg__       kickstart安装配置文件
python —> anaconda —> install Linux
```bash
yum install pykickstart
yum install system-config-kickstart          无人值守需要先安装rpm包
```
__/tftpboot/n1.cfg配置文件：__
```bash
delay=0
image=/tftpboot/osimage/rhels7.1-x86_64-install-gss1/vmlinuz
   label="Kickstart"
   initrd=/tftpboot/osimage/rhels7.1-x86_64-install-gss1/initrd.img
   append="quiet inst.repo=http://%N:80/install/rhels7.1/x86_64 inst.ks=http://%N:80/install/autoinst/cn1 ip=eth0:dhcp  net.ifnames=0 net.ifnames=0  BOOTIF=%B"
```
