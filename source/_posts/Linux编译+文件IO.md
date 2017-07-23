---
title: Linux编译知识 + 文件操作
tags: Linux
categories: Linux
---

### 编译相关命令
```bash
1. gcc -g -Wall hello_world.c -o hello_world          生成可执行程序
2. gcc -E hello_world.c > hello_world.i               预处理过程的输出文件, gcc -E 预处理结束之后，编译会自动停止
3. gcc -S hello_world.c -o hello_world.s               对源代码进行语法分析，产生汇编文件, gcc -S 汇编结束之后，编译会自动停止
4. gcc -g -Wall -v  hello_world.c -o hello_world    -v查看中间结果输出
5. readelf -a hello_world                     Linux下可执行程序格式一般为ELF格式， readelf可以读取ELF格式的文件
6. strace ./hello_world                    strace可以跟踪系统调用，从而可以了解应用程序加载／运行／退出 的过程。
```
用户空间的程序默认是通过栈来传递参数的，但对于系统调用来说，内核态和用户态使用的是不同的栈，这使得系统调用的参数只能通过寄存器的方式进行传递。
```bash
1. objdump -S thread_safe        可以对运行代码进行反汇编
```
- 阻塞的系统调用：当进行系统调用时，除非出错或被信号打断，进程将会一直陷入内核态直到调用完成。
- 非阻塞的系统调用：是指无论I/O操作成功与否，调用都会立刻返回。

### 文件I/O
1. 内核中 进程对应的数据结构是task_struct， 位于内核代码的 include/linux/sched.h 文件中。
2. 内核中 文件表对应的数据结构是files_struct，位于内核代码的 include/linux/fdtable.h 文件中。
3. 数据结构 file，位于内核代码 include/linux/fs.h 文件中。
4. Linux中的第一个进程init，位于内核代码的 include/linux/init_task.h 以及 fs/file.c 文件中。
5. 所有进程都是由init进程fork出来的，代码位于 kernel/fork.c 文件中，该文件会 调用 include/linux/fdtable.h 以及 fs/file.c 文件中的 dup_fd 函数。
6. glibc中的open函数，调用内核中的 fs/open.c 文件的do_sys_open 函数，do_sys_open函数会使用get_unused_fd_flags函数，get_unused_fd_flags函数在include/linux/file.h是宏定义，实际调用 fs/file.c 中的alloc_fd函数。
7. 内核会通过fs/open.c文件中的fd_install函数将文件管理结构file与fd结合起来，当用户使用fd与内核交互时，内核可以通过fd得到内部管理文件的结构struct file。
8. 内核在文件close的时候，会关闭该fd，如果当前next_fd小于该fd，则将next_fd的直设置为该fd值。这是内核的文件描述符使用策略，尽可能的使用刚释放的较小的fd。
9. 内核文件include/linux/fs.h中，file_operations 结构体定义了文件操作函数, inode_operations 结构体定义了linux存储inode操作函数。
10. 文件打开但忘记close的时候，可能会出现两种情况：a. 文件描述符始终没有被释放；b. 用于文件管理的某些内存结构没有被释放。
    a. 内核会扩展文件表，当文件表达到上限时，会报EMFILE错误。
    b. 未超过上限时，可以申请空的内存；达到上限时，会报VFS: file-max limit reached的错误。

### ELF文件
ELF = Executable and Linkable Format
```bash
readelf  命令 可以读 Linux 可执行文件.
```

### glibc调用 system_call
一般Linux程序编写都会使用到glibc库, 而最终glibc也只是帮助用户进行系统调用而已, 至于glibc如何进行系统调用的？
通过阅读源码发现, 所有的函数都会最终调用glibc的源码中的 __sysdep.h__ ,  里面有  __INLINE_SYSCALL__ 定义

### Linux 创建 thread
Linux中实际上 thread 也是被当成 process 来创建, process和thread 是一样的, 区别在于系统调用时的传参.
![](/images/Linux_pthread_create_1.png)

### Linux boot process
refer： http://duartes.org/gustavo/blog/post/kernel-boot-process/
