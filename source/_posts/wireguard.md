---
title: Wireguard VPN
date: 2024-07-31 00:37:32
tags: 
    - vpn
    - server
excerpt: VPN setup
---

## Setup wireguard VPN

> 要先有一台機器
>>租個雲服務都可以，這邊推薦 [Linode](https://linode.com)

`sudo apt install wireguard`

[](https://www.wireguard.com/install/)

### Config

推薦使用[這個](https://www.wireguardconfig.com/)生config檔 (我就懶👍)
>不然要用 `wg` 生Server,Client 的 private/public key 然後寫conf

把 Server File 丟進 `/etc/wireguard/wg0.conf` 後 `wg-quick up wg0`
client File 丟進 app

就行了💯

這個改定位沒什麼用，只能改ip所以有GPS驗證的可能沒辦法跳過
