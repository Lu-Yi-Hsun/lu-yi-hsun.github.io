---
title: "assambly"
date: 2021-01-30T05:23:00+08:00
draft: false
tags:
- assambly
- gcc
- linux
libraries:
- mermaid
---

注意要點

這種關於rip基底做偏移要注意%rip要帶入下一行指令的位置
0x1--`mov 0x2ee6(%rip),%rdx` 這裡%rip指向`0x1` 但執行`mov 0x2ee6(%rip),%rdx`的時候`%rip`要指向下一行指令所以帶入`0x2`,所以mov 0x2ee6(%rip)=mov 0x2ee6+0x2
0x2--`mov 1,%rax`

rip already points past the lea so the address of the next instruction should be used