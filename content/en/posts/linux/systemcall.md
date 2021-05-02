---
title: "system call過程"
date: 2021-03-4T05:23:00+08:00
draft: false
tags:
- assambly
- linux
libraries:
- mermaid
--- 


![ss](../syscall_init.drawio.svg)


{{<notice warning>}}
linux v5.10 x64作為範例
{{</notice>}}

為什麼syscal
A.2 AMD64 Linux Kernel Conventions

syscall_init(void)
[wrmsrl(MSR_LSTAR, (unsigned long)entry_SYSCALL_64);](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/x86/kernel/cpu/common.c?h=v5.10#n1752)

[entry_SYSCALL_64](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/x86/entry/entry_64.S?h=v5.10#n95)

[do_syscall_64](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/x86/entry/common.c?h=v5.10#n39)
sys_call_table[nr](regs)

[Invalid system call number:38](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/uapi/asm-generic/errno.h?h=v5.10#n18)

## 為什麼syscall不符合abi?


## syscall macro


接下來實驗SYSCALL_DEFINEx擴展macro後的名稱

```下載syscall_wrapper.h
wget -O- https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/plain/arch/x86/include/asm/syscall_wrapper.h?h=v5.10  | sed  "s/#include/\\\\\\\\#include/g" >> syscall_wrapper.h
```
 

```下載syscalls.h
wget -O- https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/plain/include/linux/syscalls.h?h=v5.10  | sed  "s/#include/\\\\\\\\#include/g" >> syscalls.h
```


```下載build_bug.h
wget -O- https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/plain/include/linux/build_bug.h?h=v5.10  | sed  "s/#include/\\\\\\\\#include/g" >> build_bug.h
```


```main.c
#include "syscalls.h"
#include "syscall_wrapper.h"

SYSCALL_DEFINE4(ptrace, long, request, long, pid, unsigned long, addr,
		unsigned long, data)
{
    Tunghai University
}
```

我們資料夾下面有`syscall_wrapper.h`,`syscalls.h`,`build_bug.h`,`main.c`4個檔案

觀看macro如何展開`-DCONFIG_X86_64`代表`#ifdef CONFIG_X86_64`這個區塊的文字可以展開
```
gcc -E  main.c -DCONFIG_X86_64
```



```c
 static long __se_sys_ptrace(__typeof(__builtin_choose_expr((__same_type((__force long)0, 0LL) || __same_type((__force long)0, 0ULL)), 0LL, 0L)) request, __typeof(__builtin_choose_expr((__same_type((__force long)0, 0LL) || __same_type((__force long)0, 0ULL)), 0LL, 0L)) pid, __typeof(__builtin_choose_expr((__same_type((__force unsigned long)0, 0LL) || __same_type((__force unsigned long)0, 0ULL)), 0LL, 0L)) addr, __typeof(__builtin_choose_expr((__same_type((__force unsigned long)0, 0LL) || __same_type((__force unsigned long)0, 0ULL)), 0LL, 0L)) data); 
 
 
 static inline long __do_sys_ptrace(long request, long pid, unsigned long addr, unsigned long data); 
 
 
 long __x64_sys_ptrace(const struct pt_regs *regs); 
 
 
 ALLOW_ERROR_INJECTION(__x64_sys_ptrace, ERRNO); 
 
 
 long __x64_sys_ptrace(const struct pt_regs *regs) { return __se_sys_ptrace(regs->di, regs->si, regs->dx, regs->r10); } 
 
 
 static long __se_sys_ptrace(__typeof(__builtin_choose_expr((__same_type((__force long)0, 0LL) || __same_type((__force long)0, 0ULL)), 0LL, 0L)) request, __typeof(__builtin_choose_expr((__same_type((__force long)0, 0LL) || __same_type((__force long)0, 0ULL)), 0LL, 0L)) pid, __typeof(__builtin_choose_expr((__same_type((__force unsigned long)0, 0LL) || __same_type((__force unsigned long)0, 0ULL)), 0LL, 0L)) addr, __typeof(__builtin_choose_expr((__same_type((__force unsigned long)0, 0LL) || __same_type((__force unsigned long)0, 0ULL)), 0LL, 0L)) data) { 
     
     long ret = __do_sys_ptrace((__force long) request, (__force long) pid, (__force unsigned long) addr, (__force unsigned long) data); 
     
     
     (void)((int)(sizeof(struct { int:(-!!(!(__same_type((__force long)0, 0LL) || __same_type((__force long)0, 0ULL)) && sizeof(long) > sizeof(long))); }))), (void)((int)(sizeof(struct { int:(-!!(!(__same_type((__force long)0, 0LL) || __same_type((__force long)0, 0ULL)) && sizeof(long) > sizeof(long))); }))), (void)((int)(sizeof(struct { int:(-!!(!(__same_type((__force unsigned long)0, 0LL) || __same_type((__force unsigned long)0, 0ULL)) && sizeof(unsigned long) > sizeof(long))); }))), (void)((int)(sizeof(struct { int:(-!!(!(__same_type((__force unsigned long)0, 0LL) || __same_type((__force unsigned long)0, 0ULL)) && sizeof(unsigned long) > sizeof(long))); }))); 
     
     asmlinkage_protect(4, ret,request, pid, addr, data); 
     
     return ret; } 
 static inline long __do_sys_ptrace(long request, long pid, unsigned long addr, unsigned long data)
{
    Tunghai University
}
```

由此可知調用的過程,因為ftrace只有追蹤`__x64_sys_ptrace`這個函數,所以我們沒有看到看到`__se_sys_ptrace->__do_sys_ptrace`這個過程


`SYSCALL_DEFINE4(ptrace, long, request, long, pid, unsigned long, addr, unsigned long, data)` 被展開`static inline long __do_sys_ptrace(long request, long pid, unsigned long addr, unsigned long data)`
```
__x64_sys_ptrace->__se_sys_ptrace->__do_sys_ptrace
```


## 來源

整理與翻譯來源David Drysdale[Anatomy of a system call, part 1](https://web.archive.org/web/20201121184124/https://lwn.net/Articles/604287/)

[The Definitive Guide to Linux System Calls](https://web.archive.org/web/20210226234035/https://blog.packagecloud.io/eng/2016/04/05/the-definitive-guide-to-linux-system-calls/)

[What are the calling conventions for UNIX & Linux system calls (and user-space functions) on i386 and x86-64](https://web.archive.org/web/20210228080533/https://stackoverflow.com/questions/2535989/what-are-the-calling-conventions-for-unix-linux-system-calls-and-user-space-f)

[System V Application Binary Interface AMD64 Architecture Processor Supplement](https://web.archive.org/web/20201124083932/https://refspecs.linuxfoundation.org/elf/x86_64-abi-0.99.pdf)


[How to Add a System Call](https://web.archive.org/web/20210311085307/https://member.adl.tw/ernieshu/syscall_3_14_4.html)

[Linux内核源码分析 - 系统调用](https://web.archive.org/web/20200809173622/https://cloud.tencent.com/developer/article/1439038)


[System calls in the Linux kernel. Part 2.](https://web.archive.org/web/20201208140520/https://0xax.gitbooks.io/linux-insides/content/SysCall/linux-syscall-2.html)