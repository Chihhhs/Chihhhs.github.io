---
title: Web fuzzer cheatsheet
date: 2024-11-1 13:01:08
tags:
    - fuzzer
    - web
excerpt: dirb | ffuf
---

## dirb

>基礎掃描

```bash=
dirb http://[URL]
```

## FFUF

### Basic

```bash
ffuf -w [字典檔路徑]:[變數] -c -u [URL]
```

### 常用選項

* `-w`：指定字典檔（支持多個）。
* `-u`：目標 URL（用 FUZZ 表示測試位置）。
* `-v`：詳細模式顯示。
* `-c`：彩色輸出。
* `-mc` 或 -ms：過濾或匹配 HTTP 狀態碼（如：200、403）。
* `-fc` or `-fs`：過濾 HTTP 狀態碼或響應大小。
* `-o`：將結果輸出至檔案（支持 json、html 等格式）。
* `-e`：文件擴展名列表。
* `-X`：HTTP 方法（GET、POST 等）。
* `-d`：POST 資料。
* `-H`：自定義 HTTP 標頭。
* `-recursio`n：啟用遞歸（針對子目錄進行掃描）。
* `-timeout`：請求超時設定。

### 用法範例

* 目錄爆破

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ
```

* 爆破檔案及副檔名

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ -e .php,.html,.txt
```

- 過濾特定狀態碼

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ -fc 404
```

- POST 請求

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/login -X POST -d "username=FUZZ&password=test"
```

- 自定義標頭

```bash
複製程式碼
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ -H "Authorization: Bearer TOKEN"
```

- 遞迴爆破

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ -recursion
```

輸出結果

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ -o output.json -of json
```

### 篩選及匹配

- 匹配內容長度

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ -ml 1024
```

- 匹配 HTTP 狀態碼

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ -mc 200
```

- 過濾內容長度

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ -fl 1024
```

### 進階用法

- 多變數模糊測試

```bash
ffuf -w /usr/share/wordlists/dirb/users.txt:USER -w /usr/share/wordlists/dirb/passwords.txt:PASS -u https://target.com/login -d "username=USER&password=PASS"
```

- 基於響應關鍵字過濾

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://target.com/FUZZ -fr "Not Found"
```
