---
title: "linux security module"
date: 2021-01-30T05:23:00+08:00
draft: false
tags:
- 安全
- linux
libraries:
- mermaid
---


`LSM_HOOK_INIT`與`call_int_hook`是linux kernel 的macro負責處理

舉例ptrace安全處理

security_ptrace_access_check

call_int_hook

LSM_HOOK_INIT(ptrace_access_check, cap_ptrace_access_check)


ftrace ptrace attach resault
```bash
cap_ptrace_access_check <-- security_ptrace_access_check
yama_ptrace_access_check <-- security_ptrace_access_check
apparmor_ptrace_access_check <-- security_ptrace_access_check
```