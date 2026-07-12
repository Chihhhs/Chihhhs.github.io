---
title: How to use gdb-WriteUp
date: 2024+08+09 15:52:02
tags: pwn
category: PWN
---

## How to use gdb

>因為我寫到srop的lab才發現我不會用gdb，所以這邊趕快補一下🤧

### open gdb

+ attach `<pid>`
+ pwntools open gdb

    ```python=
    from pwn import *
    context.terminal = ['tmux', 'splitw', '+h'] # 要先開tmux

    r = process("./rop")

    gdb.attach(r) # 在想要的位置開gdb

    ```

### 調適

gdb **Commands**

+ r = run：運行程式。如果程式已經在運行，輸入 r 會重新啟動程式。
+ b = break：設置斷點，通常在調試pwn時用來在特定的記憶體地址處停住程式，例如 b 0x400908 或 b *main+<行號>。
+ c = continue：繼續執行程式，直到遇到下一個斷點。
+ n = next：執行下一行指令。
+ ni = nexti：執行下一條指令，這是真正的下一個組譯指令。
+ s = step：單步執行程式，當進入到不同的函數時會停下來，一般會在呼叫 call 之後停在被呼叫函數的開始位置。
+ d = delete：刪除斷點，後面可以跟數字來指定刪除哪個斷點。如果不帶參數，則刪除全部斷點。
+ x：查看記憶體內容。
+ xinfo: 指定地址的值及其對應的內存內容。

>補完發現好想沒什麼好補的，剩下也是靠經驗🤧
