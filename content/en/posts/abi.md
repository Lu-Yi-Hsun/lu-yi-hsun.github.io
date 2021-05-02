---
title: "什麼是ABI？"
date: 2021-01-30T02:40:00+08:00
draft: false
tags:
- ABI
---

## 所以是什麼？

我們比較abi與api
- api
  - 你寫程式呼叫函式庫(#include<stdio.h>)與api(printf)
- abi
  - 編譯完成後,約定底層的行為
 
### 誰需要知道abi

開發編譯器,組合語言,作業系統的人
https://stackoverflow.com/questions/2171177/what-is-an-application-binary-interface-abi


## ABI範例
- [RISC-V ELF psABI specification](https://github.com/riscv/riscv-elf-psabi-doc/blob/master/riscv-elf.md)
    - 定義riscv每個暫存器負責什麼
- [Procedure Call Standard for the Arm Architecture](https://developer.arm.com/documentation/ihi0042/latest/)
    - 定義arm每個暫存器負責什麼
- System V AMD64 ABI
    - 規定Solaris，GNU/Linux，FreeBSD和其他非微軟OS如何傳遞
    - stack如何規劃與維護
    - 定義x86每個暫存器負責什麼
- Itanium C++ ABI
    - 確保編譯出來的檔案可以互相操作,舉例object file可以link在一起,或是可以呼叫動/靜連結庫
    - GCC and Clang都是依照此abi所規範,所以可以相容object file

- https://abi-laboratory.pro

### python abi 範例
- https://eklitzke.org/an-unexpected-python-abi-change
    - abi bug
- https://www.python.org/dev/peps/pep-0425/#abi-tag
    - 打包版本
- https://www.python.org/dev/peps/pep-0427/
  - The Wheel Binary Package Format

![call](../python_abi.drawio.svg)


### c

- https://imzlp.me/posts/5392/
    - c與c++