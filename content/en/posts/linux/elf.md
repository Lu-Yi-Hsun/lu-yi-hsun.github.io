---
title: "elf"
date: 2021-01-30T05:23:00+08:00
draft: false
tags:
- binary
- 系統程式
- 安全
libraries:
- mermaid
---

{{<notice warning>}}
接下來以linux v5.10 64bit elf作範例 
{{</notice>}}

## 簡介

Executable and Linking Format (ELF)這個格式包含了執行與連結用的檔案,可以搭配文件[elf.pdf](https://refspecs.linuxfoundation.org/elf/elf.pdf)與原始碼[elf.h](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/uapi/linux/elf.h?h=v5.10)一起看
## ELF header

小範例
```bash
readelf -h 你的檔名
```

```c
typedef struct elf64_hdr {
	unsigned char e_ident[16];	/* ELF "magic number" */
	Elf64_Half e_type;
	Elf64_Half e_machine;
	Elf64_Word e_version;
	Elf64_Addr e_entry;	/* Entry point virtual address */
	Elf64_Off e_phoff;	/* Program header table file offset */
	Elf64_Off e_shoff;	/* Section header table file offset */
	Elf64_Word e_flags;
	Elf64_Half e_ehsize;
	Elf64_Half e_phentsize;
	Elf64_Half e_phnum;
	Elf64_Half e_shentsize;
	Elf64_Half e_shnum;
	Elf64_Half e_shstrndx;
} Elf64_Ehdr;
```

### e_type

這邊描述elf檔案是什麼型態
| e_type    | 數值(unsigned short) | 用途                                                                                        |
| --------- | -------------------- | ------------------------------------------------------------------------------------------- |
| ET_NONE   | 0                    | No file type 未知格式(我在linux核心找不到有啥特別用途)                                       |
| ET_REL    | 1                    | Relocatable file 靜態連結.o .ko                                                             |
| ET_EXEC   | 2                    | Executable file 可以執行的檔案                                                              |
| ET_DYN    | 3                    | Shared object file 動態連結檔案.so,或是position-independent executable                                                     |
| ET_CORE   | 4                    | Core file 當發生core dumped如果有`設定`linux會輸出這個type的elf檔案讓你追蹤為什麼程式崩潰了 |
| ET_LOPROC | 0xff00               | Processor-specific ET_LOPROC~ET_HIPROC範圍的數值                                            |
| ET_HIPROC | 0xffff               | Processor-specific 同上                                                                     |

var/lib/systemd/coredump
coredumpctl

這種執行檔position independent executable aslr是ET_DYN型態要注意,並且entry point 不準確
Position-independent code
ET_DYN

> Don't communicate by sharing memory, share memory by communicating.</p>
> — <cite>Rob Pike[^app]</cite>



## 參考

 
[操作系统真象还原-5.3.3 elf 格式的二进制文件]  

[The Curious Case of Position Independent Executables](https://web.archive.org/web/20171120151138/https://eklitzke.org/position-independent-executables)


[^app]:[APP漏洞扫描器之未使用地址空间随机化](https://web.archive.org/web/20201130173004/https://developer.aliyun.com/article/65363)

[Learning Linux Binary Analysis]
 