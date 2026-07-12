---
title: 變形金剛給你一Bonk
date: 2024-12-16 15:29:17
tags:
    - AI
    - Transformer
excerpt: Transformer | Encoder-Decoder | Multi Head Attention
---

[Attention Is All You Need](https://arxiv.org/abs/1706.03762)

Transformer 是一種專為處理 Sequence-to-Sequence 設計的神經網路架構
目的是為了解決序列建模和序列轉換問題，並且克服傳統架構（如 RNN、LSTM）的訓練瓶頸。

## Transformer

<img src="https://hackmd.io/_uploads/BkwuujYE1g.png" width="350">

- Main
    1. Model in/output
    2. Encoder-Decoder
    3. Multi-Head Attention
- Sub
    1. Embedding
    2. Position encode
    3. Feed Forword
    4. Add & Norm
    5. Linear & Softmax

### In/Output

下方的 Input , Output 在 train 時可以等同於 Data,Label
上方 Output 則為 prediction

### Word embedding

把詞語轉為向量，像這邊就是 1 word -> 4 Dimensions

<img src="https://hackmd.io/_uploads/Bk8EpsK4yl.png" width="350">

### Postion Encode

給予 Embedding 後的向量詞語順序，因為模型不會知道這些詞中的關聯
每個詞向量的元素都要編碼
例如: you的位置是1 有四維 -> `PE(1,0) , PE(1,1) , PE(1,2) , PE(1,3)`

+ Postion 計算公式
![image](https://hackmd.io/_uploads/S1WezhK4ye.png)

- pos 代表位置 0,1,2,3...
- 基偶數位，上為偶數位
- i is the postion
- d is the embedding Dimensions

最後把 postion encode 加上 embedding 矩陣就被附加了位置訊息

### Add & Norm

![image](https://hackmd.io/_uploads/rJ_2Ulo4kl.png)

- Add  (殘差連接，Residual Connection)
- Norm (層正規化，Layer Normalization)

+ 通過殘差連接，避免梯度消失問題。
+ 通過層正規化，減少數據分佈的變化對訓練的影響。
+ 殘差連接確保原始輸入信號保留，正規化提升輸出穩定性。

### Feed-Forward Network

![image](https://hackmd.io/_uploads/Sy-QUgo4kl.png)

```python=
class FeedForwardNetwork(nn.Module):
    def __init__(self, d_model, d_ff, dropout=0.1):
        super(FeedForwardNetwork, self).__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.relu = nn.ReLU()
        self.linear2 = nn.Linear(d_ff, d_model)
        self.dropout = nn.Dropout(dropout)
    def forward(self, x):
        x = self.linear1(x)
        x = self.relu(x)
        x = self.dropout(x)
        x = self.linear2(x)
        return x
```

- 增強非線性能力：將多頭注意力生成的線性輸出進一步擴展到高維空間，增強表達能力。
- 捕捉特徵： 提供每個位置上的局部特徵轉換，對序列中的單一位置特徵進行深度學習。
- 模型穩定性：配合 ReLU 和 Dropout，防止過擬合並穩定訓練過程。

### Encoder-Decoder

- Encoder
    1. Multi-Head Attention
    2. Feed Forword
    3. 進行編碼
- Decoder
    1. Masked
    2. Mukti-Head Attention
    3. 進行編碼
- Decoder - 2th
    1. Multi-Head Attention
    2. Feed Forward
    3. Encode + Encode 解碼

### Muti-Head Attention

> In background "Self-attention has been used successfully in a variety of tasks including reading comprehension, abstractive summarization, textual entailment and learning task-independent sentence representations"

Muti-Head Attention 可以看成多個 Self Attention 集合

![image](https://hackmd.io/_uploads/Sy1en1sEyx.png)

1. Q, K, V 線性變換
    - Query：表⽰當前詞需要從上下⽂中獲取什麼資訊。
    - Key：提供句⼦中每個詞的相關特徵。
    - Value：提供每個詞的實際資訊。
2. $K^T$ : 矩陣 𝐾 的轉置 (Transpose)
3. Scaled_Dot_Product_Attention * h
4. 最後 concat 合併所有頭

#### Self Attention

![image](https://hackmd.io/_uploads/ryMDzgiVke.png)

![image](https://hackmd.io/_uploads/ryJj-12Nke.png)

- 通過多個 Self Attection 進行線性變換 (特徵提取)
- h: Head count
- Scaled Dot Product Attention 結合特徵變換後的 Q, K, V

```python=
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, heads):
        super(SelfAttention, self).__init__()
        self.d_model = d_model         # Embedding dim
        self.heads = heads             # head counts
        self.d_k = d_model // heads    # 每個 head 的維度

        # 生成 Q, K, V
        self.q_linear = nn.Linear(d_model, d_model)
        self.k_linear = nn.Linear(d_model, d_model)
        self.v_linear = nn.Linear(d_model, d_model)

        # 輸出層
        self.fc = nn.Linear(d_model, d_model)

    def forward(self, x):
        # x: [batch_size, seq_len, d_model]
        batch_size, seq_len, _ = x.size()

        # 生成 Q, K, V
        Q = self.q_linear(x)  # [batch_size, seq_len, d_model]
        K = self.k_linear(x)  # [batch_size, seq_len, d_model]
        V = self.v_linear(x)  # [batch_size, seq_len, d_model]

        # 拆分多頭，reshape 為 [batch_size, heads, seq_len, d_k]
        Q = Q.view(batch_size, seq_len, self.heads, self.d_k).transpose(1, 2)
        K = K.view(batch_size, seq_len, self.heads, self.d_k).transpose(1, 2)
        V = V.view(batch_size, seq_len, self.heads, self.d_k).transpose(1, 2)

        # 計算注意力得分 (scaled_dot_product_attention)
        scores = torch.matmul(Q, K.transpose(-2, -1)) / torch.sqrt(torch.tensor(self.d_k, dtype=torch.float32))  # [batch_size, heads, seq_len, seq_len]
        attention = torch.softmax(scores, dim=-1)  # [batch_size, heads, seq_len, seq_len]

        # 加權求和
        weighted_sum = torch.matmul(attention, V)  # [batch_size, heads, seq_len, d_k]

        # 多頭拼接，reshape 為 [batch_size, seq_len, d_model]
        concat = weighted_sum.transpose(1, 2).contiguous().view(batch_size, seq_len, self.d_model)

        # 輸出層
        output = self.fc(concat)  # [batch_size, seq_len, d_model]
        return output, attention
```

### Conculsion

整體可以分成五個計算區塊

1. word embedding + postion encode
2. Encoder * N 提取上下文表示
3. Decoder * N (Maked Multi-Head Attection 生成特徵)
4. Encoder-Decoder Attention
5. Softmax 輸出詞分布


## Reference

[详解Transformer中Self-Attention以及Multi-Head Attention](https://blog.csdn.net/qq_37541097/article/details/117691873)
[](https://blog.csdn.net/benzhujie1245com/article/details/117173090)
[](https://www.youtube.com/watch?v=hYdO9CscNes)
[](https://www.youtube.com/watch?v=GGLr-TtKguA)
[](https://www.youtube.com/watch?v=elmR9tWlen0)