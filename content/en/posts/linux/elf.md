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

利用指令查看
```bash
readelf -h 你的檔名
```
原始碼[elf.h](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/uapi/linux/elf.h?h=v5.10)

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

這邊描述elf檔案是什麼型態包含了`ET_REL`,`ET_EXEC`,`ET_DYN`,`ET_CORE`,接下來簡單

| e_type    | 數值(unsigned short) | 用途                                                                                        |
| --------- | -------------------- | ------------------------------------------------------------------------------------------- |
| ET_NONE   | 0                    | No file type 未知格式(我在linux核心找不到有啥特別用途)                                       |
| ET_REL    | 1                    | Relocatable file 靜態連結  .o(object file) .a(靜態連結庫) .ko(kernel module)                                                            |
| ET_EXEC   | 2                    | Executable file 可以執行的檔案                                                              |
| ET_DYN    | 3                    | Shared object file 動態連結檔案.so,或是position-independent executable                                                     |
| ET_CORE   | 4                    | Core file 當發生core dumped如果有`設定`linux會輸出這個type的elf檔案讓你追蹤為什麼程式崩潰了 |
| ET_LOPROC | 0xff00               | Processor-specific ET_LOPROC~ET_HIPROC範圍的數值                                            |
| ET_HIPROC | 0xffff               | Processor-specific 同上                                                                     |

 


#### ET_REL

ET_REL型態的elf最主要是object file
[下載ET_REL範例](../ET_REL_lab.zip),裡面有2個檔案`function.c`和`main.c`

1.  產生 function.c object file
	```
	gcc -fPIC -c function.c
	```
2.  產生 main.c object file
	```
	gcc -fPIC -c main.c
	```

 
  
#### ET_EXEC
#### ET_DYN

#### ET_CORE
var/lib/systemd/coredump
coredumpctl

這種執行檔position independent executable aslr是ET_DYN型態要注意,並且entry point 不準確
Position-independent code
ET_DYN

> Don't communicate by sharing memory, share memory by communicating.</p>
> — <cite>Rob Pike[^app]</cite>


### e_machine

這個欄位是可以知道此elf檔案室屬於哪個架構的

細節請看[elf-em.h](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/uapi/linux/elf-em.h?h=v5.10)

其中比較特別的是近幾年比較紅的`RISC-V`,`BPF`(linux核心裡面的虛擬機,指令就是虛擬機專用的`bytecode`)
```c
#define EM_RISCV	243	/* RISC-V */
#define EM_BPF		247	/* Linux BPF - in-kernel virtual machine */
```
#### 簡單bpf xdp

這裡小範例讓大家體會什麼是XDP

```bash
sudo apt-get install git
sudo apt-get install clang
git clone https://github.com/libbpf/libbpf.git
```

```c:xdp_show_icmp.c
#include <linux/if_ether.h>//ethhdr
#include <linux/ip.h>//iphdr
#include <linux/in.h>//IPPROTO_ICMP
#include <linux/icmp.h>//icmphdr
#include <linux/bpf.h>
#include <bpf/bpf_helpers.h>
SEC("show_icmp")
int xdp_show_icmp(struct xdp_md *ctx)
{
    void *data = (void *)(long)ctx->data;
	if (ctx->data_end >= ctx->data + sizeof(struct ethhdr)+sizeof(struct iphdr)+sizeof(struct icmphdr)){
		struct iphdr *ip_header = data+sizeof(struct ethhdr);
		if(ip_header->protocol==IPPROTO_ICMP){
		bpf_printk("I Get The ICMP\n");
		}
	} 
    return XDP_PASS;
}
char _license[] SEC("license") = "GPL";
```


```
clang -O2 -target bpf -c xdp_show_icmp.c -o xdp_show_icmp.o
```

```
sudo ip link set dev lo xdpgeneric obj xdp_show_icmp.o sec show_icmp
```




## 進階探討ELF
https://web.archive.org/web/20210415124218/http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html


## 參考

 
[操作系统真象还原-5.3.3 elf 格式的二进制文件]  

[The Curious Case of Position Independent Executables](https://web.archive.org/web/20171120151138/https://eklitzke.org/position-independent-executables)


[^app]:[APP漏洞扫描器之未使用地址空间随机化](https://web.archive.org/web/20201130173004/https://developer.aliyun.com/article/65363)

[Learning Linux Binary Analysis]
 