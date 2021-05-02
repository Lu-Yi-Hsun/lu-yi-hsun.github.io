---
title: "calling conventions"
date: 2021-01-30T05:23:00+08:00
draft: false
tags:
- ABI
---

其實[wiki x86 calling conventions](https://en.wikipedia.org/wiki/X86_calling_conventions#cdecl)寫的很完整,我只是整理一下

## 簡介

當函數調用的時候必須傳遞參數,但傳遞的時候到底如何`約定底層組合語言`？就稱為`calling conventions`

![call](../calling.drawio.svg)



 
## x86-64 System V AMD64 ABI

 