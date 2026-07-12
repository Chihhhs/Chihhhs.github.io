---
title: Assembly part 1.
date: 2024-08-15 01:19:59
tags: 
    - csie
    - asm
---

> Ref: [ASM tutorial](https://yassinebridi.github.io/asm-docs/asm_tutorial_01.html)

## X86 CPU Architecture

[X86-Registers and Addressing Modes.pdf](https://file.notion.so/f/f/cb883e80-075d-476a-b401-88c14fc6ecf4/f1ba7bfb-33c8-4cda-b60a-e2dd7eee6b5e/X86-Registers_and_Addressing_Modes.pdf?id=8e447b5d-09af-4fec-9996-c40a81ae1501&table=block&spaceId=cb883e80-075d-476a-b401-88c14fc6ecf4&expirationTimestamp=1698393600000&signature=ndENqs06GLxSo-Vs7JULZGqZueNbo3JzsQcyhQJv9gs&downloadName=X86-Registers+and+Addressing+Modes.pdf)

### Register

#### General Purpose Register

|暫存器| 累加器| 計數器| 資料| 基址|堆疊指標|棧基址指標|源變址|目標索引|
|---|---|---|---|---|---|---|---|---|
|64-bit|RAX|RCX|RDX|RBX|RSP|RBP|RSI|RDI|
|32-bit|EAX|ECX|EDX|EBX|ESP|EBP|ESI|EDI|
|16-bit|AX|CX|DX|BX|SP|BP|SI|DI|
|8-bit|AH  AL|CH   CL|DH   DL|BH  BL|||||

* AX：累加器(Accumulator)，用於算術和運算。
* BX：基址寄存器(Base)，通常用於指向數組或數據區域。
* CX：計數器(Counter)，通常用於迴圈計數。
* DX：數據寄存器(Data, IO)，用於 I/O 操作。

```plain text
4 個通用暫存器（AX、BX、CX、DX）由兩個獨立的 8 位元暫存器組成，
例如如果 AX= 00110000 00111001 b，則 AH= 00110000 b 且 AL= 00111001 b。
因此，當您修改任何 8 位元暫存器時，16 位元暫存器也會更新，反之亦然。
其他3個暫存器也是如此，「H」為高位，「L」為低位。
```

### Segment Register : 區段寄存器

|暫存器別名|暫存器|Size|
|--|--|--|
|程式區段暫存器|CS|16-bits|
|資料區段暫存器|DS|16-bits|
|額外區段暫存器|ES|16-bits|
|堆疊區段暫存器|SS|16-bits|

* CS：代碼段寄存器，存儲代碼段的偏移地址。
* DS：數據段寄存器，存儲數據段的偏移地址。
* SS：堆棧段寄存器，存儲堆棧段的偏移地址。
* ES：額外段寄存器，通常用於額外數據段的偏移地址。

* `邏輯位址→線性位址（實際位置）`，公式是：「Segment Register * 0x10 + Offset」

```plaintext=
段寄存器的特殊用途：段寄存器不同於通用寄存器，它們有一個特殊的用途，即指向可以存取的記憶體區塊。每個段寄存器都代表了一個記憶體段（memory segment），而不是單個數值。

與通用暫存器的結合：為了存取記憶體中的數據，你需要使用段寄存器與通用寄存器一起工作。例如，如果你想要訪問記憶體中的某個位置，比如物理位址12345h（十六進位），你應該將段寄存器DS設置為1230h，並將通用寄存器SI設置為0045h。這樣的組合可以用來計算實際的記憶體地址。

計算實體位址：8086處理器通過將段寄存器的值左移四位（乘以10h，也就是16），然後加上通用寄存器的值，來計算實際的記憶體位址。在上面的例子中，DS * 10h + SI 的結果將是12345h，這是你想要訪問的物理位址。
```

### Pointer Register : 指標暫存器

|暫存器別名|暫存器|預設的區段暫存器|Size (bits)|
|-|-|-|-|
|來源索引暫存器|ESI , SI||32 , 16|
|目的索引暫存器|EDI , DI||32 , 16|
|堆疊指標暫存器|ESP , SP|SS|32 , 16|
|基底指標暫存器|EBP , BP||32 , 16|
|程式指標暫存器|EIP , IP|CS|32 , 16|

#### Flag

![image](https://hackmd.io/_uploads/r13Oycwf6.png)

|Flag|Function|
|-|-|
|Zero flag 零時旗標 :zero: |set to 1 if result =0 ,set to 0 otherwise.|
|Sign flag 負號|set to 1 if result <0 ,set to 0 otherwise.|
|Carry flag|Carry out of a binary operation.|
|Parity flag|set to 1 if the number of 1's in a result is odd,set to 0 otherwise.|
|Overflow flag|set to 1 if there's a 2's came overflow|
|Half carry flag|Contains carry from the least sig Foour bits|
|Direction flag|STD set to 1 , CLD set to 0|

### Addressing modes

#### 暫存器定址法

* 用暫存器的位址當作指令的位址，指令的位址由暫存器的位址決定。

```asm
EX:
  MOV AX,BX
```

#### 立即定址法

+ 把資料放在指令中，不需要去讀取記憶體中的資料。

```asm
EX:
  MOV AX,111b
```

#### 直接定址法

* 直接給定資料的記憶體實際位址(物理位置)，直接讀取記憶體中的資料。

```asm
EX:
  MOV AX,[001H]
```

#### 間接定址法

* 指令的運算元欄內的值為有效位址的位址值，故需做二次的記憶體讀取，以取得所需之資料。(有點類似C++指標)

```asm
EX:
  MOV AX,[BX]
```

#### 基底定址法

* 使用基底暫存器(BX)加上位址偏移量(Offset)的方式來定址。

```asm
EX:
  MOV AX,[BX+0D5H]
```

#### 索引定址法

* 索引定址法是以固定的地址加上索引暫存器(SI,DI)的值，來得出位置。(通常用於陣列)

```asm
EX:
  MOV AX,[0000H+SI]
```

#### 基底索引定址法

* 基底索引定址法是以基底暫存器(BX)加上索引暫存器(SI,DI)的值，來得出位置。(通常用於2維陣列)

```asm
EX:
  MOV AX,[BX+SI+2]
```
