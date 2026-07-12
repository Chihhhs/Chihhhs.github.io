---
title: Hitcon CyberRange 2024 學生場-WriteUp
date: 2024-11-02 00:36:30
tags: 
    - CyberRange
    - Competition
---

## CyberRange 2024

這次的比賽跟以往打過的 CTF 很不一樣，以往主要都是以攻擊為主軸出題；而CyberRange 則是已經模擬了多個APT攻擊，解題者需要以現有的 Log 以及環境找出攻擊者留下的痕跡。

總共有三個題組

- Web attack: 尋找 splunk 中的攻擊；寫 Surricata
- Malware: 尋找 windows 環境中遺留的Malware processes 及攻擊方式；寫yara
- 第三個沒打到不太確定，但應該也是看 Windows 虛擬機

### WriteUp

- ffuf
- CVE-2024
