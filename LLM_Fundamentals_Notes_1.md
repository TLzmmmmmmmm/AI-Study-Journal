# Transformer

## 1. What is Transformer
Neural Network Sturcture

### 1.1 What problem does Transformer solve
![Image](\images\image.png)

Before transformer has appered, we usually use **RNN** and **LSTM** to handle text. But these models have a serious problem: too slow while they deal with long text.\n

Transformer uses **Self-Attention** that could look at all words at a time.

### 1.2 Core Idea
- Attention mechanism
    - focus on the most important part in a sentencce

### Training LLM process
1. Prepare data
    - collect large amount of text
2. build structure
    - design neural network
3. begin training
    - keep learning
    - adjust parameters

## 2. Neural Network

### 2.1 Artificial neuron
input1 * weight1 + input2 * weight 2 + input3 * weight3 + bias = output

### 2.2 Structure
1. input layer
2. hidden layer
3. output layer

### 2.3 activation function
- Without activation function, neural network could only do linear calculation

#### Common Actication Function
1. Sigmoid
- σ(x) = 1 / (1 + e^(-x))
- binary output (yes / no)

1. ReLU
- f(x) = max(0, x)
- fast
- reasonable
- better performance
- CNN - Convolutional Neural Network

2. Leaky ReLU
- f(x) = max(0.01x, x)
- little slope for negative side
- CNN - Convolutional Neural Network

3. GELU
- f(x) = x * Φ(x)
- more smooth curve
- combined pobability
- Transformer

4. Tanh
- tanh(x) = (e^x - e^(-x)) / (e^x + e^(-x))
- range: -1 ~ 1
- LSTM / RNN

5. Softmax
- Softmax(xi) = e^xi / Σe^xj
- pobability distribution
- sum of outputs is 1
- multiple classification

#### 2.4 Attention mechanism

- key feature
    - parallel process
    - long distance dependency
    - dynamic weight

## 3. Transformer Structure
### 3.1 Encoder & Decoder
- Encoder: understand input
- Decoder: build output
- Stacked structure: multiple encoder and decoder

### 3.2 Encoder
- self-Attention
- Feed Forward

### 3.3 Decoder
- Masked Self-Attention
- Encoder-Decoder Attention
- Feed Forward

### 3.4 Training process
1. input sentence -> Embedding + Positional Encoding -> encoder
2. encoder handle input layer by layer, extract feature
3. encoder output -> decoder's Encoder-Decoder Attention
4. decoder generate translation word by word
5. linear layer + Softmax -> probability distribution output

## 4. Decoder-Only Structure Application (GPT)

### 4.1 why decoder only
- Tranformer: build for translation
    - encoder - understand source language
    - decoder - generate target language
- Text generate task:
    - input: text prompt
    - output: output content

### 4.2 Core Idea
- take input and output as a same sequence
- use Masked Self-Attention
- no Encoder-Decoder Attention
- stack n smae decoder block

## 5. Self-Attention
### 5.1 Attention Calculation Process
1.  create 3 vectors
    - Query - What information I want
    - Key - What information I provide
    - Value - what is the exact information I have

    - How to get these 3 vectors:
        - X = word's embedding vector (dimension=512)
        - Q = X @ W_Q (dimension=64)
        - K = X @ W_K (dimension=64)
        - V = X @ W_V (dimension=64)
        - X_Q / X_K / X_V are weight matrices
2. calculate attention score
    - score = dot product of Q_currentWord and all other word's K
    - higher score -> similar vector direction -> closer relationship
3. scale
    - scaled socre = score / sqrt(d_k)
    - d_k is dimension of k
    - keep value stable, keep gradient
4. softmax
    - turn score into probability distribution
    - sum is 1
5. calculate weighted sum
    - softmax value * value V of the word
    - output = attention_weights[0] * V0 + attention_weights[1] * V1

### 5.2 Calculation
use matrix multiplication to handle all words at a time

## 6. Multi-head Attention
### 6.1 Why Multi-head is needed
- learn more kinds of different relationships
    - grammar
    - meaning
    - position

### 6.2 implementation
- Run h different attention mechanisms in parallel
- Combine multi-head output (concatenate + linear transformation)
    1. Concatenate all the attention heads
    2. Multiply with a weight matrix W that has trained jointly with the model
    3. The result would be the Z matrix that captures information from all the attention heads. We can send this forward to the FFNN

## 7. Positional Encoding
### 7.1 positional encoding plan
- input_embedding = word_embedding + positional_encoding
- Sinusoidal Positional Encoding
    - PE(pos, 2i) = sin(pos / 10000^(2i/d_model))
    - PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
    - pos = token's position
    - d_model = embedding's dimension
    - i = position encoding vector's dimension
        - the formula means even dimension use sin, and odd dimension use cos
        - for dimension 0 & 1, i=0 ; for dimension 2 & 3, i=1
- Why sin / cos
    - range between -1 and 1
    - for any length sentence, it could generate encoding
    - relative position could be found by simple linear transformation

## 8. Q K V
### 8.1 why use 64D for QKV, but 512D for X
- calculation efficiency - attention calculation complexity is O(n2), the smaller the dimension, the faster the calculation
- Multi-head attention - 8 heads * 64 D = 512 D
- information compress - force model to learn the most important feature

### 8.2 Implementation
```python
import torch
import torch.nn as nn

class SelfAttention(nn.Module):
    def __init__(self, d_model=512, d_k=64):
        super().__init__()
        self.d_k = d_k
        # 定义三个线性变换
        self.W_Q = nn.Linear(d_model, d_k, bias=False)
        self.W_K = nn.Linear(d_model, d_k, bias=False)
        self.W_V = nn.Linear(d_model, d_k, bias=False)
    def forward(self, X):
        """
        X: (batch_size, seq_len, d_model)
        输出: (batch_size, seq_len, d_k)
        """
        # Step 1: 生成Q、K、V
        Q = self.W_Q(X)  # (batch, seq_len, d_k)
        K = self.W_K(X)  # (batch, seq_len, d_k)
        V = self.W_V(X)  # (batch, seq_len, d_k)
        # Step 2: 计算attention scores
        scores = torch.matmul(Q, K.transpose(-2, -1))  # (batch, seq_len, seq_len)
        scores = scores / (self.d_k ** 0.5)  # 缩放
        # Step 3: Softmax
        attention_weights = torch.softmax(scores, dim=-1)
        # Step 4: 加权求和
        output = torch.matmul(attention_weights, V)  # (batch, seq_len, d_k)
        return output, attention_weights
```

## 9. Use PyTorch to implement complete Transformer Block
```python
import torch
import torch.nn as nn
import math

class MultiHeadAttention(nn.Module):
    """多头注意力层"""
    def __init__(self, d_model=512, num_heads=8):
        super().__init__()
        assert d_model % num_heads == 0, "d_model必须能被num_heads整除"
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads  # 每个头的维度
        # Q、K、V的权重矩阵（包含所有头）
        self.W_Q = nn.Linear(d_model, d_model)
        self.W_K = nn.Linear(d_model, d_model)
        self.W_V = nn.Linear(d_model, d_model)
        # 最后的线性变换
        self.W_O = nn.Linear(d_model, d_model)
    def split_heads(self, x, batch_size):
        """
        将最后一维分成(num_heads, d_k)
        x: (batch, seq_len, d_model)
        输出: (batch, num_heads, seq_len, d_k)
        """
        x = x.view(batch_size, -1, self.num_heads, self.d_k)
        return x.transpose(1, 2)
    def forward(self, Q, K, V, mask=None):
        """
        Q, K, V: (batch, seq_len, d_model)
        mask: (batch, seq_len, seq_len) 可选
        """
        batch_size = Q.shape[0]
        # 1. 线性变换
        Q = self.W_Q(Q)  # (batch, seq_len, d_model)
        K = self.W_K(K)
        V = self.W_V(V)
        # 2. 分成多头
        Q = self.split_heads(Q, batch_size)  # (batch, num_heads, seq_len, d_k)
        K = self.split_heads(K, batch_size)
        V = self.split_heads(V, batch_size)
        # 3. 计算attention
        scores = torch.matmul(Q, K.transpose(-2, -1))  # (batch, num_heads, seq_len, seq_len)
        scores = scores / math.sqrt(self.d_k)  # 缩放
        # 4. 应用mask（如果有）
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
        # 5. Softmax
        attention_weights = torch.softmax(scores, dim=-1)
        # 6. 加权求和
        output = torch.matmul(attention_weights, V)  # (batch, num_heads, seq_len, d_k)
        # 7. 合并多头
        output = output.transpose(1, 2).contiguous()  # (batch, seq_len, num_heads, d_k)
        output = output.view(batch_size, -1, self.d_model)  # (batch, seq_len, d_model)
        # 8. 最后的线性变换
        output = self.W_O(output)
        return output, attention_weights

class FeedForward(nn.Module):
    """前馈神经网络"""
    def __init__(self, d_model=512, d_ff=2048):
        super().__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.linear2 = nn.Linear(d_ff, d_model)
        self.relu = nn.ReLU()
    def forward(self, x):
        # x: (batch, seq_len, d_model)
        return self.linear2(self.relu(self.linear1(x)))

class TransformerBlock(nn.Module):
    """Transformer编码器块"""
    def __init__(self, d_model=512, num_heads=8, d_ff=2048, dropout=0.1):
        super().__init__()
        # 多头注意力
        self.attention = MultiHeadAttention(d_model, num_heads)
        # 前馈网络
        self.feed_forward = FeedForward(d_model, d_ff)
        # Layer Normalization
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        # Dropout
        self.dropout = nn.Dropout(dropout)
    def forward(self, x, mask=None):
        """
        x: (batch, seq_len, d_model)
        mask: (batch, seq_len, seq_len)
        """
        # 1. 多头注意力 + 残差连接 + LayerNorm
        attn_output, attention_weights = self.attention(x, x, x, mask)
        x = self.norm1(x + self.dropout(attn_output))
        # 2. 前馈网络 + 残差连接 + LayerNorm
        ff_output = self.feed_forward(x)
        x = self.norm2(x + self.dropout(ff_output))
        return x, attention_weights
```