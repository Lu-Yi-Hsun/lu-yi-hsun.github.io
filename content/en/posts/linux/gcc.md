---
title: "c inline assambly and system call"
date: 2021-01-30T05:23:00+08:00
draft: false
tags:
- assambly
- gcc
- linux
libraries:
- mermaid
---


## 簡介

這裡我們要學到如何利用c語言直接呼叫`system call`做到`printf`顯示的功能

## 範例
實做直接呼叫linux system code

根據1-1表格找到的資訊

關於此段指令為什麼填入1的理由

["mov $1, %%rdi\n"](https://stackoverflow.com/questions/5256599/what-are-file-descriptors-explained-in-simple-terms/5257718#5257718)
 
```c
#include <sys/syscall.h>
int main(void){
 syscall(SYS_write, 1, "hello, world!\n", 14);
}
```

```c
void print_asm(char *arg1,long int size){ 
    __asm__ volatile(
        "mov $1, %%rax\n\t"//system call 編碼
        "mov $1, %%rdi\n\t"//fd 設定1 代表把字串輸入/dev/stdout 這裡就是螢幕輸出地方
        "mov %0,%%rsi\n\t"//輸入字串記憶體位置
        "mov %1, %%rdx\n\t" //這裡輸入字串長度 ,可以跟記憶體位置搭配來輸出到螢幕
        "syscall"//x64 要用此呼叫systemcall 不能在使用int $0x80
        :
        :"m" (arg1),"m" (size) //詳細請參考gcc inline asm
    );
}
int main(void) {
    char *d="hello, world!\n";
    print_asm(d,14);
return 0;
}
```

## System call 傳遞約定  

{{<notice info 預備學習知>}}
[calling_conventions]({{< ref "../linux/calling_conventions" >}})
{{</notice>}}
 
我們要先了解System call 傳遞約定才能知道要如何傳遞我們要的參數告訴linux作業系統



[根據linux原始碼](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/arch/x86/entry/entry_64.S?h=v5.10#n76)我們可以建立下面的表格
 
 

表格1-1

|system call number||||||||
:---|:----|:----|:----|:----|:----|:----|:----|:----
ABI規範|1|__x64_sys_write|fd|buf|count


|||||||||
:---|:----|:----|:----|:----|:----|:----|:----|:----
x64 syscall|%rax|%rdi|%rsi|%rdx|%r10|%r8|%r9|

為什麼x64 syscall不符合abi?

%rax|System call|%rdi|%rsi|%rdx|%r10|%r8|%r9|
:----|:----|:----|:----|:----|:----|:----|:----
1|__x64_sys_write|fd|buf|count


[根據calling_conventions](../calling_conventions)


   
 

## C內嵌ASM

`__asm__`與`asm`都可以使用
但只有`__asm__` 可以在`-ansi`或`-std`(標準c規範)下使用,[原因](https://web.archive.org/web/20201231215128/https://gcc.gnu.org/onlinedocs/gcc/Alternate-Keywords.html)

```
__asm__ asm-qualifiers ( AssemblerTemplate 
                      : OutputOperands
                      : InputOperands
                      : Clobbers
                      : GotoLabels)
```

### asm-qualifiers

- volatile
    > volatile無法完美讓內嵌組合語言避免被編譯器更動
    >Do not expect a sequence of asm statements to remain perfectly consecutive after compilation, even when you are using the volatile qualifier. If certain instructions need to remain consecutive in the output, put them in a single multi-instruction asm statement.


### AssemblerTemplate

這裡放組合語言
- 指令換行"\n\t"


### OutputOperands
 
```
[ [asmSymbolicName] ] constraint (cvariablename)
```
#### constraint
prefix `=` or `+`

這裡寫要輸出變數
### InputOperands
這裡寫要輸入變數
### Clobbers


>編譯器不會用到Clobbers裏面設定的暫存器

>When the compiler selects which registers to use to represent input and output operands, it does not use any of the clobbered registers. As a result, clobbered registers are available for any use in the assembler code.

{{< tabs "side effect" "fix by Clobbers" "fix by constraint"  >}}
  {{< tab >}}
  這個範例會引發`side effect`,導致不符合預期
  因為`[arg1]"r" (arg1)`的`r`所以arg1數值會被塞到`某個暫存器`來使用,但接下來`rax`,`rdi`,`rsi`,`rdx`暫存器會被寫入,所以原本借放arg1數值的暫存器有可能被覆蓋導致不符合預期

```c
#include<stdio.h>
long long int print_asm(char *arg1,long int size){ 
    long long int rax;
    __asm__ volatile(
        "mov $1, %%rax\t\n"//system call 編碼
        "mov $1, %%rdi\t\n"//fd 設定1 代表把字串輸入/dev/stdout 這裡就是螢幕輸出地方
        "mov %[arg1],%%rsi\t\n"//輸入字串記憶體位置
        "mov %[size], %%rdx\t\n" //這裡輸入字串長度 ,可以跟記憶體位置搭配來輸出到螢幕
        "syscall\t\n"//x64 要用此呼叫systemcall 不能在使用int $0x80
        "mov %%rax,%[rax]\t\n"
        :[rax]"=m" (rax)
        :[arg1]"r" (arg1),[size]"m" (size)//詳細請參考gcc inline asm
        : 
    );
    
    return rax;
}
int main(void) {
    char *d="hello, world!\n";
    int rax=print_asm(d,14);
    printf("%d",rax);
return 0;
}
```
  {{< /tab >}}
  {{< tab >}}

告訴編譯器輸入與輸出不能用到"rax","rdi","rsi","rdx"暫存器
{{< highlight c "hl_lines=13" >}}
#include<stdio.h>
long long int print_asm(char *arg1,long int size){ 
    long long int rax;
    __asm__ volatile(
        "mov $1, %%rax\t\n"//system call 編碼
        "mov $1, %%rdi\t\n"//fd 設定1 代表把字串輸入/dev/stdout 這裡就是螢幕輸出地方
        "mov %[arg1],%%rsi\t\n"//輸入字串記憶體位置
        "mov %[size], %%rdx\t\n" //這裡輸入字串長度 ,可以跟記憶體位置搭配來輸出到螢幕
        "syscall\t\n"//x64 要用此呼叫systemcall 不能在使用int $0x80
        "mov %%rax,%[rax]\t\n"
        :[rax]"=r" (rax)
        :[arg1]"r" (arg1),[size]"m" (size)//詳細請參考gcc inline asm
        :"rax","rdi","rsi","rdx"
    );
    return rax;
}
int main(void) {
    char *d="hello, world!\n";
    int rax=print_asm(d,14);
    printf("%d",rax);
return 0;
}
{{< / highlight >}}


 
  {{< /tab >}}
  

   {{< tab >}}

解法二直接用記憶體位置,不利用暫存器借放
{{< highlight c "hl_lines=12" >}}
#include<stdio.h>
long long int print_asm(char *arg1,long int size){ 
    long long int rax;
    __asm__ volatile(
        "mov $1, %%rax\t\n"//system call 編碼
        "mov $1, %%rdi\t\n"//fd 設定1 代表把字串輸入/dev/stdout 這裡就是螢幕輸出地方
        "mov %[arg1],%%rsi\t\n"//輸入字串記憶體位置
        "mov %[size], %%rdx\t\n" //這裡輸入字串長度 ,可以跟記憶體位置搭配來輸出到螢幕
        "syscall\t\n"//x64 要用此呼叫systemcall 不能在使用int $0x80
        "mov %%rax,%[rax]\t\n"
        :[rax]"=r" (rax)
        :[arg1]"m" (arg1),[size]"m" (size)//詳細請參考gcc inline asm
        :
    );
    return rax;
}
int main(void) {
    char *d="hello, world!\n";
    int rax=print_asm(d,14);
    printf("%d",rax);
return 0;
}
{{< / highlight >}}



    {{< /tab >}}
{{< /tabs >}}
 



### GotoLabels


## 參考資源

[C/C++ function definitions without assembly](https://stackoverflow.com/questions/2442966/c-c-function-definitions-without-assembly/2444508#2444508)

[How can I find the implementations of Linux kernel system calls?](https://unix.stackexchange.com/questions/797/how-can-i-find-the-implementations-of-linux-kernel-system-calls)

[linux system call](http://www.jollen.org/blog/2006/10/linux_26_system_call12.html#c2)
[Inline Assembly & Memory Barrier](https://ithelp.ithome.com.tw/articles/10213500)
