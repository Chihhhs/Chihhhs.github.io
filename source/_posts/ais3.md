---
title: AIS3 
tags: ais3
date: 2025-02-10 21:55:23
excerpt: CyberSecurity | CTF | Camp
---

## 心得紀錄

> [AIS3](https://ais3.org/) 新型態資安暑期課程

我參加了去年與今年的 AIS3，兩次都上了很多有趣且實用的課程，學到了許多新技術與研究；除了 CTF 的技術訓練外，也接觸到大量威脅情資與軟體安全的議題，並在專題製作中累積團隊合作與專案實作經驗。

**第一年重點**

課程：IDA basic, ROP, SROP。

專題：出題並維護 CTF 題目。

- 實作技能：熟悉 exploit 撰寫與 POC（proof-of-concept）、使用 Docker 進行部署、理解 pwn OOB 類型攻擊情境。
- 收穫：建立從 static analysis 到 exploit development 的實務流程，學會環境部署與題目運維。

**專題**

我在專題中負責帶領高中生進行 Web 題目製作，也參與 pwn 題目開發，負責打包 CTF 題目環境與 Exploitation 開發。
Web 題難度為 Easy，利用 john 對 jwt 弱密碼進行爆破。
Pwn 題難度為 Medium，oob 利用與 Docker xinetd 環境架設。

**第二年重點**

課程：log analysis by BERT、MCP auto reversing（自動化 reversing pipeline 概念與工具）。

專題：使用 Neo4j 建立 knowledge graph 以進行 malware detection。

- 實作技能：將 NLP（BERT）應用於 log event 表徵，結合 graph database（Neo4j）進行關聯性分析與可視化。
- 收穫：學到如何把 ML/NLP 與 graph 技術結合，從資料處理到模型驗證走了一遍 end-to-end 的 pipeline。

**專題**

我在專題中擔任報告人與簡報製作，也負責整合資料流程與實驗設計，並主導 Neo4j 知識圖建模與資料前處理。
以 Malware EMBER 資料集建立知識圖，運用 FastRP 產生 graph embedding，接著使用多種 ML 演算法（如 RandomForest、XGBoost）進行惡意程式分類模擬，並比較效能差異。

## 整理

在逆向與 exploit 方向，透過 IDA basic、ROP、SROP 與 CTF 出題實作，我的 reverse engineering 與 exploit thinking 更系統化了；不再只是單點打洞，而能把 memory layout、call stack、以及 control flow 當成一個整體去設計攻防策略。

在資料與檢測方向，學到如何用 BERT 做 log analysis，結合自然語言表示去把 noisy log 轉成有資訊量的 feature；另外嘗試了 MCP auto reversing（自動化 reversing 流程）來加速樣本前處理，並用 Neo4j 建構 knowledge graph，把 indicator、IOC、以及攻擊路徑串成可查詢的圖形化知識庫，有利於 threat hunting 與 incident correlation。

在惡意程式與防護方面，加深了對 malware detection 的理解：從 signature-based 到 behavior-based，再到融入 ML 的 hybrid approach。這些概念讓我能把 static 與 dynamic analysis、network telemetry 與 graph-based context 串起來做更精準的偵測。

在專題製作中體會到真正重要的不只是技術實作，還有 團隊合作 (team cooperation)、任務分工與溝通——如何把 research idea 落地成 prototype、如何把不同成員的 expertise 整合成一個可驗證的系統

## 結語

這兩年的 AIS3 不只是上課、得分或做專題，而是真正把「學到的東西」變成可以用、可以驗證、能協作的實務能力。未來我希望能把在 AIS3 學到的方法帶回研究與實作中：把 BERT 的語意分析、Neo4j 的 graph reasoning、以及 auto reversing 的自動化流程，串成一個更快速、可解釋、可擴充的資安偵測與回應平台。
