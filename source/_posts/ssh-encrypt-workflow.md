---
title: SSH Key Exchange Workflow
date: 2025-04-01 11:31:06
tags: 
    - ssh
excerpt: ssh | key exchange
---

在現代網路安全中，SSH（Secure Shell）是一種不可或缺的安全通訊協議，主要用於遠端登入與資料傳輸。SSH 如何確保傳輸資料的機密性與完整性？這篇文章將帶你了解 SSH 的加密流程，並解釋 HMAC 在其中的作用。

---

## **1. 加密流程概覽**

SSH 的安全性來自三個核心技術：

1. **金鑰交換（Key Exchange）：確保雙方能安全共享 Session Key**
2. **對稱加密（Symmetric Encryption）：使用 Session Key 加密訊息**
3. **訊息完整性驗證（Message Integrity Check）：使用 HMAC 防止資料篡改**

當使用者透過 SSH 連線到遠端伺服器時，SSH 會先進行金鑰交換，然後透過對稱加密保護通訊，並透過 HMAC 驗證完整性。

---

## **2. 金鑰交換（Key Exchange）**

SSH 使用 **Diffie-Hellman（DH）或 Curve25519（ECDH）** 來交換 Session Key，確保 Client 與 Server 之間能安全共享對稱加密金鑰。

**流程：**

1. Client 與 Server 產生各自的隨機私鑰（`a` 與 `b`）。
2. 透過 DH/ECDH 演算法計算公開金鑰 `A` 和 `B`，並交換。
3. 雙方計算相同的 **共享金鑰 K**，計算公式如下：

   ```py
   K = B^a mod p = A^b mod p
   ```

4. Server 使用 SSH Host Key 簽署 `K`，Client 透過已知的 Server 公鑰驗證身份。

最終，雙方取得相同的 **Session Key（K）**，用於後續的加密通訊。

---

## **3. 訊息加密（Encryption）**

完成金鑰交換後，SSH 會使用 **對稱加密** 來保護資料。

常見的加密演算法：

- **ChaCha20-Poly1305**（推薦，速度快，安全性高）
- **AES-GCM**（傳統選擇，適合硬體加速）

範例（使用 AES）：

```py
Encrypted_Message = AES( Session_Key, Plaintext )
```

這確保傳輸的內容即使被攔截，也無法被解讀。

---

## **4. 訊息完整性驗證（HMAC）**

即使訊息被加密，仍可能遭到**竄改攻擊**。為此，SSH 使用 **HMAC（Hashed Message Authentication Code）** 來確保完整性。

HMAC 的作用是：

1. 計算 **加密訊息的 HMAC 值**。

   ```py
   HMAC = HMAC-SHA256( Session_Key, Encrypted_Message )
   ```

2. **附加 HMAC 到封包尾端**。
3. 伺服器收到封包後，重新計算 HMAC，確認資料未被竄改。

HMAC 只是**完整性驗證機制**，**不是加密技術**，它確保資料未被惡意篡改。

---

## **5. 總結：SSH 的安全機制**

| **步驟**          | **技術**                             | **用途**             |
| --------------- | ---------------------------------- | ------------------ |
| **金鑰交換**        | `curve25519-sha256`                | 產生 Session Key `K` |
| **對稱加密**        | `aes128-gcm` / `chacha20-poly1305` | 加密 SSH 資料          |
| **完整性驗證（HMAC）** | `hmac-sha2-256`                    | 確保封包未被篡改           |
