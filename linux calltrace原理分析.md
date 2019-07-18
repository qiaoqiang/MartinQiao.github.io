## Linux calltrace原理分析

本文介绍了在Linux环境下根据EABI标准进行call trace调试的一般性原理。



本文所说的call trace是指程序出问题时能把当前的函数调用栈打印出来。

本文只介绍了得到函数调用栈的一般性原理，没有涉及Linux的core dump机制。



下面简单介绍powerpc环境中如何实现call trace。

## 内核态call trace

内核态有三种出错情况，分别是bug, oops和panic。

bug属于轻微错误，比如在spin_lock期间调用了sleep，导致潜在的死锁问题，等等。

oops代表某一用户进程出现错误，需要杀死用户进程。这时如果用户进程占用了某些信号锁，所以这些信号锁将永远不会得到释放，这会导致系统潜在的不稳定性。

panic是严重错误，代表整个系统崩溃。

 

**OOPS**

先介绍下oops情况的处理。Linux oops时，会进入traps.c中的die函数。

```c
int die(const char *str, struct pt_regs *regs, long err)
{
       。。。

       show_regs(regs);

}
```



void show_regs(struct pt_regs * regs)函数中，会调用show_stack函数，这个函数会打印系统的内核态堆栈。

具体原理为：

```tex
    从寄存器里找到当前栈，在栈指针里会有上一级调用函数的栈指针，根据这个指针回溯到上一级的栈，依次类推。

    在powerpc的EABI标准中，当前栈的栈底（注意是栈底，不是栈顶，即Frame Header的地址）指针保存在寄存器GPR1中。在GPR1指向的栈空间，第一个DWORD为上一级调用函数的Frame Header指针（Back Chain Word），第二个DWORD是当前函数在上一级函数中的返回地址（LR Save Word）。通过此种方式一级级向上回溯，完成整个call dump。除了这种方法，内建函数__builtin_frame_address函数理论上也应该能用，虽然在内核中没有见到。（2.6.29的ftrace模块用到了 _builtin_return_address函数）。
```



show_regs函数在call trace的时候，只是用printk打印了一下栈中的信息。如果当前系统没有终端，那就需要修改内核，把这些栈信息根据需求保存到其它地方。

例如，可以在系统的flash中开出一块空间专门用于打印信息的保存。然后，写一个内核模块，再在die函数中加一个回调函数。这样，每当回调函数被调用，就通知自定义的内核模块，在模块中可以把调用栈还有其它感兴趣的信息保存到那块专用flash空间中去。这里有一点需要注意的是，oops时内核可能不稳定，所以为了确保信息能被正确写入flash，在写flash的函数中尽量不要用中断，而用轮循的方式。另外信号量、sleep等可能导致阻塞的函数也不要使用。

此外，由于oops时系统还在运行，所以可以发一个消息（信号，netlink等）到用户空间，通知用户空间做一些信息收集工作。

 

**Panic**

Panic时，Linux处于更最严重的错误状态，标志着整个系统不可用，即中断、进程调度等都已经停止，但栈还没被破坏。所以，oops中的栈回溯理论上还是能用。printk函数中因为没有阻塞，也还是能够使用。

## 用户态call trace

用户程序可以在以下情形call trace，以方便调试：

l         程序崩溃时，都会收到一个信号。Linux系统接收到某些信号时会自动打印call trace。

l         在用户程序中添加检查点，类似于assert机制，如果检查点的条件不满足，就执行call trace。

用户态的call trace与内核态相同，同样满足EABI标准，原理如下：

在GNU标准中，有一个内建函数__builtin_frame_address。这个函数可以返回当前执行上下文的栈底（Frame Header）指针（同时也是指向Back Chain Word的指针），通过这个指针得到当前调用栈。而这个调用栈中，会有上一级调用函数的栈底指针，通过这个指针再回溯到上一级的调用栈。以此类推完成整个call dump过程。

得到函数的地址后，可以通过符号表得到函数名字。如果是动态库中定义的函数，还可以通过扩展函数dladdr得到这个函数的动态库信息。



## 实例



```shell
Call Trace:
[  221.634988]  [<ffffffff8103fbc7>] ? kmld_pte_lookup+0x17/0x60
[  221.635016]  [<ffffffff8103ff04>] ? kmld_fault+0x94/0xf0
[  221.635051]  [<ffffffff8103fbc7>] ? kmld_pte_lookup+0x17/0x60
[  221.635079]  [<ffffffff8103ff04>] kmld_fault+0x94/0xf0
[  221.635109]  [<ffffffff815c8f95>] do_page_fault+0x3f5/0x550
[  221.635140]  [<ffffffff8104e36d>] ? set_next_entity+0x9d/0xb0
[  221.635169]  [<ffffffff8104e497>] ? pick_next_task_fair+0xc7/0x110
[  221.635209]  [<ffffffff815c5a95>] page_fault+0x25/0x30
[  221.635245]  [<ffffffff811675c7>] ? copy_strings+0x97/0x240
[  221.635272]  [<ffffffff811675af>] ? copy_strings+0x7f/0x240
[  221.635299]  [<ffffffff81167f92>] do_execve+0x202/0x2b0
[  221.635326]  [<ffffffff81012847>] sys_execve+0x47/0x70
[  221.635360]  [<ffffffff815cd81c>] stub_execve+0x6c/0xc0
```



  例如[ 221.634988] [<ffffffff8103fbc7>] ? kmld_pte_lookup+0x17/0x60

ffffffff8103fbc7 是指令在内存中的虚拟地址
kmld_pte_lookup  是函数(symbol 名)
0x17/0x60 ，0x60是这个函数编译成机器码后的长度，0x17是ffffffff8103fbc7这条指令在相对于kmld_pte_lookup这个函数入口的偏移  



