---
title: WinREV-WriteUp
date: 2024-08-05 14:49:34
tags: 
    - rev
    - ctf
excerpt: false
---

>From `aaaddress1` [資安食物不好吃](https://www.youtube.com/watch?v=O8AQwUaTmdI)

## Compiler

### .c -> .exe

1. **Source.cpp** -> `Compiler` ->
2. **Assembly codes** ->`Assembler` ->
3. **Object file** -> `Linker` ->
4. **Main.exe**

### Data section

![image](https://hackmd.io/_uploads/BySKnRdo6.png)

`.rdata Section`: push offset -> 從記憶體找到變數內容

`.idata Section`: Import address table

![image](https://hackmd.io/_uploads/S1CDpCuip.png)

### Assemble to machine code

`.text Section`

```asm=
push 0            ; 6A 00
push Oxdead       ; 68 AD DE 00 00

push Oxbeef       ; 68 EF BE 00 00
push 0            ; бА 00
call ds: 0xcafe   ; FF 15 FE CA 00 00
xor eax, eax      ; 33 CO
ret               ; C3
```

### Finish

![image](https://hackmd.io/_uploads/SJiGy1YjT.png)

+ `.data Section` point to `.text Section`

### Demo

```bash
$gcc -S hell_world.c ; compile only ,no assemble and linker -> hell_world.s
$gcc -masn=intel -S hell_world.c ; x86 asm
$gcc -c hell_world.c ; compile and assemble ,on linker -> hell_world.o (COFF object file) 
```

## COFF File

`PE viewer`

> [解析 PE 文件格式](https://ithelp.ithome.com.tw/articles/10187490)

### 欄位

+ IMAGE_FILE_HEADER
+ IMAGE_SECTION_HEADER
...
+ SECTION
    .text
    .data
    textS_Z6printfPKcz
    .rdata
    .rdataSzzz
    .eh_frames_26printfPKcz
    .eh_frame

`Visual Studio` 查看欄位

### Symbol table

> 顯示欄位中包含的變數

+ Strip
`gcc -o output_file input_file.c [--strip or -s ]`

>用於從目標文件中刪除符號表和調試信息。這可以減少目標文件的大小，使其更適合用於部署。

### Read COFF

![image](https://hackmd.io/_uploads/HkxarkYiT.png)

## Linker

`COFF overview`

![image](https://hackmd.io/_uploads/B1GvWbKsT.png)

### cat ./a.out

![image](https://hackmd.io/_uploads/H1V_lZto6.png)

### cat ./PE

![image](https://hackmd.io/_uploads/S1BhfZFjT.png)

### 計算斷點

![image](https://hackmd.io/_uploads/BkIRoZFo6.png)

+ `Star point` = Entry-Point + Image-Base

### Process

![image](https://hackmd.io/_uploads/ryJqQKnsT.png)

1. CreateProcess
2. ChildProcess
3. `File Mapping`
4. Kernal base module
5. To AddressOfEntry

![image](https://hackmd.io/_uploads/rJU4rKhjp.png)

1. Stack
2. NT Header
3. Section Header Array
4. Data

[**Loader**](https://hackmd.io/@alan303138/Sy26KTAJY#載入器) -> Section Mapping

>NT header from `4000000`, before is stack memory

![image](https://hackmd.io/_uploads/ByoJDcb30.png)

local variable -> stack
golbal variable -> .data .idata

![截圖 2024-09-01 22.22.33](https://hackmd.io/_uploads/BkOhdgMnR.png)

> [Import table](https://ithelp.ithome.com.tw/articles/10187582)

+ Mult threads from Modules
+ 每新增一個Thread會創建一個 TEB 在 process 中

![截圖 2024-09-01 23.53.27](https://hackmd.io/_uploads/BJs-0ZM3R.png)

>執行緒環境塊（Thread Environment Block，TEB）
>access from FS segment register when operating on  32 bits ,and from GS in 64 bits

+ FS[0] -> ExceptionList
+ FS[0x4] -> StackBase
+ FS[0x8] -> StackLimit
+ FS[0x18] -> 可以拿到整塊TEB的記憶體位置
+ FS[0x30] -> PEB

![image](https://hackmd.io/_uploads/r1GDCZMn0.png)

> [PEB 結構](https://learn.microsoft.com/zh-tw/windows/win32/api/winternl/ns-winternl-peb)

+ File mapping 後創建 PEB
+ FS[0x30] PEB -> File name/path , Command line下的參數

> 不論是 .exe 還是 .dll 都是一個 Obj File 的封裝，有 mapping 進不同的module
>>PEB offset +8 , ImageBaseAddress -> 標示出哪一個 Module 是主要的 (進去之後應該就會看到 MZ , DosHeader)

+ main 是對於開發者的入口點
+ 在 linking過程中 linker會把一包程式裝在前面，來自`c://windows/sysWOW64/msvcrt.dll`
+ 使用 cff explorer 可以看到 dll 裡面有什麼東西

> [dll 顯微鏡](https://ithelp.ithome.com.tw/m/articles/10241357)

+ 從 eax的變化就可以大概看出他大概做了什麼

### Process Hollowing

[r](https://ithelp.ithome.com.tw/articles/10277670)

+ M: My program
+ T: Target exe (Have digital sign)

1. Maping T
2. write_memory 注入到T的記憶體中
3. 把Ｔ的函數入口點改成Ｍ的

### Review exe

`PE-bear`
`x64-dbg`
