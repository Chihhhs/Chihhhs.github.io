---
title: Yuawn bof WriteUp
date: 2024-08-05 13:49:34
category: PWN
tags: pwn
---


>From `yuawn` [NTU-Computer-Security](https://github.com/yuawn/NTU-Computer-Security)

- [Dkoctro](https://hackmd.io/nFewizI_T7GcNPKGDwVKTQ)

- [x] week1
- [ ] week2
- [ ] week3
- [ ] PicoCTF.com
- [ ] [pwn.college](https://pwn.college/)
- [ ] Pwnable.tw

## Basic

### ELF (Executable and Linkable Format)

**INTRO**
![image](https://hackmd.io/_uploads/HJ-AkIhHp.png )

**ELF-workflow (static)**
![image](https://hackmd.io/_uploads/B1bbxL3BT.png )

**ELF-workflow (static)**
![image](https://hackmd.io/_uploads/ByEfeI2rT.png )

**ElF-section**
![image](https://hackmd.io/_uploads/B1WKkIhH6.png )

**ELF-Protections**
![image](https://hackmd.io/_uploads/BJdLlU3HT.png )

### ELF DEMO

#### readelf

- Install [peda](https://github.com/longld/peda)

```elf.c
#include<stdio.h>
#include<stdlib.h>
int a;
int b = 100;

int main(){
    int c;
    puts("I love pwning.");
    return 0;
}

```

```makefile=
all: elf.c
        gcc elf.c -o elf -no-pie
```

- readelf `offset`

```bash=
$readelf -S elf

        |addr                    | offset |
| .data |0x0000000000404008      | 3008   |
| .bss  |0x00000000040401c       | 301c   |

$ (gdb) x/30 0x0000000000404008
    <b>  0x0000000000000064

$xxd elf
```

- readelf
![image](https://hackmd.io/_uploads/Bkr45M0Hp.png )
- xxd
![image](https://hackmd.io/_uploads/S1RGOzCra.png )

#### Check file size

```cpp=
#include<stdio.h>
#include<stdlib.h>
int a;
int b = 100;
char buf[0x400];

int main(){
        int c;
        puts("I love pwning.");
        return 0;
}
```

- size
![image](https://hackmd.io/_uploads/B12E0fCB6.png)

```cpp=
#include<stdio.h>
#include<stdlib.h>
int a;
int b = 100;
char buffer[0x400*0x400*10]={'A'};

int main(){
        int c;
        puts("I love pwning.");
        return 0;
}
```

- File size: 11M
![image](https://hackmd.io/_uploads/H1Ur0GCHa.png)

#### Offset and address

- `Address（地址）`： 這是指 ELF 檔案在記憶體中的加載位置。執行 ELF 時，作業系統將檔案的不同 section 加載 (mapping) 到記憶體的不同位置。每個 section 的 "Address" 就是在記憶體中的實際位置。
- `Offset（偏移量）`： 指 ELF 中 section 在檔案本身的偏移位置。每個 section 的 "Offset" 就是指該 section 在 ELF 檔案中的位置，以字節為單位。

>"Address" 是加載到記憶體時的位置，而 "Offset" 是在檔案中的位置。
>這兩者之間的關聯可以通過 ELF 檔案的 **section 表格**進行理解。

## x86_64

- **8 byte alignment (對齊)**
- Stack `0x10` bytes

### Assembly Registers

![image](https://hackmd.io/_uploads/HJD7d0-Dp.png )

### Special Registers

![image](https://hackmd.io/_uploads/B1yrdC-wa.png )

### Instruction

![image](https://hackmd.io/_uploads/SyBRu0-Pa.png )

![image](https://hackmd.io/_uploads/B1nkF0-D6.png )

### x64 calling convention

![image](https://hackmd.io/_uploads/HJjXtRbPT.png)

## Stack Frame

### Stack Struct

![image](https://hackmd.io/_uploads/B1O2qC-wa.png)

往低地址找

- `push` : rsp-0x8
- `pop` : rsp+0x8

## Function Prologue & Epilogue

### Prologue

![image](https://hackmd.io/_uploads/Byt_YyE96.png)

- **call func** = `push next-rip` `jmp func`
- `mov eax,0` `push` **func allcation** in RAM
- **rip ->** `func : push rbp` `##save rbp`
- `mov rbp,rsp`
![image](https://hackmd.io/_uploads/H16_6k4cT.png )
- `sub rsp,0x70` **0x70 compiler 決定 >>** `To store local variable`
![image](https://hackmd.io/_uploads/HJckngNcT.png)
- **Prologue finish**

### Epilogue

![image](https://hackmd.io/_uploads/ry1KMlNc6.png )

- **leave** = `mov rsp,rbp` `pop rbp`
- `pop rbp` **-> get** `saved rbp` **segment.**
- **ret** = `pop rip`
- **epilogue finish**
- `rip` **Back to next**
![image](https://hackmd.io/_uploads/rJEyce4q6.png )

<br>
<br>

Advanced
==

## Overflow


+ Buffer overflow
+ Stack overflow
+ Heap overflow


### Buffer Overflow

`Hijack return address , control rip.`

+ `bof.c`
```c bof.c
#include<stdio.h>
#include<stdlib.h>

void call_me(){
    system('sh');
}

int main(){
    char buf[0x10];
    gets(buf);
    return 0;
}
```

1. 輸入超過 0x10 bit，將覆蓋堆疊中 `main function` 的 return address。
3. 當返回時，將堆疊的值放入 rip -> **Illegal virtual addr**
4. 引發 **segmentation fault** -> 堆疊崩潰

>**蓋 0x10 bit 會到 `rbp` 所以要加 8 bit覆蓋rbp 之後才填入要去的address (使用pwntools `p64()`)**

+ `gets(buf)`: **danger function**
+ Will print out secrct key or other info
+ `Control rip` -> `pwned`

<br>


## Linux syscall


>和 kernal 溝通的 interface

+ instruction - syscall
+ **rax** - syscall number
+ Arguments - **rdi, rsi, rdx, r10, r8 ,r9**
+ return value - **rax**
+ Ex: `read(0,buf,0x100)`

```asm=
xor rdi, rdi       ; 將 rdi 寄存器清零，用作文件描述符
mov rsi, 0x60100   ; 將 rsi 寄存器設置為緩衝區的地址（0x60100）
mov rdx, 0x100     ; 將 rdx 寄存器設置為要讀取的字節數（0x100，256 字節）
mov eax, 0         ; 將 eax 寄存器設置為系統調用編號，0 表示 read
syscall            ; 執行系統調用
```
**REF**: [Linux 核心設計: System call](/@RinHizakura/S1wfy6nQO)



## Shell Code


+ Assembler
+ asm -> machine code
+ `pwntools.shellcraft`
+ `i386` , `amd64`

### x86 shellcraft

+ `shellcraft.open(b'/home/orw/flag')`：
    + 參數：文件路徑的字串表示（例如 b'/home/orw/flag'）。
    + 意義：要打開的文件的路徑。
+ `shellcraft.read('eax','esp',50)`：
    * 參數：read 系統調用的三個參數，分別是文件描述符、緩衝區地址和要讀取的字節數。
    * 意義：
        * `eax`：作為文件描述符（通常是 open 系統調用的返回值）。
        * `esp`：作為緩衝區地址，這裡是使用堆疊頂部作為緩衝區地址。
        * `50`：要讀取的字節數，這裡是讀取 50 個字節。
* `shellcraft.write('1','esp',50)`：
    * 參數：write 系統調用的三個參數，分別是文件描述符、緩衝區地址和要寫入的字節數。
    * 意義：
        * `1`：作為文件描述符，表示寫入到文件描述符為 1 的文件中（通常是**標準輸出**）。
        * `esp`：作為緩衝區地址，這裡是使用堆疊頂部作為緩衝區地址。
        * `50`：要寫入的字節數，這裡是寫入 50 個字節。

<br>

## Protector

> [Protector](https://ithelp.ithome.com.tw/articles/10227876)

### stack-protector

+ With protector
![image](https://hackmd.io/_uploads/ryWOmUF5T.png )
+ **If overflow**
![image](https://hackmd.io/_uploads/Hkm_8IF5a.png )

### Canary

![image](https://hackmd.io/_uploads/rky6Qx_cp.png )

>編譯器會將一個特殊的值（canary value）插入到函式的堆疊框架中，通常是在函式的返回地址之前。當函式執行完成時，編譯器會檢查這個 canary value 是否被修改。如果 canary value 被修改，則認為發生了**堆疊溢位**，可能是一次攻擊，因此程式執行會被**中斷**或採取其他相應的安全措施。

### NX

>Data segment no execute

+ stack,heap
+ rw-

>Code segment
+ r-x

### ASLR
>Address Space Layout Randomization

+ kernel
+ 每次載入時，base 都是隨機的

`lld ./orw`
>`List Dynamic Dependencies`
>當運行 `ldd` 後跟著執行檔或共享庫的路徑時，它將輸出指定文件所依賴的**共享庫**和**路徑**列表。

### PIE

>Position-Independent Executable
> `PIC` : Position-Independent Code 在動態連結庫中重新定位符號
+  ELF code 和 data sections mapping 到 virtual addresses 的ASLR. 
+ Changing code base  every time ,or 0x40000
+ Record in ELF file
+ 可以從 ELF header 去掉


### Lazy Binding

> [深入理解動態連結](https://ithelp.ithome.com.tw/m/articles/10268401)

![image](https://hackmd.io/_uploads/HJ15ez5hT.png)

>動態庫解析函數地址的策略。
>函數地址在**第一次調用**時才被解析，而不是在程式加載時解析。
提高性能和減少內存使用。

>總結：**當真正調用時，才會去載入 Function**

<br>

## GOT & PLT

> [REF](https://hackmd.io/@rhythm/ry5pxN6NI)

### GOT

> `Global Offset Table`

![image](https://hackmd.io/_uploads/HJOt8M5h6.png)

+ 存儲**動態連結庫中全局變量和函數**的地址，當程式需要存取這些全局變量或函數時，通過GOT中存儲的地址來進行訪問。
+ 解析全局變量和函數的地址，從而實現**動態連結**，**程式共享**
+ `替換GOT地址` inject

### PLT (過程連結表)
> `Procedural Linkage Table`

第一次調用一個函數時，會調用形如 `function@PLT` 的函數。跳轉到函數對應的PLT表開頭執行，解析出函數真正的地址**填入GOT表**。再調用時，會直接從GOT表中取出函數起始地址執行。

### GOT & PLT 呼叫步驟

1. 呼叫 `printf()` 函式時，在組合語言中會看到 call `printf@plt`。
2. printf@GOT 會從 .got.plt 中取得函式的地址。
    >`If function == FirstCall :` 在 GOT 表中找不到地址 -> 透過 PLT 進行定位。
4. 將函式所需的參數推入堆疊中
5. 通過執行 `dl_runtime_resolve` 找出函式位址。
6. 最後，系統會將函式的位址寫入 **.got.plt** 中，等待下次呼叫。

>Attack: [`Ret2dlresolve`](https://b0ldfrev.gitbook.io/note/pwn/returntodlresolve-yuan-li-ji-li-yong)


### h3GOT Hijacking

+ 因為 Lazy binding，GOT為**可寫區域**
+ 一旦GOT被寫入覆蓋，下一次呼叫 `Library function` 時可以被劫持，從而控制即將執行的 `Function Pointer`

NEXT
==

[NTU-computer-security-pwn-2](https://hackmd.io/@KzcDuD/NTU-computer-security-pwn-2)

解題 & REF
==
>[Lab](/@KzcDuD/r1i7cBoaa)
[Pico](/@KzcDuD/Bk3DMy2tT)

>[Note](https://b0ldfrev.gitbook.io/note/pwn)

> [CheatSheet](/@u1f383/pwn-cheatsheet) from `@u1f383`