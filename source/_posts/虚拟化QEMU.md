---
title: 虚拟化QEMU
tags: QEMU
categories: 虚拟化
---

### QEMU
神人 Fabrice Bellard最开始创立： https://bellard.org/
refer :  https://wiki.archlinux.org/index.php/QEMU_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

### 常用命令
创建硬盘镜像
```bash
$ qemu-img create -f raw image_file 4G
```
执行 qemu-img 带 resize 选项调整硬盘驱动镜像的大小.它适用于 raw 和 qcow2. 例如, 增加镜像 10 GB 大小, 运行:
```bash
$ qemu-img resize disk_image +10G
```
qemu-system-* 程序 (例如 qemu-system-i386 或 qemu-system-x86_64, 取决于客户机架构)用来运行虚拟化的客户机. 用法是:
```bash
$ qemu-system-i386 options disk_image
```
