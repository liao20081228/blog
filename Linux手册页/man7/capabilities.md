---
title: capabilities
tags: man7
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Expect manpages》，</font>expect当前版本为5.45.4，手册更新日期为1994-12-29。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------




# 描述
## 能力列表

**CAP_CHOWN** 
* 对文件UID和GID进行任意更改（请参见Chown(2）)。
  
**CAP_DAC_OVERRIDE**
* 绕过文件读、写和执行权限检查。（DAC是“自由访问控制”的缩写。）在DAC权限方面CAP_DAC_OVERRIDE 是 CAP_DAC_READ_SEARCH的超集，但是CAP_DAC_READ_SEARCH还有DAC之外的能力。
  
**CAP_DAC_READ_SEARCH**
* 绕过文件读取权限检查、目录读取和执行权限检查；
* 调用open_by_handle_at(2)；
* 使用linkat(2) AT_EMPTY_PATH标志创建指向文件描述符引用的文件的链接。
  
**CAP_FOWNER**
* 绕过通常要求进程的文件系统UID与文件UID匹配的操作的权限检查（例如，chmod(2）、utime(2))，不包括CAP_DAC_OVERRIDE和CAP_DAC_READ_SEARCH所涵盖的操作；
* 在任意文件上设置inode标志（见ioctl_iflags(2）)；
* 在任意文件上设置访问控制列表（ACL）；
*  删除文件时忽略目录粘滞位；
* 修改任何用户拥有的粘滞目录的用户扩展属性；
* 在open(2)和fcntl(2)中为任意文件指定O_NOATIME。
 
**CAP_FSETID**
* 修改文件时，不要清除set-user-ID和set-group-ID模式位；
* 为GID与文件系统或调用进程的任何附加GID不匹配的文件设置set-group-ID位。
  
**CAP_KILL**
* 绕过发送信号的权限检查（请参见kill(2）)。这包括使用ioctl(2)KDSIGACC操作。

**CAP_SETGID**
* 对进程GID和附加GID列表进行任意操作；
* 通过UNIX域套接字传递套接字凭据时伪造GID；
* 在用户命名空间中写入组ID映射（请参见user_namespaces(7）)。

**CAP_SETUID**
* 对进程UID进行任意操作(setuid(2)、setreuid(2)、setresuid(2)、setfsuid(2))；
* 通过UNIX域套接字传递套接字凭据时伪造UID；
* 在用户命名空间中写入用户ID映射（请参见user_namespaces(7）)。
  
**CAP_SETPCAP**
* 如果支持文件能力（即，自Linux 2.6.24起）：将调用线程的边界能力集中的任意能力添加到其可继承能力集；从边界能力集中删除能力（通过prctl(2）PR_CAPBSET_DROP)；更改安全位标志。

* 如果文件功能不受支持（即Linux 2.6.24之前的内核）：向任何其他进程授予或删除调用者允许能力集中的任何能力。（当内核配置为支持文件能力时，CAP_SETPCAP的此属性不可用，因为CAP_SETPCAP对此类内核具有完全不同的语义。）

CAP_LINUX_IMMUTABLE	
FS_APPEND_FL和FS_IMMUTABLE_FL inode标志（请参见ioctl_iflags(2）)。
CAP_NET_BIND_SERVICE
将套接字绑定到Internet域特权端口（端口号小于1024）。
CAP_NET_BROADCAST
未使用）进行套接字广播，并侦听组播。
CAP_NET_ADMIN
执行各种网络相关操作：
·接口配置；
·IP防火墙、伪装和计费管理；
·修改路由表；
·绑定到任何地址进行透明代理；
·设置服务类型（TOS）；
·清楚驱动统计数据；
·设置混杂模式；
·启用多播；
·使用setockopt(2)设置以下套接字选项：SO_DEBUG、SO_标记、SO_PRIORITY（对于范围0到6之外的优先级）、SO_RCVBuffORCE和SO_SNDBuffORCE。"

CAP_NET_RAW
*使用RAW和数据包套接字；
*绑定到任何地址进行透明代理。"

CAP_IPC_LOCK
锁定内存(mlock(2)、mlockall(2)、mmap(2)、shmctl(2))。

CAP_IPC_OWNER
绕过对System V IPC对象的操作权限检查。

CAP_SYS_MODULE
*加载和卸载内核模块（请参见init_module(2）和delete_module(2))；
*在2.6.25之前的内核中：从系统范围的边界能力集中删除能力。"

CAP_SYS_RAWIO	17	"*执行I/O端口操作(ipl(2)和ioperm(2))；
*访问/proc/kcore；
*使用FIBMAP ioctl(2)操作；
*打开用于访问x86型号特定寄存器的设备（MSRs，见msr(4）)；
*更新/proc/sys/vm/mmap_min_addr；
*在低于/proc/sys/vm/mmap_min_addr指定值的地址创建内存映射；
*/proc/bus/pci目录下的映射文件；
*打开/dev/mem和/dev/kmem；
*执行各种SCSI设备命令；
*对hpsa(4)和cciss(4)设备进行某些操作；
*在其他设备上执行一系列特定于设备的操作。"
CAP_SYS_CHROOT	18	"*使用chroot(2)；
*使用setns(2)更改挂载命名空间。"
CAP_SYS_PTRACE	19	"*使用ptrace(2)跟踪任意进程；
*将get_robust_list(2)应用于任意进程；
*使用process_vm_readv(2)和process_vm_writev(2)将数据传输到任意进程的内存或从任意进程的内存中传输数据；
*使用kcmp(2)检查进程。"
CAP_SYS_PACCT	20	使用acct(2)
CAP_SYS_ADMIN	21	"注意：此能力已过载；请参阅下面的内核开发人员注释。

*执行一系列系统管理操作，包括：quotactl(2)、 mount(2)、umount(2)、pivot_root(2)、swapon(2)、swapoff(2)、sethostname(2)和setdomainnameset(2)；
*执行特权syslog(2)操作（自Linux 2.6.37起，应使用CAP_SYSLOG允许此类操作）；
*执行VM86_REQUEST_IRQ vm86(2)命令；
*访问由CAP_CHECKPOINT_RESTORE管理的相同检查点/恢复功能（但后者，访问该功能时首选较弱的能力）。
*执行与CAP_BPF管理的相同的BPF操作（但后者，访问该功能时首选较弱的能力）。
*使用与CAP_PERFMON管理的相同的性能监控机制（但后者，访问该功能时首选较弱的能力）。
*对任意System V IPC对象执行IPC_SET和IPC_RMID操作；
*覆盖RLIMIT_NPROC资源限制；
*对可信和安全扩展属性执行操作（见xattr(7）)；
*使用lookup_dcookie(2)；
*使用ioprio_set(2)分配IOPRIO_CLASS_RT和（Linux 2.6.25之前）IOPRIO_CLASS_IDLE I/O调度类；
*通过UNIX域套接字传递套接字凭据时伪造PID；
*在打开文件的系统调用中（例如accept(2), execve(2), open(2), pipe(2)），超过/proc/sys/fs/file-max，即打开文件数量的系统范围限制；
*与clone（2）和unshare（2）一起使用 CLONE_*标志创建新的命名空间（但是，自Linux 3.8以来，创建用户命名空间不需要任何能力）；
*访问特权性能事件信息；
*调用setns（2）（需要目标命名空间中的CAP_SYS_ADMIN）；
*调用fanotify_init(2)；
*执行KEYCTL_CHOWN和键CTL_SETPERM keyctl(2)操作；
*执行madvise(2) MADV_HWPOSON操作；
*使用TIOCSTI ioctl(2)向调用者的控制终端以外的终端的输入队列插入字符；
*使用过时的nfsservctl(2)系统调用；
*使用过时的bdflush(2)系统调用；
*执行各种特权块设备ioctl(2)操作；
*执行各种特权文件系统ioctl(2)操作；
*在/dev/random设备上执行特权ioctl(2)操作（见random(4）)；
*安装一个seccomp(2)过滤器，而不必首先设置no_new_pris线程属性；
*修改设备控制组的允许/拒绝规则；
*使用ptrace(2) PTRACE_SECCOMP_GET_FILTER操作转储tracee的seccomp过滤器；
*使用ptrace(2) PTRACE_SETOPTIONS操作挂起tracee的seccomp保护（即PTRACE_O_SUSPEND_SECCOMP标志）；
*对许多设备驱动程序执行管理操作；
*通过写入/proc/[pid]/autogroup（请参见Sched(7）)来修改自动组的nice值。"
CAP_SYS_BOOT	22	使用reboot(2)和kexec_load(2)。
CAP_SYS_NICE	23	"*降低进程nice值（nice(2），setpriority（2）)，以及更改任意进程的nice值；
*设置调用进程的实时调度策略，设置任意进程的调度策略和优先级（sched_setscheduler(2）、sched_setparam(2)、sched_setattr(2))；
*为任意进程设置CPU亲和性（sched_setaffinity(2）)；
*设置任意进程的I/O调度类别和优先级（ioprio_set(2）)；
*将migrate_pages（2）应用于任意进程，并允许进程迁移到任意节点；
*将move_pages（2）应用于任意进程；
*将MPOL_MF_MOVE_ALL标志与mbind(2)和move_pages（2）一起使用。"
CAP_SYS_RESOURCE	24	"*使用ext2文件系统上的保留空间；
*进行ioctl(2)调用控制ext3日志记录；
*覆盖磁盘配额限制；
*增加资源限制（见setrlimit(2）)；
*覆盖RLIMIT_NPROC资源限制；
*在控制台分配中覆盖最大控制台数量；
*覆盖keymap的最大数量；
*允许实时时钟产生超过64hz的中断；
*将System V消息队列的msg_qbytes限制提高到/proc/sys/kernel/msgmnb中的限制以上（见msgop(2）和msgctl(2))；
*允许在通过UNIX域套接字将文件描述符传递给另一个进程时绕过对“in-flight”文件描述符数量的RLIMIT_NOFILE资源限制（见unix(7）)；
*使用F_SETPIPE_SZ fcntl(2)命令设置管道容量时，覆盖/proc/sys/fs/pipe-size-max限制；
*使用F_SETPIPE_SZ将管道的容量增加到/proc/sys/fs/pipe-max-size指定的限制以上；
*创建POSIX消息队列时，覆盖/proc/sys/fs/mqueue/queues_max、/proc/sys/fs/mqueue/msg_max，/proc/sys/fs/mqueue/msgsize_max限制（见mq_overview(7）)；
*使用prctl(2) PR_SET_MM操作；
*将/proc/[pid]/oom_score_adj 设置为低于有CAP_SYS_RESOURCE的进程上次设置的值。"
CAP_SYS_TIME	25	设置系统时钟(settimeofday(2)、stime(2)、adtimex(2))；设置实时（硬件）时钟。
CAP_SYS_TTY_CONFIG	26	使用vhangup(2)；在虚拟终端上使用各种特权ioctl(2)操作。
CAP_MKNOD	27	使用mknod(2)创建特殊文件。
CAP_LEASE	28	在任意文件上建立租约（请参见fcntl(2）)。
CAP_AUDIT_WRITE	29	向内核审计日志写记录
CAP_AUDIT_CONTROL	30	启用和禁用内核审计、修改审计过滤器规则、检索审计状态和过滤规则
CAP_FSETCAP	31	在文件上设置任意能力。（文件能力包括P E I，进程能力还包括B A）
CAP_MAC_OVERRIDE	32	覆盖强制访问控制(MAC)。为Smack LSM实现。
CAP_MAC_ADMIN	33	允许MAC配置或状态更改。为Smack Linux安全模块（LSM）实现。
CAP_SYSLOG	34	"*执行特权的syslog(2)操作。有关哪些操作需要权限的信息，请参见syslog(2)。
*当/proc/sys/kernel/kptr_restrict值为1时，查看通过/proc和其他接口公开的内核地址。（请参见proc(5）中对kptr_restrict的讨论。)"
CAP_WAKE_ALARM	35	触发将唤醒系统的东西（设置CLOCK_REALTIME_ALARM和CLOCK_BOOTTIME_ALARM 计时器）。
CAP_BLOCK_SUSPEND	36	使用可以阻塞系统挂起的特性(epoll(7) EPOLLWAKEUP, /proc/sys/wake_lock).
CAP_AUDIT_READ	37	允许通过一个多播netlink socket读取审计日志
CAP_PERFMON	38	"采用各种性能监测机制，包括：
*调用perf_event_open(2)；
*采用各种具有性能影响的BPF操作。
此功能是在Linux 5.8中添加的，目的是将性能监视功能与过载的CAP_SYS_ADMIN能力分开。另请参见内核源文件文档/admin-Guide/perf-security.rst。"
CAP_BPF	39	"使用特权BPF操作；请参见bpf(2)和bpf-helpers(7)。
此功能是在Linux 5.8中添加的，以将BPF功能与过载的CAP_SYS_ADMIN能力分开。"
CAP_CHECKPOINT_RESTORE	40	"*更新/proc/sys/kernel/ns_last_pid（见pid_namespaces(7）)；
*使用clone3(2)的set_tid特性；
*读取/proc/[pid]/map_file中其他进程的符号链接的内容。

此能力是在Linux 5.9中添加的，用于将检查点/恢复功能与过载的CAP_SYS_ADMIN功能分开。"
		
