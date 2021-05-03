---
title: "逆向工程"
date: 2021-01-30T05:23:00+08:00
 
draft: false
pinned: true
tags:
- 逆向工程
---

## 工具

- 靜態分析程式
    - Binwalk
    - Ghidra
- 動態分析程序
    - [gdb]({{< ref "gdb.md" >}})
- [Z3 Theorem Prover]({{< ref "z3.md" >}})
  - 超猛拉
- 分析linux核心
  - User-Mode Linux
  - bpftrace
  - [ftrace相關工具]({{< ref "ftrace.md" >}})

## 基礎知識
- microcode
  - 比機械碼還底層
    - Meltdown
    - Spectre
- 機械碼編碼方法
    - intel手冊
    - shellcode
- 組合語言
    - 定址模式
    - [abi]({{< ref "../abi.md" >}})
- 動態語言
- 檔案格式
  - 執行檔
    - [ELF]({{< ref "elf.md" >}})
    - PE
- c
  - [CERT Coding Standards ](https://wiki.sei.cmu.edu/confluence/)
    - 卡內基梅隆大學教你做人
  - [godbolt](https://gcc.godbolt.org/)
    - 線上看組合語言工具
  
 

 