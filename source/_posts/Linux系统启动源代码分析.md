---
title: Linux 系统启动源代码分析
tags: Linux Source Code
categories: Linux
---

## Linux 系统启动源代码
以x86_64 arch机器为例：

—————————————————— 内核load进内存阶段 （arch/*/boot/）----------------------------------------------------

入口文件是：  __arch/x86/boot/compressed/head_64.S__  注意：  kernel 代码不是以 main 函数为入口的。
head_64.S 是 汇编 代码文件， 该文件会 call  verify_cpu + make_boot_params(初始化boot_params) + efi_main(处理并返回boot_params)，最终里面  call  extract_kernel 会调用 入口函数 extract_kernel, 该 函数 位于 arch/x86/boot/compressed/misc.c 会：
    1. 拿到 boot_params , 由汇编代码传入该参数      该 参数的结构体 位于 arch/x86/include/uapi/asm/bootparam.h .
    2. 通过 sanitize_boot_params 函数  初始化 boot_params 参数部分内容， 代码位于 arch/x86/include/asm/bootparam_utils.h .
    3. 通过 boot_params->screen_info 内容，  调用 arch/x86/boot/compressed/console.c  文件中的 console_init 函数, 初始化 tty 的console .
    4. (arch/x86/boot/compressed/kaslr.c) choose_random_location 函数 会随机挑选一段内存地址， 用于解压内核 压缩文件。
    5. 调用 (lib/decompress_bunzip2.c) __decompress 函数解压缩内核压缩文件, 根据不同的压缩文件类型，调用不同的解压缩函数， 压缩文件区分应该是发生在 编译内核时。
    6. 内核文件解压之后会成为 elf 文件， 位于内存中， 通过调用 parse_elf 函数 load 进 内核内容,  parse_elf 位于 arch/x86/boot/compressed/misc.c 中。
    7. 判断是否 需要重新 分配内存地址， 调用 handle_relocations 函数。

arch/x86/boot/compressed/head_64.S 在解压缩内核之后会执行解压缩之后的内核代码！！！

而 与arch 无关， 较为 common(legacy) 的 文件  arch/x86/boot/header.S  会 最终调用 (arch/x86/boot/main.c) main 函数 ， 该函数会：
    1. copy_boot_params   拷贝启动参数
    2. console_init  初始化console
    3. init_heap
    4. validate_cpu
    5. set_bios_mode
    6. detect_memory
    7. keyboard_init
    8. query_ist
    9. query_apm_bios (if  config_apm)
    10. query_edd  (if config_edd)
    11. set_video
    12. go_to_protected_mode



—————————————————— 内核启动阶段 （init/main.c） ----------------------------------------------------

入口文件是 ： init/main.c        启动函数是：  start_kernel   该函数会：
```bash
    1. 调用 (kernel/fork.c)  set_task_stack_end_magic(&init_task) 函数， 注册系统内核启动后的 idle(PID=0)  进程。 该 init_task 在 init_task.h 文件中定义，在 fork.c 文件中设置栈边界。
    2. smp_setup_processor_id 函数, 检查cpu是否为多处理器，获取当前处理器逻辑号。
    3. 调用(debugobjects.c)debug_objects_early_init 函数,  初始化debug对象的锁，并将debug对象链接成链表。
    4. 调用(stackprotector.h)boot_init_stack_canary函数, 尽可能早地进行stack protect，防止 栈越界 canary 攻击，关于canary attack 可以参照： https://hardenedlinux.github.io/2016/11/27/canary.html
    5.  调用(group.c)group_init_early 函数,  初始化 group_root 结构体， 并且将每一个 cgroup_subsys 加入到 group_root 对象中，并初始化每一个 cgroup_subsys 对象。 支持的 cgroup_subsys 位于 include/linux/cgroup_subsys.h 文件中。
    6. local_irq_disable() 函数 将本地中断暂时disabled。
    7. 调用(kernel/cpu.c)boot_cpu_init函数，  设置 获得 cpu 第一个处理器标志对象， 标志 该处理器对象为 online+active+present+possible.
    8. 调用 (mm/highmem.c)page_address_init 函数,  初始化 高端内存页表 page_address_htable 对象 .
    9. 调用 pr_notice 函数 打印 Linux 版本信息(linux_banner) .
    10. 调用 setup_arch(command_line) 函数， 设置与 硬件架构相关的 配置,  command_line 为内核启动参数。 相关结构体有：  hwrpb_struct， notifier_block(中断处理结构体,注册形成notifier_chain)，alpha_using_srm/alpha_using_qemu(使用 srm或者qemu), callback_init(初始化 kernel_page + kernel_PCB + third_level_PTE)  
    11. 调用 mm_types.h(mm_init_cpumask)函数， 初始化 init_mm 对象，init_mm 是 mm_struct 结构体对象，位于 init-mm.c 文件中.
    12. 调用 setup_command_line 函数，保存 原始的 command_line 对象， 保存给以后分析。    
    13. cpu 相关的4个函数(不太明白)： setup_nr_cpu_ids + setup_per_cpu_areas + boot_cpu_state_init + smp_prepare_boot_cpu
    14. 调用 (page_alloc.c)build_all_zonelists 函数 初始化 页表区， 建立系统内存页表链表 .
    15. 调用 (page_alloc.c)page_alloc_init 函数 初始化页表， 初始化内存页表。
    16. 调用 parse_early_param 函数, 解析 command_line 对象，拿到 早期 参数, 参数以 kernel_param 结构体保存 。
    17. 调用 pase_args 函数， 解析不能被 pase_early_param 函数解析的参数。
    18. 调用 (pid.c)pidhash_init 函数,  call  (page_alloc.c)alloc_large_system_hash 函数, allocate 一个大的系统 hash table，名字为 PID .
    19. 调用 (dcache.c)bfs_caches_init_early 函数, call 两次 alloc_large_system_hash 函数, allocate 两个大的系统 hash table，名字为 Dentry Cache + Inode-Cache .
    20. 调用 (eatable.c)sort_main_extable 函数, 对异常处理函数table进行排序。
    21. 调用 (arch/x86/kernel/traps.c)trap_init 函数,  初始化 硬件中断.
    22. 调用 (main.c)mm_init 函数, 建立内存分配器.   该函数会调用  page_ext_init_flatmem + mem_init + kmem_cache_init + percpu_init_late + pgtable_init + vmalloc_init + ioremap_huge_init  .
    23. 调用 (core.c)sched_init 函数， 初始化 调度器 .  该调度器会处理各种中断.  很复杂.
    24. preempt_disable 函数， 禁止调度抢占.
    25. 调用 idr_init_cache 函数， call  kmem_cache_create 函数 创建 kmem_cache , 该cache 名为 idr_layer_cache .
    26. 调用 (workqueue.c)workqueue_init_early 函数,  该函数 建立 各种数据结构／系统workqueue，  调用 多次 alloc_workqueque 函数 构建 各种事件队列.
    27. rcu_init 函数， 初始化互斥访问机制.
    28. (tiny.c/tree.c)trace_init 函数,   trace printk 调用.
    29. (context_tracking.c)context_tracking_init 函数,   tracking context 在哪个cpu上运行.
    30. (radix-tree.c)radix_tree_init函数,  call  kmem_cache_create 创建 kmem_cache， 该cache 名为 radix_tree_node .  radix tree 为 基数树.
    31. (irq.c)early_irq_init 函数, 调用 arch_early_irq_init 函数, 构建 irq_domain 结构体.
    32. (irqinit.c)init_IRQ 函数,  调用  arch/x86/kernel/irqinit.c init_IRQ 函数， 初始化中断向量. 关键结构体 有 x86_init_ops .
    33. (tick-common.c)tick_init 函数, 初始化 时钟控制.
    34. rcu_init_nohz 函数.
    35. (timer.c)init_timers 函数, 初始化 定时器. 原理是： 通过 调用 open_softirq 软中断， 注册中断处理函数为 run_timer_softirq.
    36. hrtimers_init 函数， 初始化 高精度定时器.
    37. (softer_init.c)softirq_init 函数，  软中断 初始化.  调用 open_softirq 注册2个级别 TASKLET_SOFTIRQ + HI_SOFTIRQ  中断 向量， 关键对象 softirq_vec .   可以学习:  soft_irq 与 tasklet 区别。
    38. timekeeping_init : 初始化资源和普通计时器,  初始化 clocksource 和 common timekeeping values.
    39. time_init :    call  (arch/x86/kernel/time.c) x86_late_time_init 函数, 注册 结构体 x86_init_ops 对象的  timer 属性.   以及 时钟中断, 在后面的 late_time_init 函数中调用.           
    40. sched_clock_postinit :  初始化 clock_data 结构体, 更新 调度 计时器.   
    41. printk_safe_init: 调用 init_irq_work 函数 初始化中断栈, 将 pending 的 所有 message 全都print 出去.
    42. (core.c)perf_event_init:  初始化 idr 结构体, call perf_pmu_function 注册 performance monitoring unit(pmu) 各类事件，   pmu 事件类型有：perf_swevent +   perf_tracepoint + perf_cpu_clock + perf_task_clock + perf_breakpoint .
    43. (profile.c)profile_init : kernel profiling 工具 初始化.  初始化 内核调优 的代码, 与内核启动的传入参数有关.  CPU_PROFILING + SCHED_PROFILING + SLEEP_PROFILING + KVM_PROFILING .
    44.  call_function_init: 不知道做啥.
    45.  (slab.c)kmem_cache_init_late : 调.整 cpu cache大小, 并且 注册 一段 memory 用于 hotplug 回调.
    46. (tty_io.h)console_init : 初始化 console  device. 可以显示 printk 的内容.
    47. (locked.c)lockdep_info :  打印 一些 依赖 信息.
    48. (locking-selftest.c)locking_selftest : self-test  for  hard/soft-irqs.
    49. (ifdef  CONFIG_BLK_DEV_INITRD)initrd_start :  kernel启动时是否传入 initrd 参数， 传入的话 会进入 raw disk.
    50. (page_ext.c)page_ext_init : 防止高位内存出界, 出界的内存可能没有被初始化，重新初始化, 并且 设置 回调 函数.
    51. (debugobjects.c) debug_objects_mem_init :  debug 内存 是否 越界.
    52. (kmemleak.c)kmemleak_init : 内核内存 分配 泄漏 检测 逻辑 初始化.
    53. (page_alloc.c)setup_per_cpu_pageset :  为每一个 cpu 分配内存 页表 及 页表区.  在此函数调用之前, 可被使用的内存 仅为 boot memory.     
    54. (mempolicy.c)numa_policy_init : 设置内存 numa(Non-uniform memory access, 非统一内存访问架构) 规则. 关键结构体: mempolicy
    55. (arch/x86/kernel/time.c)later_time_init : 执行 time_init 函数初始化的内容.
    56. calibrate_delay : 校准 时延.
    57. (pid.c)pidmap_init :  初始化  pid_max 值, 以及 初始化 pid_namespace 结构体.  保留 pid 为0 的位置.
    58. (rmap.c)anon_vma_init : (Anonymous Virtual Memory Access) 匿名虚拟内存区域初始化.  
    59. (drivers/acpi/bus.c)acpi_early_init:     ACPI ？？？  不知道 干什么?
    60. (efi.c)efi_enter_virtual_mode :  也不清楚 EFI 做啥？
    61.  (esprit_64.c)init_espfix_bsp : 调整 进程 esp 寄存器位置, 在 non-init 进程创建之前调用.
    62.  (fork.c)thread_stack_cache_init : 调用 kmem_cache_create 在内核内存区 创建 thread_stack cache.
    63. (cred.c)cred_init : 调用 kmem_cache_create 在内核内存区 创建 cred_jar  cache, 用于 存储 credentials.
    64. (fork.c)fork_init : 进程创建机制 初始化, 调用 kmem_cache_create 在内核内存区 创建 task_struct cache,  set_max_threads, 关键结构体 task_struct
    65. (fork.c)proc_caches_init : 进程创建所需的其它结构体 初始化, 调用 kmem_cache_create 在内核内存区 创建 sighand_cache/signal_cache/files_cache/fs_cache/mm_struct  cache .
    66. (fs/buffer.c)buffer_init : 调用 kmem_cache_create 在内核内存区 创建 buffer_head  cache. 将一定数量的内存区 设置为 buffer.
    67. (security/keys/key.c)key_init : 内核密钥 管理,  调用 kmem_cache_create 在内核内存区 创建 key_jar  cache .    
    68. (security.c)security_init : 内核安全框架. 比较复杂, 没看懂 ?
    69. (debug_core.c)dbg_late_init : kernel debug stuff.
    70. (fs/dcache.c)vfs_caches_init : 创建虚拟文件系统.  主要调用函数有:  dcache_init, inode_init, files_init, files_maxfiles_init, mnt_init(该函数还会调用 kernfs_init/sysfs_init/kobject_create_and_add/init_rootfs/init_mount_tree), bdev_cache_init, chrdev_init.  调用 kmem_cache_create 在内核内存区 创建 names_cache + dentry + inode_cache + filp + mnt_cache + bdev_cache   cache.  调用 alloc_large_system_hash   分配 Dentry cache + Inode-cache + Mount-cache + Mountpoint-cache     HashMap.
        (mount.c)kernfs_init : 该函数 在内核内存区 创建 kernfs_node_cache  cache.  
        (kobject.c)kobject_create_and_add : 创建 kobject  fs 对象.
        (mount.c)sysfs_init : 该函数 调用  register_filesystem 函数, 初始化 file_system_type 对象 sysfs .  register_filesystem 当加载fs对应module时调用该函数.   
        (do_mounts.c)init_rootfs : 该函数调用  register_filesystem 函数, 初始化 file_system_type 对象 rootfs + tmpfs/ramfs  
        (namespace.c)init_mount_tree :  设置 rootfs mount tree.
        (block_dev.c)bdev_cache_init : 注册 block 设备,  注册对象为 file_system_type  bdev .   bd_mount 扫描磁盘, 获得 磁盘 文件系统.      
        (char_dev.c)chrdev_init :  注册 字符设备.
    71. (mm/filemap.c)pagecache_init :  该函数 主要是初始化 页写回 机制, 主要函数为 (page_writeback.c)page_writeback_init .
    72. (kernel/signal.c)signals_init :  调用 kmem_cache_create 在内核内存区 创建  sigqueue  cache 对象.  
    73. (fs/proc/root.c)proc_root_init :  proc 文件系统初始化, 主要调用  proc_init_inodecache + set_proc_pid_nlink + register_filesystem + proc_self_init + proc_thread_self_init + proc_symlink + proc_net_init + proc_mkdir + proc_create_mount_point + proc_tty_init + proc_sys_init .        
            (fs/proc/inode.c)proc_init_inodecache :  调用 kmem_cache_create 在内核内存区 创建  proc_inode_cache 对象.
            (fs/proc/base.c)set_proc_pid_nlink :  主要 调用 pid_entry_nlink 函数,  关键结构体 为 pid_entry . 初始化 2个对象:  tid_base_stuff  + tgid_base_stuff .         
            (filesystem.c)register_filesystem : 注册 proc  filesystem .  关键 函数为  proc_mount, 该函数会注册 /proc/self 文件夹.  
            (fs/proc/thread_self.c)proc_self_init + proc_thread_self_init :  初始化 /proc/self 文件夹.  
            (fs/proc/thread_self.c)proc_symlink :  创建 链接 文件 /proc/mounts  链接  /proc/self/mounts .  
            (fs/proc/proc_net.c)proc_net_init :   创建  链接 文件 /proc/net  链接  /proc/self/net .  初始化  /proc/self/net  文件夹.
            proc_mkdir : 创建 sysvipc, fs, driver, bus  文件夹 .  
            proc_create_mount_point : 创建  /proc/fs/nfsd 文件夹 .
            proc_tty_init : 创建 /proc/tty 文件夹.
            (fs/proc/proc_sysctl.c)proc_sys_init :  注册和初始化  /proc/sys (sysctl)  文件系统.   关键函数 ：  __register_sysctl_paths
    74. nsfs_init : 不知道干啥 ???
    75. (kernel/cpuset.c)cpuset_init :  初始化 cpuset 系统, 并且 创建 /sys/fs/cgroup/cpuset 文件夹.  
    76. (kernel/cgroup.c)cgroup_init :  control group 初始化.  
    77. (kernel/taskstats.c)taskstats_init_early : 早期初始化，初始化 taskstats 结构体.
    78. delayacct_init() + check_bugs() + acpi_subsystem_init() + arch_post_acpi_subsys_init() + sfi_init_late()
    79. (main.c)rest_init :  完成 剩下 non-init 的 任务.    包括 :    —> https://danielmaker.github.io/blog/linux/images/start_kernel_call_graph.svg
                rcu_scheduler_starting :     启动 rcu_scheduler
                kernel_thread(kernel_init) :  do_fork 启动一个 进程 执行 kernel_init 函数.  PID 为 1 的进程.   —>  kernel_init_freeable函数   +   run_init_process
                numa_default_policy : 内存分配  numa 策略.
                kernel_thread(kthreadd) :   执行 kthreadd 函数.  PID 为 2 的 进程.  为spawn所有其它的 thread 进程 .
                find_task_by_pid_ns
                init_idle_bootup_task
                schedule_preempt_disabled
                cpu_startup_entry            ——>   do_idle  cpu 调度        idle 为 PID 为 0 的进程.
```
rest_init函数所做内容示意图:
![](/images/start_kernel_call_graph.svg)

内核代码中 调用 alloc_large_system_hash 函数的位置有不少，会构建一些 大的系统 hash table， 名称分别有：
1. Dentry cache
2. Inode-cache
3. Mount-cache
4. Mountpoint-cache
5. PV qspinlock
6. futex
7. PID
8. TCP established
9. TCP bind
10. UDP  or  UDP-Lite

—————————————中断 宏： SAVE_ALL & RESTORE_ALL    用户态和内核态 上下文切换 ------------------------------------
中断向量表：    __linux/arch/x86/entry/systemcalls/syscall_32.tbl__
中断结构体：    __irq_desc__

linux/arch/x86/entry/entry_32.S  : 系统调用的代码段,   入口应该是：
http://lxr.linux.no/linux+v3.19/arch/x86/kernel/entry_32.S      —>      ENTRY(system_call)

Linux 进程 fork 关键代码：
http://lxr.linux.no/linux+v3.19/kernel/fork.c#L1185
