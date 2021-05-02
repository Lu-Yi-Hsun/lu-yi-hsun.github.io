---
title: "canary analysis"
date: 2021-01-30T05:23:00+08:00
draft: false
tags:
- assambly
- gcc
- linux
- 安全
- pwn
libraries:
- mermaid
---
 
https://ctf-wiki.org/pwn/linux/mitigation/canary/

你的程式
https://hardenedlinux.github.io/2016/11/27/canary.html


https://stackoverflow.com/questions/10325713/why-does-this-memory-address-fs0x28-fs0x28-have-a-random-value

## 範例

### 題目
下面程式會觸發smashing detected請修好下面的程式
```
** stack smashing detected ***: terminated
```
```c
#include<stdio.h>
int print_asm(char *arg1,long int size){ 
     int rax;
    __asm__ volatile(
        "mov $1, %%rax\t\n"//system call 編碼
        "mov $1, %%rdi\t\n"//fd 設定1 代表把字串輸入/dev/stdout 這裡就是螢幕輸出地方
        "mov %[arg1],%%rsi\t\n"//輸入字串記憶體位置
        "mov %[size], %%rdx\t\n" //這裡輸入字串長度 ,可以跟記憶體位置搭配來輸出到螢幕
        "syscall\t\n"//x64 要用此呼叫systemcall 不能在使用int $0x80
        "mov %%rax,%[rax]\t\n"//根據amd64 abi 回傳數值放在rax
        :[rax]"=m"(rax)
        :[arg1]"m" (arg1),[size]"m" (size) //詳細請參考gcc inline asm
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
### 再思考一下

執行到第3行(int rax;)記憶體與cpu
![ss](../stack.drawio.svg)
執行到第10行( `"mov %%rax,%[rax]\t\n"`)可以看到原先規劃`4bytes`(int rax)被塞入`8ytes`(紅色部份)直接重疊到canary value
![ss](../stack2.drawio.svg)
### 答案
{{< expand "答案" >}}
```c {linenos=table,hl_lines=["2-3"]} 
#include<stdio.h>
long long int print_asm(char *arg1,long int size){ 
    long long int rax;
    __asm__ volatile(
        "mov $1, %%rax\t\n"//system call 編碼
        "mov $1, %%rdi\t\n"//fd 設定1 代表把字串輸入/dev/stdout 這裡就是螢幕輸出地方
        "mov %[arg1],%%rsi\t\n"//輸入字串記憶體位置
        "mov %[size], %%rdx\t\n" //這裡輸入字串長度 ,可以跟記憶體位置搭配來輸出到螢幕
        "syscall\t\n"//x64 要用此呼叫systemcall 不能在使用int $0x80
        "mov %%rax,%[rax]\t\n"//根據amd64 abi 回傳數值放在rax所以接收的大小要符合8byte不然可能溢位
        :[rax]"=m"(rax)
        :[arg1]"m" (arg1),[size]"m" (size) //詳細請參考gcc inline asm
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
{{< /expand >}}

