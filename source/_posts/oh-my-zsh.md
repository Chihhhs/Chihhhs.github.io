---
title: oh-my-zsh p10k 
date: 2024-08-11 02:13:04
tags: zsh
excerpt: Mod oh-my-zsh
---

## oh-my-zsh

oh-my-zsh 就是一個可以讓你的 Zsh 變好看的東東，p10k 是他的一個 Theme
這篇文章主要講我以為裝完 oh-my-zsh 後把 python-env 搞爛，然後找方法變回來的過程。

首先看了[這篇](https://stackoverflow.com/questions/38928717/virtualenv-name-not-show-in-zsh-prompt)，

試了一下重新 `python -m venv .venv` 就可以了，但還是沒有 `(.venv)` 的prompt

google一下，應該跟這個資料夾有關 `.oh-my-zsh/plugins/virtualenv`，註解掉裡面的 `export VIRTUAL_ENV_DISABLE_PROMPT=1`

```bash
~/.zshrc 
plugins=(virtualenv)
source ~/.zshrc
```

還是沒用，但輸入 `virtualenv_prompt_info` 會跳出 `[.venv]` 所以開始找把這個函數嵌入prompt的方法

然後發現是我plugins寫錯了 正確寫法是`plugins = (git virtualenv)` 把兩個各寫一個plugins，然後就變成這樣了

![image](/images/zsh-bug-1.png)

有點破爛啊 (~~後來發現是把 .bashrc 的 Theme 改壞了~~)

## 修改 .p10k.zsh

```bash
  typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
    # =========================[ Line #1 ]=========================
    os_icon               # os identifier
    dir                     # current directory
    vcs                     # git status
    # =========================[ Line #2 ]=========================
    newline                 # \n
    # prompt_char           # prompt symbol
  )
```

然後發現本來就有pyenv的資訊了，就寫在後面的小框框 :>
發現時間 4:00 AM，有時候都覺得我根本沒腦

三小時學會改zsh :>>>

## REF

zsh p10k可以看[這篇](https://onejar99.com/zsh-powerlevel10k-custom-config-note/)
