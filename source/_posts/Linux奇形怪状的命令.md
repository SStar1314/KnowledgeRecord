---
title: Linux一些命令总结
tags: Linux
categories: Linux
---

### 通过Sighup信号让Linux Daemon重新加载配置文件
对于守护进程而言，一般情况下守护进程都是detach from shell，所以接受不到sighup信号，这个信号经常被用作 reload configuration左右。
```bash
kill  -s   SIGHUP  $pid  
```
### Linux syslog
Linux 系统中的主要log都在/var/log目录下：
- /var/log/lastlog:  用户登录系统记录日志。
- /var/log/secure:  与Linux中身份验证相关的日志文件，比如：ssh。
- /var/log/messages:  是Linux中许多应用的首选日志文件。
- /var/log/maillog:  邮件相关。
- /var/log/cron:  计划任务日志， /etc/cron.d/sysstat。

```bash
klogd  —dmesg               —> /var/log/dmesg
syslogd                     —> /ect/syslog.conf
两个共同的配置文件 —> /etc/sysconfig/syslog
```
redhat 中使用了   __rsyslog__ 替代 syslog
```bash
/etc/rsyslog.d/*
```

### Linux tty/ttyS/pts + X-Window
- tty：是串口的终端模拟器，模拟鼠标／键盘／显示器等设备。
- ttyS：Linux下的真正的串口。
- pts：虚拟的设备终端，由application（如：X-window，SSH）动态创建。 (X-window中的终端模拟器Teminal 也会创建相应的pts/#)
- X-window： 是Linux的图形界面application。

w: 显示当前所有的登录用户

多登陆端口通信：  （pts/1向pts/0发送消息）
```bash
__pts/1:__  echo  aaaaaaaaa  >  /dev/pts/0
__pts/0:__  aaaaaaaaa
```

- Alt＋F7: 打开第一个X-window窗口，Alt+F8 打开第二个
- Linux启动的时候，会首先打开6个tty串口模拟器(Alt + F1-6) +++++ 1个X-window application(Alt + F7)。
- startx: 启动X-window命令。   startx --:1 (打开第二个X-window)     startx --:2 (打开第三个X-window)         这个命令在pts虚拟终端中无法运行。
- skill -9 pts/2  让另一个控制台的人掉线。

vim /etc/inittab     可以减少默认启动的tty个数！！！！

### Linux 打包及压缩工具
__compress/uncompress__     最古老的Unix压缩工具
__gzip/gunzip__          最广泛的压缩工具，Linux系统中标准压缩工具，对于文本文件能够达到很高压缩率
__bzip2/bunzip2__     新版Linux压缩工具，比gzip拥有更高的压缩率
__tar__ 打包(备份)作用，参数：
  -   -c:  将文件备份create
  -   -v:  将过程输出verbose
  -   -x:  从一个文件中解出备份extract
  -   -r:  将文件添加入已经存在的文件中
  -   -C: 解出备份指向位置
  -   -z:  gzip压缩
  -   -j:  bzip2压缩
```bash     
tar xvf/xvf/rvf
```

### Linux 文件类型(7种) + 查询inode的命令
Linux 下文件类型共有7种：
"-"     文件
d     文件夹
l     链接
b     block
c     char字符设备
s     socket文件
p     protocol网络文件

文件分为三个部分进行存储：(目录文件中存储文件名)
1.存储文件名(指向inode号)   —>   2.inode(指向块存储单元)   —>  3.block (4k为单位)

inode 存储文件的属性，  可以用 __stat__ 命令查看文件inode的内容。

### Linux 文件命令
文本文件的操作命令：
- cat： 查看文件内容
- more： 逐屏查看文件内容
- less： 逐行查看文件内容
- head：显示文件开头部分内容
- tail： 显示文件结尾部分内容
- diff： 报告文件差异
- uniq： 去除文件中的相邻的重复行
- cut： 只显示文件中的某一行                         cut -d: -f1  /etc/passwd
- sort：按序重排文本                                sort  -t:  +2  -n  /etc/passwd
- wc： 统计文件的行，词，字数

- which  ls     :   查找命令,  查找 $PATH 路径中的文件
- whereis  ls  :   可以查找命令及相应的man文件, 查找 $PATH 和 $MANPATH 路径
- locate  ls :      查找所有匹配ls字母的文件，  注意!!!  locate 命令从数据库中查找文件，而不是直接查找系统文件，数据库位置为/var/lib/slocate/slocate.db.        当数据库没有更新的时候，可能会查找不到。
```bash
yum install mlocate     !!!                   
updatedb 更新locate 的数据库                   
/etc/cron.daily/mlocate 每天有定时更新数据库的任务
```

- find

```bash
find  .  -iname  “xxx”  -ok   file  {}  \;        交互执行file命令
find  .  -iname  “xxx”  -exec   file  {}  \;      直接执行file命令
```
- grep

### Linux 系统服务启动(readhat6.x的init程序, readhat7之后的systemd之后的并非如此)
系统服务启动顺序：
|-/etc/inittab                    总的配置文件
|-----/etc/rc.d/rc.sysinit                    系统初始化文件, 加载许多内容
------------fsck.ext3   mount  -o  rw.remount   /dev/sda2  /
------------mount  -a  /etc/fstab
|-----/etc/rc.d/rcX.d/*                    X为系统运行级别
|----------/etc/rc.d/rcX.d/ServiceXXX1  start               用户定义自启动的service
|----------/etc/rc.d/rcX.d/ServiceXXX2  start
|-----/etc/rc.d/rc.local          最后访问的文件
|
|-minigetty   启动   /dev/tty1-6                                   启动tty
|----------login    —>  bash     —>  /etc/profile     ~/.bash_profile
|-gdm               监控进程是否死掉，重启进程

- respawn          监控tty进程是否死掉，如果死掉会重启tty进程    respawn与init进程共同死，共同活
- chkconfig  service  on/off --level X    将service 启动/关闭 脚本 加入/删除 到/etc/rc.d/rcX.d 目录下
- 加入到rcX.d 目录之后，可以用        service  Sname   start/stop/restart      来控制service

### Linux 一些命令
shutdown -h now／init 0 ／ halt  -p -f ／ poweroff： 关机
users： 显示当前系统登录的用户
who： 当前登录在本机的用户及来源
w： 当前登录本机的用户及运行的程序
write： 给当前联机的用户发消息
wall： 给所有登录在本机的用户广播消息
last： 查看用户的登录日志
lastlog： 查看每个用户最后登录的情况
finger： 查看用户信息

### Linux 网络
__ping -s 1024  www.baidu.com__          -s 可以指定测试包的大小，用于测试不同包大小的带宽。
__ab -n 1000 -c 1000  www.baidu.com__         ab为 linux压力测试的命令，模拟1000个端口，进行总共1000次请求          (Apache HTTP server benchmarking tool)
__traceroute  www.baidu.com__                    查询整个转发路径上的访问结点的掉包率
__mtr  www.baidu.com__                    查看通路的掉包率
__arping__  查询哪个机器网卡ip地址是多少

__top__          
__vmstat__          (Report virtual memory statistics)
__netstat__          (Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships)
__netstat__ 查看tcp连接时, 如果
ESTABLISHED特别多              __CC(Challenge Collapsar)__ 攻击, 建立链接攻击
ESTABLISHED很少,LISTEN很多     __DDOS__ 攻击  sync-flood(泛滥攻击)

抓包工具：
- iptraf
- tcpdump
- wireshark

### Linux 网络内核参数修改
Linux 内核内核参数：
位于 /proc/sys 目录下, 修改内核参数：
```bash
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
sysctl  -w  net.ipv4.icmp_echo_ignore_all=1
sysctl  -p  $file     load config from file
```
输出内核参数：
```bash
sysctl  -a  > /tmp/sysctl.output
```
从文件中读入内核参数：
```bash
sysctl  -f   /tmp/sysctl.output  -p
```

### rpm包管理
- 解压rpm包
```bash
rpm2cpio *.rpm | cpio -div
```
- 查询rpm包里面内容
```bash
[root@vmenable core]# rpm -qpl *.rpm
/etc/*-release
/opt/*/swidtag
/opt/*/swidtag/*.swidtag
```
- 查询rpm包里面的install/uninstall script
```bash
[root@vmenable core]# rpm -qpl   *.x86_64.rpm  --scripts
preinstall scriptlet (using /bin/sh):
postinstall scriptlet (using /bin/sh):
preuninstall program: /bin/sh
postuninstall scriptlet (using /bin/sh):
```
