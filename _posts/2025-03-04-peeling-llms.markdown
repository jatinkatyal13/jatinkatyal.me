---
layout: post
title:  "Peeling LLMs"
date:   2025-03-04 01:25:00 +0530
categories: ml ai llm
author: "Jatin Katyal"
---

### **🚀 Layers in a Large Language Model (LLM) – A Deep Dive**  

LLMs like GPT, DeepSeek, and LLaMA consist of multiple layers that process text sequentially, refining the information at each stage. These layers can be categorized as follows:  

---

# **1️⃣ Tokenization Layer (Input Representation)**
### **🔹 Function**: Converts raw text into numerical representations (tokens).  
### **🔹 Components**:
✅ **Byte-Pair Encoding (BPE)** / **Unigram Tokenization** – Splits words into subwords.  
✅ **SentencePiece** – A variant that handles out-of-vocabulary words better.  
✅ **Embedding Lookup** – Maps each token to a high-dimensional vector.  

### **🔍 Deep Dive**:
- GPT uses **BPE**, while **LLaMA and DeepSeek** use **SentencePiece**.  
- Embeddings capture word relationships based on pretraining data.  

📌 **Example**:  
"Transformer models are amazing" ⟶ `["Transform", "er", "models", "are", "amazing"]`  
Each token gets converted into a **vector of floating-point numbers**.

---

# **2️⃣ Embedding Layer**
### **🔹 Function**: Converts discrete tokens into dense, continuous vector representations.  
### **🔹 Components**:
✅ **Word Embeddings** – Each token gets a unique vector from a pre-trained table.  
✅ **Position Embeddings** – Adds sequence order information.  
✅ **Rotary Position Embeddings (RoPE)** – Alternative method for relative positioning.  

### **🔍 Deep Dive**:
- **Regular embeddings** (GPT-3) use fixed **position encoding** (sinusoids).  
- **RoPE embeddings** (LLaMA, DeepSeek) use a rotation-based approach for better generalization in long contexts.  

📌 **Mathematical Formula** (for sin-cos encoding):  
$$PE(pos, 2i) = \sin(pos / 10000^{2i/d})$$
$$
PE(pos, 2i+1) = \cos(pos / 10000^{2i/d})
$$

📌 **RoPE formula** (Rotation Matrix):  
$$
RoPE(x) = x \cos(\theta) + R(x) \sin(\theta)
$$
where $$ R(x) $$ is a shifted version of $$ x $$.

🔍 **Why RoPE?** It enables models to extrapolate better to **longer sequences**.

---

# **3️⃣ Self-Attention Layer (Core of Transformers)**
### **🔹 Function**: Allows the model to **focus on important words** in a sentence.  
### **🔹 Components**:
✅ **Query (Q), Key (K), and Value (V) Matrices** – Compute attention weights.  
✅ **Scaled Dot-Product Attention** – Assigns importance scores to tokens.  
✅ **Multi-Head Attention (MHA)** – Runs multiple attention mechanisms in parallel.  

### **🔍 Deep Dive**:
- **Formula for attention weights**:
  $$
  \text{Attention}(Q, K, V) = \text{softmax} \left(\frac{QK^T}{\sqrt{d_k}}\right) V
  $$
- **Why Scale by $$\sqrt{d_k}$$?**  
  Prevents values from becoming too large, stabilizing training.  

### **Multi-Head Attention**
- Instead of a **single attention mechanism**, we use **multiple heads** that learn different aspects of relationships.  

📌 **Example**:  
- One head focuses on **syntax** ("is" related to "running").  
- Another head focuses on **meaning** ("fast" relates to "speed").  

---

# **4️⃣ Feedforward Layer (MLP Block)**
### **🔹 Function**: Applies **non-linear transformations** to enhance representation learning.  
### **🔹 Components**:
✅ **Two linear layers (FC1 & FC2)** – Transform and scale features.  
✅ **Activation function (ReLU/GELU)** – Adds non-linearity.  

### **🔍 Deep Dive**:
The **standard feedforward block**:
$$
\text{FFN}(x) = \text{ReLU}(W_1 x + b_1) W_2 + b_2
$$
where:  
- $$ W_1 $$, $$ W_2 $$ are weight matrices.  
- $$ b_1 $$, $$ b_2 $$ are bias terms.  

🔍 **Why GELU Instead of ReLU?**  
- **GELU** (used in GPT) improves smoothness and model performance.  
- GELU formula:
  $$
  \text{GELU}(x) = 0.5x (1 + \tanh(\sqrt{2/\pi} (x + 0.044715 x^3)))
  $$

---

# **5️⃣ Normalization Layers**
### **🔹 Function**: Stabilizes training and prevents overfitting.  
### **🔹 Types**:
✅ **Layer Normalization (LayerNorm)** – Normalizes across feature dimensions.  
✅ **RMS Normalization (RMSNorm)** – Alternative used in DeepSeek models.  

### **🔍 Deep Dive**:
**LayerNorm Formula**:
$$
LN(x) = \frac{x - \mu}{\sigma} \gamma + \beta
$$
- **$$ \mu $$ and $$ \sigma $$** are mean and std of activations.  
- **$$ \gamma $$, $$ \beta $$** are learnable parameters.  

📌 **Why RMSNorm?**  
- Removes **mean centering**, making computation **more efficient**.

---

# **6️⃣ Residual Connections**
### **🔹 Function**: Helps deep models learn by skipping connections between layers.  
### **🔍 Deep Dive**:
Instead of passing data **sequentially**, residual layers **add** the input back to the output:  
$$\text{Output} = x + \text{Transform}(x)$$

📌 **Why?**  
- Helps gradients flow better (solves vanishing gradients).  
- Allows **deep networks** to train **without degradation**.

---

# **7️⃣ Output Layer (Final Prediction)**
### **🔹 Function**: Converts processed representations into a probability distribution over vocabulary tokens.  
### **🔹 Components**:
✅ **Final Linear Layer** – Maps to vocab size.  
✅ **Softmax Activation** – Converts logits into probabilities.  

### **🔍 Deep Dive**:
$$P(\text{word} | \text{context}) = \frac{e^{z_i}}{\sum_j e^{z_j}}$$
where $$ z_i $$ is the logit for word $$ i $$.

📌 **Why Softmax?**  
- Converts raw scores into **probabilities** that sum to **1**.

---

# **🚀 Special Layers in LLM Variants**
### **🔹 Rotary Position Embeddings (RoPE)**
✅ **Used in LLaMA, DeepSeek** for better **long-context learning**.  

### **🔹 Gated Linear Units (GLU)**
✅ **Used in DeepSeek** to improve **transformer efficiency**.  

### **🔹 SwiGLU Activation**
✅ **Used in GPT-4** to improve training dynamics.  

---

# **🔬 Summary: How These Layers Work Together**
1️⃣ **Tokenization Layer** – Converts text to tokens.  
2️⃣ **Embedding Layer** – Converts tokens to vectors.  
3️⃣ **Self-Attention Layer** – Learns relationships between words.  
4️⃣ **Feedforward Layer** – Applies transformations.  
5️⃣ **Normalization Layers** – Stabilizes training.  
6️⃣ **Residual Connections** – Helps deep learning.  
7️⃣ **Output Layer** – Generates predictions.  

📌 **All these layers stack to form a deep transformer model!**  
