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
接下來以linux v5.10 64bit Elf64作範例 
{{</notice>}}

## 簡介

![elf_arch](../elf_arch.drawio.svg)

Executable and Linking Format (ELF)這個格式包含了執行與連結用的檔案,可以搭配文件交叉看
- [elf.pdf](https://refspecs.linuxfoundation.org/elf/elf.pdf)
- 原始碼[elf.h](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/uapi/linux/elf.h?h=v5.10)
- linux載入elf行為-[load_elf_binary](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/fs/binfmt_elf.c?h=v5.10#n820)
- [man](https://man7.org/linux/man-pages/man5/elf.5.html)
- [eheader](https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.eheader.html)
- [ELF文件可执行栈的深入分析](https://mudongliang.github.io/2015/10/23/elf.html)

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
### e_ident

| Name    | e_ident第幾個bytes | Purpose                                                                                        |
| --------- | -------------------- | ---------------------------------------------------------------------- |
| EI_MAG0   | 0                    | 0x7f                                      |
|EI_MAG1	|1	|'E'
|EI_MAG2	|2	|'L'
|EI_MAG3	|3	|'F'
|EI_CLASS	|4	|ELFCLASSNONE=0</br>ELFCLASS32=1</br>ELFCLASS64=2
|EI_DATA	|5	|ELFDATANONE=0</br>ELFDATA2LSB=1 又稱little endian</br>ELFDATA2MSB=2 又稱big endian
|EI_VERSION	|6	|原始的ELF specification規範這裡固定是1,除非你要設計自己客製擴展ELF的規範不然`e_version`與`EI_VERSION`都是固定1
|EI_OSABI	|7	|ELFOSABI_NONE/ELFOSABI_SYSV 0 UNIX System V ABI</br>ELFOSABI_HPUX	1	Hewlett-Packard HP-UX</br>ELFOSABI_NETBSD	2	NetBSD</br>ELFOSABI_LINUX	3	Linux</br>ELFOSABI_SOLARIS	6	Sun Solaris</br>ELFOSABI_AIX	7	AIX</br>ELFOSABI_IRIX	8	IRIX</br>ELFOSABI_FREEBSD	9	FreeBSD</br>ELFOSABI_TRU64	10	Compaq TRU64 UNIX</br>ELFOSABI_MODESTO	11	Novell Modesto</br>ELFOSABI_OPENBSD	12	Open BSD</br>ELFOSABI_OPENVMS	13	Open VMS</br>ELFOSABI_NSK	14	Hewlett-Packard Non-Stop Kernel</br>
|EI_ABIVERSION	|8	|ABI version
|EI_PAD	|9	|Start of padding bytes 從這裡開始就補0直到最後 關於padding請看(你所不知道的 C 語言：記憶體管理、對齊及硬體特性)[https://hackmd.io/@sysprog/c-memory#data-alignment]
|EI_NIDENT	|16	|Size of e_ident[]

{{<notice info>}}
嘗試利用[vscode的Hex Editor](https://marketplace.visualstudio.com/items?itemName=ms-vscode.hexeditor)修改執行檔並且利用readelf觀察
{{</notice>}}
 
 
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

##### object file
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
3. 觀察object file的elf type
	```
	readelf -h function.o
	```
##### kernel module

[下載kernel_module範例](../kernel_module.zip)

編譯
```
make
```
載入kernel module
```
sudo insmod hello.ko
```
查詢kernel module已經載入了哪些
```
sudo lsmod | grep "hello"
```
刪除kernel module
```
sudo rmmod hello.ko
```
查詢printk輸出什麼
```
dmesg
```
我們來觀察kernel module的elf type
```
readelf -h hello.ko
```

#### ET_EXEC 

這種類型的執行檔載入到固定記憶體位置,所以e_entry的位置就是固定的位置

```c:main.c
#include<stdio.h>
int main(){
	printf("Hello World\n");
	return 0;
}
```
編譯
```
gcc -no-pie main.c
```
觀察型態
```
readelf-h a.out 
```
#### ET_DYN

##### position-independent executable
這種類型的執行檔才可以被作業系隨機載入到不同記憶體位置
```c:main.c
#include<stdio.h>
int main(){
	printf("Hello World\n");
	return 0;
}
```
編譯
```
gcc -pie main.c
```
觀察型態
```
readelf-h a.out 
```
##### share libary
接下來繼續用剛才生成a.out來查詢share libary是哪種elf型態
 
查詢a.out需要動態連結哪些share libary
```
ldd a.out
```
我的電腦就出現`/lib/x86_64-linux-gnu/libc.so.6`
```
linux-vdso.so.1 (0x00007ffcb0cba000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f5f6cd3a000)
/lib64/ld-linux-x86-64.so.2 (0x00007f5f6cf4e000)
```

分析看看是什麼型態
```
readelf -h /lib/x86_64-linux-gnu/libc.so.6
```

#### ET_CORE

這種型態的elf是當你的程式發生core dumped,作業系統就會吐出來`ET_CORE`的elf讓你用gdb來分析

設定core dump檔案的大小不限制
```
ulimit -c unlimited
```
我們故意寫一個會出錯的程式
```c:main.c
#include<stdio.h>
int main(){
        int *a=0;
        printf("%d",*a);
        return 0;
}
```
編譯
```
gcc -g main.c
```
執行`a.out`
```
./a.out
```

發生`Segmentation fault (core dumped)`並且看到core這個檔案

我們來觀察`core`的type
```
readelf -h core
```

我們來利用gdb與core來幫助我們分析哪裡出錯了

```
gdb a.out core
```

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

##### 安裝
```bash
sudo apt-get install git
sudo apt-get install clang
git clone https://github.com/libbpf/libbpf.git
```
##### 創建檔案
```c:xdp_show_icmp.c
#include <linux/if_ether.h>//ethhdr
#include <linux/ip.h>//iphdr
#include <linux/in.h>//IPPROTO_ICMP
#include <linux/icmp.h>//icmphdr
#include <linux/bpf.h>
#include "libbpf/src/bpf_helpers.h"
SEC("show_icmp")
int xdp_show_icmp(struct xdp_md *ctx)
{	//ctx->data代表收到封包的起始記憶體位置
	//ctx->data_end代表收到封包的結束記憶體位置
    void *data = (void *)(long)ctx->data;
	//ctx->data起始記憶體位置+EtherType大小+IPv4 header大小+ICMP header大小希望小於等於結束記憶體位置
	if (ctx->data_end >= ctx->data + sizeof(struct ethhdr)+sizeof(struct iphdr)+sizeof(struct icmphdr)){
		struct iphdr *ip_header = data+sizeof(struct ethhdr);
		//確認IPv4 header的protocol是否有icmp
		if(ip_header->protocol==IPPROTO_ICMP){
		bpf_printk("I Get The ICMP\n");
		}
	} 
    return XDP_PASS;
}
char _license[] SEC("license") = "GPL";
```
##### 編譯bpf
{{< alert theme="info" >}}
利用`clang`編譯c語言 先加入include的路徑`-isystem` 為`/usr/include/x86_64-linux-gnu` 並且優化等級為`-O2` 編譯為(`-target`) `bpf`架構的執行檔 只處理到編譯尚未處理連結(`-c`) 的檔名`xdp_show_icmp.c` 輸出(`-o`) 檔名`xdp_show_icmp.o`
{{< /alert >}}

 
```
clang -isystem /usr/include/x86_64-linux-gnu -O2 -target bpf -c xdp_show_icmp.c -o xdp_show_icmp.o
```

##### 觀察架構
 

觀察此elf的指令架構
```
readelf -h xdp_show_icmp.o
```

##### 設定XDP

{{< alert theme="info" >}}
需要root權限`sudo` 使用`ip`工具 修改網卡(`link`) 設定(`set`) 網卡(`dev`) 名稱為`lo` eneric模式(`xdpgeneric`) 從object file(`obj`) 檔名為`xdp_show_icmp.o` 的sections(`sec`) 名稱為`show_icmp`取出我們要的程式塞入核心
{{< /alert >}}

```
sudo ip link set dev lo xdpgeneric obj xdp_show_icmp.o sec show_icmp
```

##### 開始實驗

開啟終端機1
```
sudo cat /sys/kernel/debug/tracing/trace_pipe
```

開啟終端機2
```
ping 127.0.0.1
```

### e_version

`e_version`與`EI_VERSION`都是固定1
```
The value 1 signifies the original file format; extensions will create new versions with higher numbers. The value of EV_CURRENT changes as necessary to reflect the current version number.
```
### e_entry

只有`e_type`為`ET_EXEC`的執行檔他的`e_entry`才值得參考
`e_type`為`ET_EXEC`的執行檔載入的記憶體位置會變動
#### 實驗e_entry

先從[ET_EXEC](#et_exec)準備好環境

找一下`Entry point address`的數值並且確認`e_type`為`ET_EXEC`
```
readelf -h a.out
```
 
進入gdb
```
gdb a.out
```

中斷在`Entry point address`
```
b *0x1234
```
開始執行
```
r
```

{{< alert theme="info" >}}
觀察是不是中斷在`_start()`? 代表`elf`執行檔`e_type`為`ET_EXEC`的`Entry point address`是程式的進入點
{{< /alert >}}

### e_phoff
紀錄`program header table`位置在哪,如果沒有就`0`
 
### e_shoff
紀錄`section header table`位置在哪,如果沒有就`0`
### e_flags

`processor-specific`這裡定義硬體的特徵,細節的定義在不同的指令架構的規格書裡面,可以參考[RISC-V ELF psABI specification][https://github.com/riscv/riscv-elf-psabi-doc/blob/master/riscv-elf.md#-file-header]裡面關於elf的規範


### e_ehsize

ELF header有多少bytes.

### e_phentsize

每一個program header大小有多少bytes

### e_phnum

有多少個program header,如果沒有就為0

### e_shentsize

每一個section header大小有多少bytes
 
### e_shnum
有多少個section header,如果沒有就為0

### e_shstrndx

`e_shstrndx`的數值代表,第幾個section存放`.shstrtab`這個section,`.shstrtab`很特殊負責存放所有section`人看的懂的名稱`

[e_shstrndx介紹](https://paper.seebug.org/89/)

#### 實驗看看

準備一個elf檔案

觀察`Section header string table index`數值(`e_shstrndx`)
```
readelf -h a.out
```

確認`.shstrtab`的位置是不是與`e_shstrndx`標示的一樣
```
readelf -S a.out
```

直接看`.shstrtab`內容,可以發現裡存放很多字串
```
readelf -x .shstrtab a.out
```



## Program Headers

利用指令查看
```bash
readelf -l 你的檔名
```

只有`executable`與`shared object files`才有特別的用途
> [Program headers are meaningful only for executable and shared object files](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.pheader.html)

{{<notice warning>}}
名詞注意:一個`segment`是由一個到多個`section`組成
{{</notice>}}


一個elf有0或多個`Program Header`,每個`Program Header`專門處理一個`segment`載入記憶體的規劃,如果你的elf檔案不是拿來載入記憶體執行用途可能就沒有這個header,如下範例
{{<notice info>}}
利用`readelf -l 你的檔名`指令觀察`e_type` 為`REL (Relocatable file)`的elf有沒有program headers
{{</notice>}}

 
每一個`Program Header`的結構如下圖
```c
typedef struct elf64_phdr {
  Elf64_Word p_type;
  Elf64_Word p_flags;
  Elf64_Off p_offset;		/* Segment file offset */
  Elf64_Addr p_vaddr;		/* Segment virtual address */
  Elf64_Addr p_paddr;		/* Segment physical address */
  Elf64_Xword p_filesz;		/* Segment size in file */
  Elf64_Xword p_memsz;		/* Segment size in memory */
  Elf64_Xword p_align;		/* Segment alignment, file & memory */
} E
```


### p_type
|		p_type	|		Value			|	用途		|
| --------- | -------------------- |-------|
PT_NULL	|0 |
PT_LOAD	|1|
PT_DYNAMIC|	2|
PT_INTERP	|3| 當程式需要動態連結 就需要`PT_INTERP`型態的`segment` 並且裡面包含`dynamic linker/loader`的路徑
PT_NOTE	|4|
PT_SHLIB	|5|
PT_PHDR|	6|
PT_TLS|	7|
PT_LOOS	|0x60000000| 從0x60000000~0x6fffffff的數值保留給作業系統使用
PT_HIOS	|0x6fffffff| 同上
PT_LOPROC|	0x70000000|從0x70000000~0x7fffffff的數值保留給處理器使用
PT_HIPROC|	0x7fffffff|同上

#### PT_INTERP實驗

1.  觀察Program Header `PT_INTERP`型態的segment
	
	```
	readelf -l a.out
	```


2.  編譯此[範例](../../reverse/gdb/#有趣的程式)程式再觀察一次,可以發現`PT_INTERP`型態的segment不見了,go語言也是屬於這種不需要動態連結的程式,好處是有很好的可攜性,缺點是執行檔比較大
	
3.  修改ld-linux(`dynamic linker `)名稱會發生什麼事?
	{{<notice warning>}}
此實驗會毀損linux執行的過程謹慎練習
實驗中我的`Requesting program interpreter`是`/lib64/ld-linux-x86-64.so.2`
如果我把`/lib64/ld-linux-x86-64.so.2`名稱改掉會怎樣?
是不是只有實驗2的程式才能跑?
	{{</notice>}}



### p_flags
### p_offset
### p_vaddr
### p_paddr
### p_filesz
### p_memsz
### p_align
 



## Section Headers

利用指令查看
```bash
readelf -S 你的檔名
```

```c
typedef struct elf64_shdr {
  Elf64_Word sh_name;		/* Section name, index in string tbl */
  Elf64_Word sh_type;		/* Type of section */
  Elf64_Xword sh_flags;		/* Miscellaneous section attributes */
  Elf64_Addr sh_addr;		/* Section virtual addr at execution */
  Elf64_Off sh_offset;		/* Section file offset */
  Elf64_Xword sh_size;		/* Size of section in bytes */
  Elf64_Word sh_link;		/* Index of another section */
  Elf64_Word sh_info;		/* Additional section information */
  Elf64_Xword sh_addralign;	/* Section alignment */
  Elf64_Xword sh_entsize;	/* Entry size if section holds table */
} Elf64_Shdr;
```

### sh_name
### sh_type

#### 
### sh_flags
### sh_addr
### sh_offset
### sh_size
### sh_link
### sh_info
### sh_addralign
### sh_entsize







## 進階探討ELF
https://web.archive.org/web/20210415124218/http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html


## 參考

 
[操作系统真象还原-5.3.3 elf 格式的二进制文件]  

[The Curious Case of Position Independent Executables](https://web.archive.org/web/20171120151138/https://eklitzke.org/position-independent-executables)


[^app]:[APP漏洞扫描器之未使用地址空间随机化](https://web.archive.org/web/20201130173004/https://developer.aliyun.com/article/65363)

[Learning Linux Binary Analysis]

[ELF之學習心得02 - ELF Header(e_ident篇)](https://web.archive.org/web/20190916092435/http://nano-chicken.blogspot.com/2011/07/elf02-elf-header.html)

[ELF Header](https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.eheader.html)