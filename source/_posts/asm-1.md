---
title: Assembly part 2.
date: 2024-08-15 01:24:08

tags:
    - csie
    - asm
---

## Basic Instructions

```ams=
; Program to show use of interrupts
; Also, Hello World program !
hello: DB "Hello World" ; store string

; actual entry point of the program, must be present
start:
MOV AH, 0x13            ; move BIOS interrupt number in AH
MOV CX, 11              ; move length of string in cx
MOV BX, 0               ; mov 0 to bx, so we can move it to es
MOV ES, BX              ; move segment start of string to es, 0
MOV BP, OFFSET hello    ; move start offset of string in bp
MOV DL, 0               ; start writing from col 0
int 0x10                ; BIOS interrupt
```

### CMP

#### CMPX1,X2  `JMP`

- **JE (Jump if Equal):** Jumps if the Zero Flag (ZF) is set (`ZF = 1`).
- **JNE (Jump if Not Equal):** Jumps if the Zero Flag (ZF) is not set (`ZF = 0`).
- **JZ (Jump if Zero):** Jumps if the Zero Flag (ZF) is set (`ZF = 1`).
- **JNZ (Jump if Not Zero):** Jumps if the Zero Flag (ZF) is not set (`ZF = 0`).
- **JS (Jump if Sign):** Jumps if the Sign Flag (SF) is set (`SF = 1`).
- **JNS (Jump if Not Sign):** Jumps if the Sign Flag (SF) is not set (`SF = 0`).
- **JC (Jump if Carry):** Jumps if the Carry Flag (CF) is set (`CF = 1`).
- **JNC (Jump if Not Carry):** Jumps if the Carry Flag (CF) is not set (`CF = 0`).
- **JO (Jump if Overflow):** Jumps if the Overflow Flag (OF) is set (`OF = 1`).
- **JNO (Jump if Not Overflow):** Jumps if the Overflow Flag (OF) is not set (`OF = 0`).
- **JA (Jump if Above):** Jumps if (`CF = 0` and `ZF = 0`).
- **JAE (Jump if Above or Equal):** Jumps if (`CF = 0`).
- **JNA (Jump if Not Above):** Jumps if (`CF = 1` and `ZF = 1`).
- **JNAE (Jump if Not Above or Equal):** Jumps if (`CF = 1`).
- **JG (Jump if Greater):** Jumps if (`ZF = 0` and `SF = OF`).
- **JGE (Jump if Greater or Equal):** Jumps if (`SF = OF`).
- **JNG (Jump if Not Greater):** Jumps if (`ZF = 1` or `SF != OF`).
- **JNGE (Jump if Not Greater or Equal):** Jumps if (`SF != OF`).
- **JB (Jump if Below):** Jumps if (`CF = 1`).
- **JBE (Jump if Below or Equal):** Jumps if (`CF = 1` or `ZF = 1`).
- **JNB (Jump if Not Below):** Jumps if (`CF = 0`).
- **JNBE (Jump if Not Below or Equal):** Jumps if (`CF = 0` and `ZF = 0`).
- **JL (Jump if Less):** Jumps if (`SF != OF`).
- **JLE (Jump if Less or Equal):** Jumps if (`SF != OF` or `ZF = 1`).
- **JNL (Jump if Not Less):** Jumps if (`SF = OF`).
- **JNLE (Jump if Not Less or Equal):** Jumps if (`SF = OF` and `ZF = 0`).
- **JP/JPE (Jump if Parity/Even Parity):** Jumps if Parity Flag (PF) is set (`PF = 1`).
- **JNP/JPO (Jump if Not Parity/Odd Parity):** Jumps if Parity Flag (PF) is not set (`PF = 0`).

### Integer Addition and Subtraction

```asm=
start:
mov al, 5
mov cl, 2
add al,cl
```

```asm=
start:
mov dx, 0
mov ax, 0083
mov bx, 2
div bx
```

```asm=
start:
mov ah, 255
mov bh, 254
xor ah, bh
```

- Operation List

```asm=
ADD X1,X2    ;將X1 X2的數字相加並存入X1中
SUB X1,X2    ;同 上 ，只 是 變 減 法 MUL BXAX去乘BX
INC X1     ;X1的數字加1 ，並放回X1 D E C X 1 同 上 ，但 變 減 1
NEG X1     ;將X1的數字變為2的補數
OR X1,X2     ;X1,X2 去做OR邏輯運算，並將結果放回X1
AND X1,X2     ;同上只是去做AND邏輯運算
TEST X1,X2     ;同上，他是去做AND邏輯運算但比較特殊的是他不會把結果存入X1中
END     ;表示程式結束處 DB[字串，數值]去定義一個位元組的資料不影響旗標的程式=〉
            ;移動資料的程式(註記碼) like: MOV AX,BX/POP AX/PUSH AX
```

## Exam

[EM1](https://hackmd.io/54U7j1vaR7ebbLQtVb4FNQ)
[EM2](https:hackmd.io/cVT7YTEgRw22Vab8npsRqA)
