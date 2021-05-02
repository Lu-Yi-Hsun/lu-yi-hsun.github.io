---
title: "godbolt"
date: 2021-01-30T05:23:00+08:00
draft: false
tags:
- 逆向工程
---

## 介紹
[godbolt](https://gcc.godbolt.org/) 就是給你`看`,`各個語言`,`編譯器`,`不同參數` 出來的`機械碼`,`組合語言`,`執行`

## 範例

可以看到`xor %eax,%eax` 的機械碼拉與c語言的組合語言
Output:`compile to binary`,`x86-64 gcc 10.2`

```c
int main(){
    asm volatile(
        "xor  %%eax,%%eax\n"
    );
    return 0;
}
```
 