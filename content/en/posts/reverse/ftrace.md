---
title: "ftrace"
date: 2021-01-30T05:23:00+08:00
draft: false
tags:
- 逆向工程
- 核心追蹤
- linux
- 安全
libraries:
- mermaid
---

## 介紹

ftrace是Steven Rostedt開發`trace-cmd`也是他開發的,接下來用`trace-cmd`示範
細節文件請看[ftrace.txt](https://www.kernel.org/doc/Documentation/trace/ftrace.txt)


## 查詢KERNEL CONFIG

### 法一
用`cat`讀出`/proc/config.gz`並且Pipeline(`|`)傳遞給`gunzip`解壓縮輸出(`>`)名為`kernel.config`的檔案
```bash
cat /proc/config.gz | gunzip > kernel.config
```
CONFIG_XXX替換你要查詢的字
```bash
cat kernel.config | grep "CONFIG_XXX"
```

### 法二
```
zcat /proc/config.gz | grep "CONFIG_XXX"
```



## trace-cmd

{{<alert theme="warning">}}
ftrace並不是每個函數都有追蹤,可以查詢哪些核心函數支援追蹤
{{</alert>}}

```bash
#察看支援追蹤哪些函數
sudo trace-cmd list -f
#察看支援追蹤哪些機制
sudo trace-cmd list -e
#察看支援追蹤哪些機制
sudo trace-cmd list -o
#察看有哪些支援的tracer可以plugin
sudo trace-cmd list -t
#hwlat blk mmiotrace function_graph wakeup_dl wakeup_rt wakeup function nop
```
 
### record


不同`tracer`會追蹤與顯示的方式不同,我喜歡`function`有興趣可以試試其他`tracer`,執行完會生成`trace.dat`檔案
```bash
#這裡plugin(-p)的tracer叫function
sudo trace-cmd record -p function ./你的程式
```

### report

`report`會讀你目錄底下的`trace.dat`

```bash
sudo trace-cmd report | grep "你有興趣核心函數名稱"
```
這裡的TIMESTAMP是microseconds

```txt
# tracer: function
#
# entries-in-buffer/entries-written: 144405/9452052   #P:4
#
#           TASK-PID   CPU#      TIMESTAMP  FUNCTION
#              | |       |          |         |
          <idle>-0     [002]  23636.756054: ttwu_do_activate.constprop.89 <-try_to_wake_up
          <idle>-0     [002]  23636.756054: activate_task <-ttwu_do_activate.constprop.89
          <idle>-0     [002]  23636.756055: enqueue_task <-activate_task

```
