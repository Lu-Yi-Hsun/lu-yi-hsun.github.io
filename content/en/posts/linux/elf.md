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
- [Oracle Solaris 11.1 Linkers and Libraries Guide ](https://docs.oracle.com/cd/E26502_01/html/E26507/chapter6-43405.html)
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

這種類型的執行檔載入到固定虛擬記憶體位置,所以e_entry的位置就是固定的位置

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
這種類型的執行檔才可以被作業系隨機載入到不同虛擬記憶體位置
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
 


### e_machine

這個欄位是可以知道此elf檔案是屬於哪個架構的

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
`e_type`為`ET_DYN`的執行檔載入的虛擬記憶體位置會變動
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

`processor-specific`這裡定義硬體的特徵,細節的定義在不同的指令架構的規格書裡面,如果是`RISC-V`可以參考[RISC-V ELF psABI specification][https://github.com/riscv/riscv-elf-psabi-doc/blob/master/riscv-elf.md#-file-header]裡面關於elf的規範


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

{{<notice warning>}}
名詞注意:一個`segment`是由一個到多個`section`組成
{{</notice>}}


利用指令查看
```bash
readelf -l 你的檔名
```

只有`executable`與`shared object files`的`Program Headers`才有特別的用途
> [Program headers are meaningful only for executable and shared object files](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.pheader.html)

 
一個elf有0或多個`Program Header`,每個`Program Header`專門處理一個`segment`載入記憶體的規劃,如果你的`elf檔案`或是某個`section`不是拿來載入記憶體執行用途可能就沒有這個header,如下範例
{{<notice info>}}
先準備好[ET_REL](#et_rel)的執行檔案
利用`readelf -l function.o`指令觀察`e_type` 為`REL (Relocatable file)`的elf有沒有program headers?
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

`Program Header`的型態

|		p_type	|		Value			|	用途		|
| --------- | -------------------- |-------|
PT_NULL	|0 | 沒有在使用, `program header table`會忽略`PT_NULL`型態的`segment`,FIXME: 細節還需要在研究.
PT_LOAD	|1| 	單純載入`segment`到虛擬記憶體,如果載入到記憶體(`p_memsz`)時比在檔案(`p_filesz`)的時候還大,多出來的地方補0,可能是為了對齊記憶體
PT_DYNAMIC|	2|  載入這個`segment`裡面都是`Dynamic Section`去虛擬記憶體
PT_INTERP	|3| 當程式需要動態連結 就需要載入這個`PT_INTERP`型態的`segment`去記憶體 並且裡面包含`dynamic linker/loader`的路徑
PT_NOTE	|4|  載入這個`segment`裡面都是`Note Section`去虛擬記憶體
PT_SHLIB	|5| 這個保留尚未定義
PT_PHDR|	6| 這個型態代表載入整個`Program Header Table`去虛擬記憶體,裡面有1個以上`Program Header`
PT_TLS|	7| 載入記憶體是以`PT_TLS`型態,這個`segment`是存放[`Thread-local storage`](https://selfboot.cn/2016/08/22/threadlocal_overview/)變數初始數值
PT_LOOS	|0x60000000| 從0x60000000~0x6fffffff的數值保留給作業系統使用
PT_HIOS	|0x6fffffff| 同上
PT_LOPROC|	0x70000000|從0x70000000~0x7fffffff的數值保留給處理器使用
PT_HIPROC|	0x7fffffff|同上

#### PT_INTERP實驗

1.  觀察Program Header `PT_INTERP`型態的segment
	
	```
	readelf -l a.out
	```


2.  編譯此[範例](../../reverse/gdb/#有趣的程式)程式再觀察一次,可以發現`PT_INTERP`型態的segment不見了,go語言也是屬於這種不需要動態連結的程式,好處是有很好的可攜性,缺點是執行檔比較大,並且`e_type`屬於`ET_EXEC`代表每次都載入到固定虛擬記憶體可能有安全疑慮
	
3.  修改ld-linux(`dynamic linker `)名稱會發生什麼事?
	{{<notice warning>}}
此實驗會毀損linux執行的過程謹慎練習
實驗中我的`Requesting program interpreter`是`/lib64/ld-linux-x86-64.so.2`
如果我把`/lib64/ld-linux-x86-64.so.2`名稱改掉會怎樣?
是不是只有實驗2的程式才能跑?
	{{</notice>}}

#### PT_TLS實驗

參考[Linux Thread Local Storage](http://weng-blog.com/2016/07/Linux-tls/)

如下範例可以知道`TLS`(Thread Local Storage)的作法

每開起一個線程會從`TLS_data`(elf會定義這個變數存放的segment載入記憶體時`p_type`為`PT_TLS`)拿出變數的初始數值存放每個線程`內部`的`TLS_data`並且每個函數都可以直接使用`TLS_data`.

好處可以看到`TLS`版本不用傳遞參數給函數就可以直接使用,


{{< codes NO_TLS TLS >}}
  {{< code >}}
{{< highlight c "linenos=table,hl_lines=5 11,linenostart=1" >}}
#define  NUMTHREADS   2 
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
void show_tls(int local_data) {
    printf("線程 %.16lx: TLS data=%d\n",pthread_self(), local_data);
}
void *thread_run(int i)
{
   int local_data = i;
   show_tls(local_data);
   return NULL;
}
int main(int argc, char **argv)
{
  pthread_t             thread[NUMTHREADS];
  int                   rc=0;
  printf("開始執行\n");
  for (int i=0; i < NUMTHREADS; i++) { 
     rc = pthread_create(&thread[i], NULL, &thread_run, i);
  }
  printf("執行中..\n");
  for (int i=0; i < NUMTHREADS; i++) {
     rc = pthread_join(thread[i], NULL);  
  }
  printf("執行完成\n");
  return 0;
}
{{< / highlight >}}
  {{< /code >}}

  {{< code >}}


{{< highlight c "linenos=table,hl_lines=6 12,linenostart=1" >}}
#define  NUMTHREADS   2 
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
__thread int TLS_data=87;
void show_tls() {
    printf("線程 %.16lx: TLS data=%d\n",pthread_self(), TLS_data);
}
void *thread_run(int i)
{
   TLS_data = i;
   show_tls();
   return NULL;
}
int main(int argc, char **argv)
{
  pthread_t             thread[NUMTHREADS];
  int                   rc=0;
  printf("開始執行\n");
  for (int i=0; i < NUMTHREADS; i++) { 
     rc = pthread_create(&thread[i], NULL, &thread_run, i);
  }
  printf("執行中..\n");
  for (int i=0; i < NUMTHREADS; i++) {
     rc = pthread_join(thread[i], NULL);  
  }
  printf("執行完成\n");
  return 0;
}
{{< / highlight >}}
  
  {{< /code >}}
{{< /codes >}}

編譯
```
gcc t.c -lpthread
```


 
 
### p_flags

定義這個區塊的`segment`在記憶體的權限 最後3個bit定義 讀/寫/執行

舉例
可讀/不可寫/可執行
0x00000000000000000000000000000101(32bit)

可讀/可寫/可執行
0x00000000000000000000000000000111(32bit)

### p_offset

定義這個`segment`在檔案哪個位置開始

### p_vaddr

定義這個`segment`載入到虛擬記憶體的哪個位置,只有沒有經過ASLR(Address space layout randomization)載入的執行檔(`e_type`為`ET_EXEC`的執行檔),這個數值才有參考價值 

### p_paddr
定義這個`segment`載入到實體記憶體的哪個位置,因為實體記憶體被作業系統保護所以這個數值不準確.
### p_filesz
定義這個`segment`在檔案的大小多少bytes,可為0
### p_memsz
定義這個`segment`在虛擬記憶體的大小多少bytes,可為0
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

`sh_name`存放名稱的index

section的名稱字串存在`.shstrtab`

### sh_type

Name|	Value|用途
|----|------|-----|
SHT_NULL|	0| 這個型態代表這個section header無效
SHT_PROGBITS	|1|這裡面的section放程式執行用到的東西
SHT_SYMTAB|	2| 這裡面的section放symbol table FIXME [Symbol Table](https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.symtab.html)
SHT_STRTAB|	3| 這裡面的section放滿人可以看得懂的字串,例如可以拿來表示section的名稱,
SHT_RELA|	4| 給object file重定位資訊用的(with explicit addends) FIXME:[Relocation](https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.reloc.html) 
SHT_HASH|	5|這個section給symbol table使用 
SHT_DYNAMIC	|6| 這裡放動態連結的訊息  FIXME:[Dynamic Section](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.dynamic.html#dynamic_section)
SHT_NOTE	|7| 這裡檔案的訊息  FIXME:[Note Section](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.pheader.html#note_section)
SHT_NOBITS|	8|代表這個section在檔案裡面沒有佔用任何大小,它只有section header而已
SHT_REL|	9|給object file重定位資訊用的(without explicit addends) FIXME:[Relocation](https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.reloc.html) 
SHT_SHLIB	|10|這個保留尚未定義
SHT_DYNSYM|	11| 這裡面的section放symbol table
SHT_INIT_ARRAY	|14|裡面有陣列記憶體位置,指向主程式開始前的函數FIXME:[Initialization and Termination Functions](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.dynamic.html#init_fini)
SHT_FINI_ARRAY	|15|裡面有陣列記憶體位置,指向主程式結束後要執行的函數,`atexit` FIXME:[Initialization and Termination Functions](https://refspecs.linuxfoundation.org/elf/gabi4+/ch5.dynamic.html#init_fini)
SHT_PREINIT_ARRAY	|16|裡面有陣列記憶體位置指向函數,這個函數會在`SHT_INIT_ARRAY`裡面所有函數執行之前執行
SHT_GROUP	|17|
SHT_SYMTAB_SHNDX	|18|
SHT_LOOS	|0x60000000|從0x60000000~0x6fffffff的數值保留給作業系統使用
SHT_HIOS	|0x6fffffff|同上
SHT_LOPROC|	0x70000000|從0x70000000~0x7fffffff的數值保留給處理器使用
SHT_HIPROC|	0x7fffffff|同上
SHT_LOUSER	|0x80000000|從0x80000000~0xffffffff的數值保留給應用程式使用
SHT_HIUSER|	0xffffffff|同上

#### Initialization and Termination Functions
尚未完成
這裡深入探討`SHT_PREINIT_ARRAY`,`SHT_FINI_ARRAY`,`SHT_INIT_ARRAY`


### sh_flags

 
Name|	Value|用途
|----|------|-----|
SHF_WRITE|	0x1|程式執行時這個section可寫
SHF_ALLOC|	0x2|程式執行時這個section會佔用記憶體
SHF_EXECINSTR|	0x4|這個section包含機械碼
SHF_MERGE|	0x10|
SHF_STRINGS|	0x20|包含以零結尾的字符串組成
SHF_INFO_LINK|	0x40|
SHF_LINK_ORDER|	0x80|
SHF_OS_NONCONFORMING|	0x100|
SHF_GROUP|	0x200|
SHF_TLS|	0x400|
SHF_MASKOS|	0x0ff00000|保留給作業系統使用
SHF_MASKPROC|	0xf0000000|保留給處理器使用



### sh_addr

此section在執行時的記憶體哪個位置
0代表沒有在記憶體中

### sh_offset

此section在檔案時哪個位置

### sh_size
代表section在檔案的大小
{{<notice warning>}}
如果`sh_type`為`SHT_NOBITS`,雖然檔案沒有存放,但是sh_size可能不為0
舉例
c語言還沒初始化的變數放在`.bss section`屬於`SHT_NOBITS`在檔案沒佔空間(只有section header)但載入記憶體後要佔空間

{{</notice>}}

### sh_link
### sh_info
### sh_addralign

此section要對齊記憶體
必須讓sh_addr數值剛好,sh_addr/sh_addralign整除才能對齊
記憶體對齊可以增加效能,或是對齊cache避免[False_sharing](https://en.wikipedia.org/wiki/False_sharing)

### sh_entsize


## Special Sections

這個部份負責探討特殊Sections在幹麻
這個部份尚未完成

 
### Relocation

重定位,用在靜態連結動態連結

現在流行可攜性高的程式很多程式golange



|重定位方式|效能|檔案大小
|----|------|-----|----|
|靜態連結|快|大|
|動態連結|中|||
|動態載入||
#### Types


關於Relocation Types (Processor-Specific)是屬於每個指令架構abi所定義,而我們目前只討論AMD64的細節,文件[System V Application Binary Interface AMD64 Architecture Processor Supplement](https://refspecs.linuxfoundation.org/elf/x86_64-abi-0.99.pdf)有定義elf-Processor-Specific的規範

 
 procedure linkage table



## 進階練習

學完elf後,如何從這個[範例](../../reverse/gdb/#有趣的程式)刪減elf執行檔的大小又可以執行?

可以作答在最下面的討論區

可以參考[A Whirlwind Tutorial on Creating Really Teensy ELF Executables for Linux](https://web.archive.org/web/20210415124218/http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html)

[lief工具修改](https://lief.quarkslab.com/)

## 參考

 
[操作系统真象还原-5.3.3 elf 格式的二进制文件]  
[Learning Linux Binary Analysis]
[The Curious Case of Position Independent Executables](https://web.archive.org/web/20171120151138/https://eklitzke.org/position-independent-executables)
[APP漏洞扫描器之未使用地址空间随机化](https://web.archive.org/web/20201130173004/https://developer.aliyun.com/article/65363)
[ELF之學習心得02 - ELF Header(e_ident篇)](https://web.archive.org/web/20190916092435/http://nano-chicken.blogspot.com/2011/07/elf02-elf-header.html)

[ELF Header](https://refspecs.linuxfoundation.org/elf/gabi4+/ch4.eheader.html)