---
title: NTTU CSclube pwn part 1.
date: 2024-05-09 16:21:46
category: PWN
tags: 
    - pwn
    - ctf
---

> 資安社總務第二次上社課

## Prepare

1. VirtualMachine [install tutorial](https://hackmd.io/@SCIST/VirtualBox)
2. ubuntu [22.04 LTS](https://ubuntu.com/download/desktop)
3. gdb `sudo apt udate; apt install gdb`
4. [peda](https://github.com/longld/peda)
5. [lab](https://chihhhs.github.com/)

## x86_64 assembly

### Registers

+ RAX RBX RCX RDX RDI RSI - 64 bit
+ EAX EBX ECX EDX EDI ESI - 32 bit
+ AX BX CX DX DI SI - 16 bit
+ AX -> AH AL - 8 bit

+ RSP
+ RBP
+ RIP

### Instruction

+ jmp

```asm
# jmp a = 
mov rip,a
```

+ call

```asm
# call a =
push next_rip;
mov rip,a;
```

+ leave

```asm
mov rsp,rbp
pop rbp
```

+ ret

```asm
pop rip
```

## ELF

+ `.bss`： 未初始化全域變數
+ `.data`： 初始化全域變數
+ `.rodata`：Read only data
+ `.text`： 程式碼段

`readelf -S <elf>`

```c
int a;
int b=100;
int  main(){
    int c; 
    puts("30 cm"); 
    return 0;
}
```

+ Read offset

```bash
$readelf -S elf

        |addr                    | offset |
| .data |0x0000000000404008      | 3008   |
| .bss  |0x00000000040401c       | 301c   |

$ (gdb) x/30 0x0000000000404008
    <b>  0x0000000000000064

$xxd elf
```

## pwntools

> [pwntools](https://github.com/Gallopsled/pwntools)

## Function Prologue & Epilogue

### Prologue

1. call func = `push %next-rip;` `jmp func`
1. `mov $eax,0` `push func` allcation in RAM
1. %rip -> func : push rbp `%save-rbp`
1. mov $rbp, $rsp
1. sub %rsp,0x70 0x70 #compiler 決定 >> To store local variable
1. Prologue finish

### Epilogue

1. leave = `mov %rsp,%rbp;`  `pop %rbp;`
1. `pop %rbp` -> get `%saved-rbp` segment.
1. ret = `pop $rip`
1. %rip Back to `%next-rip`
1. Epilogue finish

## Buffer Overflow

> `Hijack return address , control rip.`

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

1. 輸入超過 0x18 bit，將覆蓋堆疊中 main function 的 return address。
2. 當返回時，將堆疊的值放入 rip -> Illegal virtual addr
3. 引發 segmentation fault -> 堆疊崩潰

> 蓋`0x10 bit` 會到 rbp 所以要加 8 bit覆蓋rbp 之後才填入要去的address (使用pwntools p64())

+ `gets(buf)`: **danger function**

### Linux SysCall

> 和 kernal 溝通的 `interface`

+ instruction -syscall
+ $rax -Syscall_number
+ Arguments - `(rdi, rsi, rdx, r10, r8 ,r9)`
+ return value - $rax
Ex: read(0,buf,0x100)

``` asm
xor rdi, rdi       ; 將 rdi 寄存器清零，用作文件描述符
mov rsi, 0x60100   ; 將 rsi 寄存器設置為緩衝區的地址（0x60100）
mov rdx, 0x100     ; 將 rdx 寄存器設置為要讀取的字節數（0x100，256 字節）
mov eax, 0         ; 將 eax 寄存器設置為系統調用編號，0 表示 read
syscall            ; 執行系統調用
```

## ShellCode

1. input **shellcode** (Syscall)
1. overflow to the address of shellcode

## docker-compose

```bash
$cd prectice
$docker compose up -d
```
