---
title: Linux Networking Internals 3 - Network Device Initialize
tags: Linux Networking Internals
categories: Linux
---

## 网络设备的初始化 ：
    包括2个阶段：
1.     作为常规device，初始化
2.     作为network device，初始化

```bash
Kernel boot up  —>  start_kernel (initializes bunch of subsystems)
                                        —>  invokes  init kernel thread   (takes care of the rest of  initializations)
                                                —>  do_basic_setup()        —>  deriver_init/sock_init/ ….
                                                —>  free_init_mem()
                                                —>  run_init_process()

Three mainly interested points:
1. Boot-time options               parse_args   handle configuration parameters that a boot loader(LILO/GRUB) has passed to kernel at boot time
2. Interrupts and timers        init_IRQ(hardware interrupts)/softirq_init(software interrupts)     
3. Initialization routines        do_initcalls will initialize kernel subsystems and built-in device drivers        

run_init_process determines the first process(PID 1) run on the system, through the init=  boot time option
```
![](/images/Linux_network_internal_system_start.png)

### Initialization  Options:
Both components built into the kernel and components loaded as modules can be passed input parameters so that users can fine-tune the functionality implemented by the components, override defaults compiled into them, or change them from one system boot to the next, kernel provides two kinds of macros to define options:
1. Module options                module_param  family
2. Boot-time kernel options        _ _setup  family
drivers/block/loop.c

Module options.                      <include/linux/moduleparam.h>
__module_param__
The module parameters are listed in the module’s directory “/sys/modules”. The subdirectory /sys/modules/module/parameters holds a file for each parameter exported by module.
```bash
[root@localhost src]# ls -la /sys/module/sis900/parameters/
total 0
drwxr-xr-x  2 root root    0 Apr  9 18:31 .
drwxr-xr-x  4 root root    0 Apr  9 18:31 ..
-r--r--r--  1 root root    0 Apr  9 18:31 debug
-r--r--r--  1 root root 4096 Apr  9 18:31 max_interrupt_work
-r--r--r--  1 root root 4096 Apr  9 18:31 multicast_filter_limit
[root@localhost src]#
```

### Initializing the Device Handling Layer: net_dev_init
```bash
net_dev_init defined in net/core/dev.c
static int _ _init net_dev_init(void)                    _ _init  macro
{
    ...
}
```
subsys_initcall(net_dev_init);
subsys_initcall   macros ensure net_dev_init runs before any NIC device drivers register themselves.
main parts of  net_dev_init:
```bash
1. the per-CPU data structures used by two networking software interrupts(softirqs) are initialized.
2. when kernel is compiled with support for /proc filesystem, a few files are added to /proc with dev_proc_init and dev_mcast_init.
3. netdev_sysfs_init registers the net class with sysfs. This creates directory /sys/class/net,  under this you can find a subdirectory for each registered network device.
4. net_random_init initializes a per-CPU vector of seeds that will be used when generating random numbers with net_random routine.
5. The protocol-independent destination cache(DST) is initialized with dst_init.
6. The protocol handler vector ptype_base, used to demultiplex ingress traffic, is initialized.
7. When OFFLINE_SAMPLE symbol is defined, the kernel sets up a function to run at regular intervals to collect statistics about the devices’ queue lengths.
8. A callback handler is registered with the notification chain that issues notifications about CPU hotplug events. Callback used is dev_cpu_callback.             
```

__/proc/net__  is created by net_dev_init, via __dev_proc_init__ and __dev_mcast_init__:
```bash
1. dev                   for each network device registered with kernel, a few statistics about reception and transmission.
2. dev_mcast     for each network device registered with kernel, the values of a few parameters used by IP multicast.
3. wireless          for each wireless device, prints the value of a few parameters from the wireless block.
4. softnet_stat       exports statistics about the software interrupts used by networking code.
```

### User-Space 工具
- /sbin/modprobe
        Invoke when the kernel needs to load a module.

- /sbin/hotplug
        Invoke when the kernel detects that a new device has been plugged or unplugged from system.

The kernel provides a function named call_usermodehelper to execute such user-space helper.
Two kernel routines, request_module and kobject_hotplug , invoke call_usermodehelper to invoke /sbin/modprobe and /sbin/hotplug    
![](/images/Linux_network_internal_event_propagation_from_user_space.png)

__kmod__ is the kernel module loader that allows kernel components to request the loading of module. call  request_module   

Hotplug was introduced into the kernel to implement popular consuer feature known as Plug and Play (PnP). When compile the kernel modules, the object files are placed by default in directory: /lib/modules/kernel_version/.  kobject_hotplug function is invoked by the kernel to respond to insertion and removal of a device.   

__modprobe__ and __hotplug__ create file/directory in /proc/sys/kernel  

### When a Device is Registered:
The  registration of a network device takes place in the following situations:
1. Loading an NIC’s device driver
2. Inserting a hot-pluggable network device

### When a Device is Unregistered:
Two main conditions trigger the unregisteration of a device:
1. Unloading an NIC device driver
2. Removing a hot-pluggable network device

### Allocating net_device Structures:
Network devices are defined with net_device structures.             <net/core/dev.c>
include 3 input parameters:
1. Size of private data structure
2. Device name
3. Setup routine

![](/images/Linux_network_internal_netdevice_register.png)

### Device Initialization:
net_device structure is pretty bug, its fields are initialized in chunks by different routines.
1. Device drivers :   Parameters such as IRQ, I/O memory, and I/O port, those values depend on hardware configuration, are taken care of by device driver.      xxx_probe
2. Device type :     the type family is taken care by xxx_setup routines.
3. Features:           Mandatory and optional features also need to be initialized.

![](/images/Linux_network_internal_netdevice_initialization.png)

#### Device Type Initialization: xxx_setup Functions
alloc_ xxxdev  function pass right xxx_setup routine to alloc_netdev .
void ether_setup(struct net_device *dev)
{
    dev->change_mtu           = eth_change_mtu;
    dev->hard_header          = eth_header;
    dev->rebuild_header       = eth_rebuild_header;
    dev->set_mac_address      = eth_mac_addr;
    dev->hard_header_cache    = eth_header_cache;
    dev->header_cache_update  = eth_header_cache_update;
    dev->hard_header_parse    = eth_header_parse;

    dev->type                 = ARPHRD_ETHER;
    dev->hard_header_len      = ETH_HLEN;
    dev->mtu                  = 1500;
    dev->addr_len             = ETH_ALEN;
    dev->tx_queue_len         = 1000;
    dev->flags                = IFF_BROADCAST|IFF_MULTICAST;

    memset(dev->broadcast,0xFF, ETH_ALEN);
}

#### Organization of net_device Structures:
![](/images/Linux_network_internal_netdevice_registerd_device_list.png)

#### Device State:
__net_device__ include:
1. flags                    bitmap used to store different flags.
2. reg_state  device registration state
3. state      device state with regard to its queuing discipline.

#### Net Device Queuing Discipline State:
Each network device is assigned a queuing discipline, used by Traffic Control to implement its QoS mechanisms.      <include/linux/netdevice.h>

LINK_STATE_START
LINK_STATE_PRESENT
LINK_STATE_NOCARRIER
LINK_STATE_LINKWATCH_EVENT
LINK_STATE_XOFF
LINK_STATE_SHED
LINK_STATE_RX_SCHED

#### Net Device Registration State:
The state of a device with regard to its registration with the network stack is saved in reg_state field of the net_device structure.      <include/linux/netdevice.h>
NETREG_UNINITIALIZED
NETREG_REGISTERING
NETREG_REGISTERED
NETREG_UNREGISTERING
NETREG_UNREGISTERED
NETREG_RELEASED

![](/images/Linux_network_internal_netdevice_registration_state.png)

#### Net Device register + unregister
![](/images/Linux_network_internal_netdevice_register_unregister.png)

#### Device Registration Status Notification:
Both kernel components and user-space applications are interested in knowing when a network device is registered, unregistered, goes down, or comes up.

netdev_chain: kernel components can register with this notification chain.    <net/core/dev.c>
All the NETDEV_XXX events that are reported via neTDev_chain are listed in <include/linux/notifier.h>.   
Notification event list:
NETDEV_UP
NETDEV_GOING_DOWN
NETDEV_DOWN
NETDEV_REGISTER
NETDEV_UNREGISTER
NETDEV_REBOOT
NETDEV_CHANGEADDR
NETDEV_CHANGENAME
NETDEV_CHANGE

Quite a few kernel components register to __netdev_chain__  List:
1. Routing
2. Firewall
3. Protocol code
4. Virtual device
5. RTnetlink

#### Enabling and Disabling a Network Device:
![](/images/Linux_network_internal_netdevice_enable_disabler.png)

#### Virtual Devices:
The virtual devices need to be registered and enabled just like real ones, to be used.
register_netdevice/unregister_netdevice


#### Device initialization phases:
1. Hardware initialization      this is done by the device driver in cooperation with the generic bus layer(PCI or USB).      
2. Software initialization        before the device can be used, it may need to provide some configuration parameters     
3. Feature initialization          configure some options     

net_device data structure include a set of function pointers, that kernel uses to interact with the device driver and special kernel features

#### Basic Goals of NIC Initialization:
establish  device/kernel  communication. such as:
1. IRQ line             the /proc/interrupts file can be used to view the status of the current assignments.
2. I/O ports and memory registration         map an area of device’s memory into system memory.     I/O ports and memory are registered and released with request_region/release_region

#### Interaction Between Devices and Kernel:
nearly all devices interact with kernel in two ways:
1. Polling                driven on the kernel side, the kernel check device status at regular intervals.
2. Interrupt            driven on the device side, the device sends a hardware signal to kernel.

#### Hardware Interrupts:
every interrupt runs a  function called an interrupt handler. IRQ are defined in kernel/irq/manage.c and are overridden by arch/XXX/kernel/irq.c.
```bash
int request_irq(unsigned int irq, void (*handler)(int, void*, struct pt_regs*), unsigned long irqflags, const char * devname, void *dev_id)
void free_irq(unsigned_int irq, void *dev_id)
```

#### NIC  Inteerrupt Types:
1. Reception of a frame
2. Transmission failure
3. DMA transfer has completed successfully
4. Device has enough memory to handle a new transmission
```bash
drivers/net/3c509.c:
static int
el3_start_xmit(struct sk_buff *skb, struct net_device *dev)
{
    ... ... ...
    netif_stop_queue (dev);
    ... ... ...
    if (inw(ioaddr + TX_FREE) > 1536)
        netif_start_queue(dev);
    else
        outw(SetTxThreshold + 1536, ioaddr + EL3_CMD);
    ... ... ...
}
```

### Organization of IRQs to handler mappings:
```bash
kernel/irq/handler.c
irqaction data structure:
void (*handler)(int irq, void *dev_id, struct pt_regs *regs)
int irq
void *dev_id
...
```

![](/images/Linux_network_internal_netdevice_IRQ_handler.png)
