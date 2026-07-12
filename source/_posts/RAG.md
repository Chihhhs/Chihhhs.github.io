---
title: 威脅情資 LLM 應用
date: 2024-11-26 12:52:25
excerpt: ATT&CK | LLM | RAG
tags: 
    - Research
    - LLM
    - Finetune
    - RAG
---

## 前言

上學期自主學習因應老師要求，以 ATT&CK 框架為基礎製作可以在駭客攻防課程中使用的**攻擊鏈**助教，最後以應用技術效率對比結尾。
但我覺得應該做成情資分析，例如：輸入情報分析 Technique | KillChain | Mitigations 等 ，並轉換成結構化數據，以 LLM 做 Data pre-process，也可以有更多使用方向。

主要實作方式：

1. LLama2
    + RAG
    + LLM Finetune
2. ChatGPT3.5-turbo

最後做出來的效果是 Finetune > GPT3.5 > RAG ，但Finetune所需要的製作成本比 RAG 高出超級多，所以會建議在製作 LLM 聊天客服應用時還是以 GPT + RAG 打底。

### 文章架構

這個文章應該會是一個系列，總共分三篇

1. RAG 應用搭建
2. LLM Finetune
3. Fintune 後的模型格式轉換及模型壓縮

最後做出來的成果主要以 Llama2 RAG 和 Finetune 的效果、硬體資源做評估，主要結論在上面。

### Repositories

實作範例

>做完 LangChain 版本就換了🤡🤡 有些可能沒辦法跑，但邏輯都差不多

1. RAG系統
[Assistant System](https://github.com/Chihhhs/Assistant_for_Attck)

2. 微調訓練及模型轉換
[TRL_Trainer](https://github.com/Chihhhs/TRL_Trainer)

3. Model 主要存放在 HuggingFace 和 OLLama 平台上。
    1. [LoRA WeightFile](https://huggingface.co/chihhh/attack-llama-2-7b-chat-lora)
    2. [GGUF File](https://huggingface.co/chihhh/llama2-chat-attck-gguf)
    3. [Ollama Interface](https://www.ollama.com/chih/llama-2-chat-attack)

## 系列文

1. [研究介紹](./RAG.md)
2. [RAG簡介] 還沒寫
3. [Finetune簡介] 還沒寫
