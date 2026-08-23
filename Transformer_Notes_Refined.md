# Transformer — Fundamental Theory Notes

> Goal: build a clean mental model of the Transformer architecture, with emphasis on the concepts that are essential for understanding modern LLMs.

---

## 1. What Is a Transformer?

A **Transformer** is a neural network architecture designed to process sequences such as text.

It was introduced in the paper **Attention Is All You Need (2017)**. Its central idea is to use **attention** instead of recurrence to model relationships between tokens.

### 1.1 Why Transformer Was Introduced

Before Transformers, sequence models commonly used **RNNs** and **LSTMs**.

These models have two major limitations:

1. **Sequential computation**
   - Tokens are processed one step at a time.
   - This limits parallelization during training.

2. **Long-range dependencies are difficult to model**
   - Information must propagate through many recurrent steps.

Transformers address these problems with **Self-Attention**, which allows each token to directly interact with other tokens in the sequence.

> Important: Self-Attention improves parallelization during training, but standard attention still has roughly quadratic cost with sequence length.

### 1.2 Core Components

A Transformer is mainly built from:

- Token Embedding
- Positional Information
- Self-Attention
- Multi-Head Attention
- Feed-Forward Network (FFN)
- Residual Connections
- Layer Normalization
- Linear Output Layer
- Softmax

---

## 2. Essential Neural Network Concepts

Only a few neural-network concepts are necessary to understand the basic Transformer.

### 2.1 Linear Transformation

A linear layer performs:

$$
y = xW + b
$$

where:

- $x$ = input vector
- $W$ = weight matrix
- $b$ = bias
- $y$ = output vector

Transformer uses linear transformations extensively, especially when creating **Q, K, V** and inside the **FFN**.

### 2.2 Activation Function

Without activation functions, stacking linear layers would still produce only a linear transformation.

The original Transformer uses **ReLU** inside the FFN:

$$
\mathrm{ReLU}(x)=\max(0,x)
$$

Modern LLMs often use activations such as **GELU** or gated variants such as **SwiGLU**.

### 2.3 Softmax

Softmax converts a list of scores into a probability-like distribution:

$$
\mathrm{Softmax}(x_i)=\frac{e^{x_i}}{\sum_j e^{x_j}}
$$

The outputs sum to 1.

In Transformers, Softmax is mainly used in:

- attention weights
- final vocabulary prediction

### 2.4 Residual Connection

A residual connection adds the original input back to the output of a sublayer:

$$
y = x + \mathrm{Sublayer}(x)
$$

This helps information and gradients flow through deep networks.

### 2.5 Layer Normalization

**LayerNorm** normalizes the features of each token representation and helps stabilize training.

A Transformer block usually combines:

- attention / FFN
- residual connection
- LayerNorm

---

## 3. Input Representation

Before entering Transformer blocks, text must be converted into vectors.

### 3.1 Tokenization

Text is first divided into **tokens**.

Example:

```text
"I love machine learning"
        ↓
["I", "love", "machine", "learning"]
```

Modern tokenizers often use subwords rather than complete words.

### 3.2 Token Embedding

Each token is mapped to a dense vector:

$$
\text{token} \rightarrow \text{embedding vector}
$$

If:

$$
d_{model}=512
$$

then every token representation has 512 dimensions.

For a sequence of $n$ tokens:

$$
X \in \mathbb{R}^{n \times d_{model}}
$$

### 3.3 Why Positional Information Is Needed

Self-Attention itself does not inherently encode token order.

For example:

```text
Tom loves Jerry
Jerry loves Tom
```

contain the same tokens but have different meanings because of their order.

Therefore the model must receive information about token positions.

---

## 4. Positional Encoding

The original Transformer uses **Sinusoidal Positional Encoding**.

The input to the first Transformer layer is:

$$
X_{input}=X_{embedding}+X_{position}
$$

### 4.1 Sinusoidal Formula

For token position $pos$:

$$
PE(pos,2i)=\sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

$$
PE(pos,2i+1)=\cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

where:

- $pos$ = token position in the sequence
- $d_{model}$ = embedding dimension
- $i$ = index of a pair of positional-encoding dimensions
- even dimensions use sine
- odd dimensions use cosine

Example for $d_{model}=4$:

$$
PE(pos)=
[\sin(pos),\cos(pos),\sin(pos/100),\cos(pos/100)]
$$

### 4.2 Why Different Frequencies?

Different dimensions change at different rates.

This gives the model positional signals at multiple scales:

- high-frequency dimensions change quickly
- low-frequency dimensions change slowly

The sine/cosine construction also makes relative offsets expressible through simple relationships between positional vectors.

### 4.3 Modern Positional Methods

Sinusoidal encoding is important historically, but modern LLMs may instead use:

- learned positional embeddings
- relative positional encoding
- **RoPE (Rotary Positional Embedding)**

The basic purpose is the same: inject token-order information into the model.

---

## 5. Self-Attention

Self-Attention allows every token to decide how strongly it should use information from other tokens.

### 5.1 Query, Key, and Value

For an input matrix:

$$
X \in \mathbb{R}^{n \times d_{model}}
$$

three learned linear projections are applied:

$$
Q=XW_Q
$$

$$
K=XW_K
$$

$$
V=XW_V
$$

A useful intuition is:

- **Query (Q):** what information am I looking for?
- **Key (K):** what information do I match with?
- **Value (V):** what information do I actually contribute?

These descriptions are intuition only; mathematically, Q, K, and V are learned vector projections.

### 5.2 Attention Scores

The similarity between a query and all keys is computed with dot products:

$$
QK^T
$$

For one token, a larger dot product usually means that the corresponding key is more aligned with its query.

### 5.3 Scaling

The scores are divided by:

$$
\sqrt{d_k}
$$

so:

$$
\frac{QK^T}{\sqrt{d_k}}
$$

Why?

When $d_k$ is large, dot products can become large in magnitude, which may cause Softmax to become extremely sharp and produce small gradients.

Scaling keeps the values in a more stable range.

### 5.4 Softmax

Attention weights are:

$$
A=\mathrm{Softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)
$$

Each row of $A$ tells us how strongly one token attends to the other tokens.

### 5.5 Weighted Sum of Values

The final attention output is:

$$
Z=AV
$$

Therefore the complete Scaled Dot-Product Attention equation is:

$$
\boxed{
\mathrm{Attention}(Q,K,V)=
\mathrm{Softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
}
$$

---

## 6. Matrix Dimensions in Self-Attention

Suppose:

- sequence length: $n$
- model dimension: $d_{model}$
- head dimension: $d_k$

Then:

$$
X: n \times d_{model}
$$

$$
Q,K,V: n \times d_k
$$

$$
QK^T: n \times n
$$

$$
A: n \times n
$$

$$
Z: n \times d_k
$$

The $n \times n$ attention matrix is the main reason standard attention has roughly quadratic time and memory growth with sequence length.

---

## 7. Multi-Head Attention

A single attention mechanism learns one set of relationships.

**Multi-Head Attention** allows the model to learn multiple kinds of relationships in parallel.

Different heads may learn patterns related to:

- syntax
- semantic relationships
- coreference
- local dependencies
- long-range dependencies

These interpretations are not manually assigned; they emerge during training.

### 7.1 Per-Head Attention

For head $i$:

$$
head_i=\mathrm{Attention}(Q_i,K_i,V_i)
$$

### 7.2 Concatenate Heads

All head outputs are concatenated:

$$
\mathrm{Concat}(head_1,head_2,\ldots,head_h)
$$

Then an output projection $W^O$ combines their information:

$$
\boxed{\mathrm{MultiHead}(Q,K,V)=\mathrm{Concat}(head_1,\ldots,head_h)W^O}
$$

### 7.3 Original Transformer Example

The original Transformer base model uses:

- $d_{model}=512$
- $h=8$ attention heads
- $d_k=d_v=64$

because:

$$
8 \times 64 = 512
$$

However, **64 is not a universal Q/K/V dimension**. It is a design choice from the original model configuration.

---

## 8. Attention Masks

Masks control which token relationships are allowed.

### 8.1 Padding Mask

Sequences in the same batch may have different lengths.

Padding tokens should not affect attention, so they are masked out.

### 8.2 Causal Mask

Decoder-only language models must not see future tokens while predicting the next token.

For example, when predicting token 3:

```text
Token 1   Token 2   Token 3   Token 4
   ✓         ✓          ✓         ✗
```

Future attention scores are set to a very negative value before Softmax, making their attention weights approximately zero.

This is called **Masked Self-Attention** or **Causal Self-Attention**.

---

## 9. Feed-Forward Network (FFN)

After attention mixes information between tokens, each token is processed independently by a small neural network.

The original Transformer FFN is:

$$
\mathrm{FFN}(x)=\mathrm{ReLU}(xW_1+b_1)W_2+b_2
$$

For the original base model:

$$
512 \rightarrow 2048 \rightarrow 512
$$

Important property:

> The same FFN parameters are applied independently to every token position.

A useful mental model is:

- **Attention:** exchange information between tokens
- **FFN:** transform the information inside each token representation

---

## 10. Transformer Block

A Transformer is created by stacking many Transformer blocks.

### 10.1 Encoder Block

A classical encoder block contains:

```text
Input
  ↓
Multi-Head Self-Attention
  ↓
Residual Connection + LayerNorm
  ↓
Feed-Forward Network
  ↓
Residual Connection + LayerNorm
  ↓
Output
```

In compact form:

$$
X' = \mathrm{LayerNorm}(X + \mathrm{MHA}(X))
$$

$$
Y = \mathrm{LayerNorm}(X' + \mathrm{FFN}(X'))
$$

This corresponds to the **Post-Norm** style used in the original Transformer.

Many modern LLMs use **Pre-Norm**, where LayerNorm is applied before the attention or FFN sublayer.

### 10.2 Why Stack Multiple Blocks?

Each block produces a new contextual representation.

Lower layers may capture simpler patterns, while deeper layers can build more abstract relationships.

A stack can be viewed as:

```text
Transformer Block N
        ↑
Transformer Block N-1
        ↑
       ...
        ↑
Transformer Block 2
        ↑
Transformer Block 1
        ↑
      Input
```

---

## 11. Original Encoder–Decoder Transformer

The original Transformer was designed for sequence-to-sequence tasks such as machine translation.

### 11.1 Encoder

The encoder processes the source sequence.

Each encoder block contains:

1. Multi-Head Self-Attention
2. Feed-Forward Network
3. Residual Connections
4. Layer Normalization

The encoder produces contextual representations of the input sequence.

### 11.2 Decoder

Each decoder block contains:

1. Masked Self-Attention
2. Encoder–Decoder Attention (Cross-Attention)
3. Feed-Forward Network
4. Residual Connections
5. Layer Normalization

### 11.3 Cross-Attention

In Encoder–Decoder Attention:

- **Q** comes from the decoder
- **K** comes from the encoder output
- **V** comes from the encoder output

This lets the decoder consult the encoded source sequence while generating output tokens.

### 11.4 Generation Flow

A simplified translation process is:

```text
Source Tokens
    ↓
Embedding + Positional Information
    ↓
Encoder Stack
    ↓
Encoder Representations
    ↓
Decoder Stack
    ↓
Linear Layer
    ↓
Softmax
    ↓
Next-Token Probability Distribution
```

The decoder generates tokens autoregressively, one token at a time.

---

## 12. Encoder-Only and Decoder-Only Variants

The original Transformer contains both an encoder and decoder, but many modern models use only one side.

### 12.1 Encoder-Only Models

Example: **BERT**

Characteristics:

- bidirectional Self-Attention
- each token can attend to tokens on both sides
- strong for understanding / representation tasks

Typical uses:

- classification
- information extraction
- semantic representation

### 12.2 Decoder-Only Models

Examples: **GPT-style language models**

Characteristics:

- Causal Self-Attention
- no separate encoder
- no encoder–decoder cross-attention in the basic architecture
- many decoder-style Transformer blocks are stacked

The model treats prompt and generated text as one continuous sequence.

---

## 13. Decoder-Only Language Modeling

A GPT-style model is trained primarily to predict the next token.

Given:

```text
I love machine
```

it learns to predict a probability distribution for the next token:

```text
learning   0.62
vision     0.10
models     0.07
...
```

Mathematically:

$$
P(x_t \mid x_1,x_2,\ldots,x_{t-1})
$$

The language-model objective is to maximize the probability of the correct next tokens across the training data.

A simplified decoder-only flow is:

```text
Tokens
  ↓
Token Embedding + Positional Information
  ↓
Transformer Block 1
  ↓
Transformer Block 2
  ↓
...
  ↓
Transformer Block N
  ↓
Final LayerNorm
  ↓
Linear / LM Head
  ↓
Vocabulary Logits
  ↓
Softmax
  ↓
Next-Token Probabilities
```

---

## 14. Training vs. Generation

This distinction is important.

### 14.1 During Training

Because causal masks prevent access to future tokens, the model can compute predictions for many positions in parallel.

Example:

```text
Input:   I    love    machine    learning
Target: love machine   learning      ...
```

### 14.2 During Generation

Generation is autoregressive:

```text
Prompt
  ↓
Predict token 1
  ↓
Append token 1
  ↓
Predict token 2
  ↓
Append token 2
  ↓
...
```

So Transformer training is highly parallelizable across sequence positions, but token generation is still sequential at the outer autoregressive loop.

---

## 15. Computational Complexity

For sequence length $n$, the attention matrix has size:

$$
n \times n
$$

Standard Self-Attention therefore has approximately:

$$
O(n^2 d)
$$

time complexity for the attention calculation, and attention memory also grows roughly as:

$$
O(n^2)
$$

This is why very long contexts are computationally expensive.

Important distinction:

- reducing $d_k$ can reduce computation per attention operation
- but the **quadratic dependence on sequence length comes from the $n \times n$ attention matrix**

---

## 16. Original Transformer Base Configuration

A useful reference configuration from **Attention Is All You Need**:

| Hyperparameter | Value |
|---|---:|
| Number of encoder layers | 6 |
| Number of decoder layers | 6 |
| $d_{model}$ | 512 |
| Attention heads | 8 |
| $d_k$ | 64 |
| $d_v$ | 64 |
| FFN hidden dimension $d_{ff}$ | 2048 |
| Dropout | 0.1 |

These values are historical examples, not fixed Transformer rules.

---

## 17. End-to-End Mental Model

For a decoder-only LLM, remember this flow:

```text
Raw Text
   ↓
Tokenization
   ↓
Token IDs
   ↓
Token Embeddings
   +
Positional Information
   ↓
┌──────────────────────────────┐
│ Transformer Block            │
│                              │
│  LayerNorm                   │
│      ↓                       │
│  Causal Multi-Head Attention │
│      ↓                       │
│  Residual Connection         │
│      ↓                       │
│  LayerNorm                   │
│      ↓                       │
│  Feed-Forward Network        │
│      ↓                       │
│  Residual Connection         │
└──────────────────────────────┘
   ↓
Repeat N times
   ↓
Final Token Representations
   ↓
Linear / LM Head
   ↓
Vocabulary Logits
   ↓
Softmax
   ↓
Next-Token Probabilities
```

The most important conceptual division is:

> **Attention moves information between tokens.**
>
> **FFN transforms information within each token representation.**

---

## 18. Key Formulas to Remember

### Q, K, V

$$
Q=XW_Q,\qquad K=XW_K,\qquad V=XW_V
$$

### Scaled Dot-Product Attention

$$
\boxed{
\mathrm{Attention}(Q,K,V)=
\mathrm{Softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
}
$$

### Multi-Head Attention

$$
head_i=\mathrm{Attention}(Q_i,K_i,V_i)
$$

$$
\boxed{
\mathrm{MultiHead}(Q,K,V)=
\mathrm{Concat}(head_1,\ldots,head_h)W^O
}
$$

### Feed-Forward Network

$$
\mathrm{FFN}(x)=\mathrm{Activation}(xW_1+b_1)W_2+b_2
$$

### Sinusoidal Positional Encoding

$$
PE(pos,2i)=\sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

$$
PE(pos,2i+1)=\cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

---

## 19. Common Misunderstandings

### Misunderstanding 1: Q, K, and V have fixed meanings

The phrases "what I want", "what I provide", and "actual information" are useful intuition, but Q, K, and V are learned projections rather than manually defined semantic roles.

### Misunderstanding 2: Q/K/V must be 64-dimensional

No. $64$ comes from the original configuration:

$$
512 / 8 = 64
$$

Other models use different dimensions and numbers of heads.

### Misunderstanding 3: Transformer completely removes sequential computation

Training can process many token positions in parallel, but autoregressive generation still produces tokens sequentially.

### Misunderstanding 4: Attention alone is the whole Transformer

Attention is central, but a complete Transformer block also depends on:

- FFN
- residual connections
- normalization
- positional information

### Misunderstanding 5: Positional Encoding always means sine/cosine

Sinusoidal encoding is the original method. Modern Transformer models may use other positional mechanisms such as RoPE.

---

## 20. What You Should Be Able to Explain After Learning This

You should be able to answer these questions without looking at notes:

1. Why did Transformer replace RNN/LSTM in many NLP tasks?
2. Why does Self-Attention need positional information?
3. What are Q, K, and V, and how are they produced?
4. Why do we compute $QK^T$?
5. Why divide by $\sqrt{d_k}$?
6. What does Softmax do inside attention?
7. Why is $V$ multiplied after Softmax?
8. Why use multiple attention heads?
9. What does $W^O$ do after concatenating heads?
10. What is the role of the FFN?
11. What do residual connections and LayerNorm do?
12. What is the difference between encoder, decoder, and decoder-only architectures?
13. Why does GPT need a causal mask?
14. What is next-token prediction?
15. Why does standard attention become expensive for long contexts?

If these questions are clear, you have the core theoretical foundation needed to move on to topics such as **BERT, GPT, RoPE, KV Cache, FlashAttention, LoRA, RAG, and Agents**.
