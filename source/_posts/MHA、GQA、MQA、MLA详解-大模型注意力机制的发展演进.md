---
title: MHA、GQA、MQA、MLA详解--大模型注意力机制的发展演进
toc: true
date: 2026-07-22 01:40:42
categories: AI
tags: AI
mathjax: true
---

在现代大语言模型（LLM）的演进过程中，**注意力机制（Attention Mechanism）** 的优化是一个核心议题。随着模型上下文长度（Context Length）的不断增长，推理（Inference）阶段的 **KV Cache（键值缓存）** 逐渐成为了限制吞吐量和并发量的主要内存瓶颈（即所谓的“内存墙”瓶颈）。

为了在维持模型表达能力的同时减少 KV Cache 的显存占用，研究人员相继提出了 **MHA、MQA、GQA** 以及由 DeepSeek 提出的 **MLA**。

---

### 背景：为什么 KV Cache 会成为瓶颈？
在自回归（Autoregressive）生成文本时，模型需要逐字输出。为了避免重复计算历史 Token 的 Key（键）和 Value（值），模型会将它们保存在显存中，这就是 **KV Cache**。
随着上下文长度 $s$、批量大小（Batch Size）$b$、模型层数 $l$ 的增加，KV Cache 的显存占用呈线性甚至几何级数增长。这导致 GPU 的显存快速被填满，限制了最大并发数。

---

### 1. MHA (Multi-Head Attention, 多头注意力机制)
**MHA** 是 Transformer 原型（Vaswani 等，2017）采用的标准机制。

*   **结构原理**：
    每个注意力头（Head）都有自己独立且不共享的 Query ($Q$)、Key ($K$) 和 Value ($V$) 投影矩阵。
    *   设头数为 $h$，每个头的维度为 $d_h$。
    *   对于每个 Token，其 $Q, K, V$ 维度均为 $h \times d_h$。
*   **KV Cache 占用（单层单 Token）**：
    $$\text{Size}_{\text{MHA}} = 2 \times h \times d_h \quad (\text{参数量})$$
*   **优缺点**：
    *   **优点**：每个头可以独立关注不同的特征，模型表达能力和精确度极高。
    *   **缺点**：KV Cache 极大，推理时受 GPU 显存带宽限制严重，难以支持超长上下文或大 Batch 部署。

---

### 2. MQA (Multi-Query Attention, 多查询注意力机制)
**MQA**（Shazeer，2019）为了极致的推理效率，对 MHA 进行了大刀阔斧的修剪。

*   **结构原理**：
    所有的注意力头共享 **唯一的一组** Key 和 Value。也就是说，Query 依然有 $h$ 个头，但 Key 和 Value 只有 1 个头。
*   **KV Cache 占用（单层单 Token）**：
    $$\text{Size}_{\text{MQA}} = 2 \times 1 \times d_h \quad (\text{参数量})$$
    其显存占用仅为 MHA 的 $1/h$。
*   **优缺点**：
    *   **优点**：极大地减少了 KV Cache 的大小（可降低 90% 以上），大幅度释放了显存，提高了推理吞吐量。
    *   **缺点**：强制所有头共享相同的 K 和 V，严重削弱了模型对多维度特征的捕捉能力，导致模型精度和表现力有所下降。

---

### 3. GQA (Grouped-Query Attention, 分组查询注意力机制)
**GQA**（Ainslie 等，2023）是目前绝大多数主流开源大模型（如 LLaMA-2/3、Mistral、Qwen 等）采用的折中方案。

*   **结构原理**：
    它介于 MHA 与 MQA 之间。将 $h$ 个 Query 头平分为 $g$ 个组。**每个组内** 的 Query 头共享一组 Key 和 Value。
    *   若 $g=1$，则退化为 MQA；
    *   若 $g=h$，则等价于 MHA。
    *   一般实践中，会选择 $g = 8$。
*   **KV Cache 占用（单层单 Token）**：
    $$\text{Size}_{\text{GQA}} = 2 \times g \times d_h \quad (\text{参数量})$$
*   **优缺点**：
    *   **优点**：通过调节组数 $g$，在**推理速度**（KV Cache 压缩）与**模型质量**（特征捕获能力）之间取得了良好的平衡。
    *   **缺点**：虽然比 MHA 好得多，但在极长上下文或极大并发下，KV Cache 仍嫌不够精简。

---

### 4. MLA (Multi-head Latent Attention, 多头潜在注意力机制)
**MLA** 是由 DeepSeek 团队在 DeepSeek-V2 中首次提出的创新型注意力机制。它的目标是：**在实现类似甚至超越 MQA 的 KV Cache 压缩率的同时，保持乃至超越 MHA 的模型精度**。

MLA 的核心思路并非像 GQA/MQA 那样“减少注意力头的数量”，而是通过**低秩矩阵分解（Low-Rank Compression）**，在潜在空间（Latent Space）中对 Key 和 Value 进行联合压缩。

#### 核心机制 ①：低秩 KV 联合压缩 (Low-Rank Joint Compression)
在 MLA 中，输入向量 $\mathbf{h}_t$ 首先被投影到一个维度极低的潜在向量 $\mathbf{c}_t^{KV}$（例如维度 $d_c = 512$，而常规 MHA 的 $h \times d_h$ 可能达到 16384）：

$$\mathbf{c}_t^{KV} = W^{DKV} \mathbf{h}_t \in \mathbb{R}^{d_c}$$

在推理时，**KV Cache 只需要保存这一个低维的潜变量 $\mathbf{c}_t^{KV}$ 即可**。

当需要计算注意力时，模型再通过升维矩阵恢复出每个头对应的 Content Key $\mathbf{k}\_{t,i}^C$  和 Value  $\mathbf{v}\_{t,i}^C$ ：
$$\mathbf{k}\_{t,i}^C = W_{i}^{UK} \mathbf{c}\_t^{KV} \in \mathbb{R}^{d_h}$$
$$\mathbf{v}\_{t,i}^C = W_{i}^{UV} \mathbf{c}\_t^{KV} \in \mathbb{R}^{d_h}$$

#### 核心机制 ②：矩阵吸收（Weight Absorption）
如果每次计算 Attention 都要实时把 $\mathbf{c}_t^{KV}$ 升维成高维的 K 和 V，计算开销会非常大。MLA 巧妙地利用了矩阵乘法的结合律（**矩阵吸收**）来规避这一开销：

已知 Query $\mathbf{q}\_{t,i}^C = W\_{i}^{UQ} \mathbf{c}\_t^Q$（Query 也经过了类似的低秩压缩）。
Content Attention 权重的计算公式为：
$$(\mathbf{q}\_{t,i}^C)^T \mathbf{k}\_{s,i}^C = (W\_{i}^{UQ} \mathbf{c}\_t^Q)^T (W\_{i}^{UK} \mathbf{c}\_s^{KV}) = (\mathbf{c}\_t^Q)^T \left[ (W\_{i}^{UQ})^T W\_{i}^{UK} \right] \mathbf{c}\_s^{KV}$$

因为 $(W_{i}^{UQ})^T W_{i}^{UK}$ 均为静态的权重矩阵，我们可以在推理前将其合并为一个低维的注意力权重矩阵 $W_{i}^{attn} \in \mathbb{R}^{d_c^Q \times d_c^{KV}}$。
这样，计算公式化简为：
$$(\mathbf{c}\_t^Q)^T W\_{i}^{attn} \mathbf{c}\_s^{KV}$$
这意味着，**我们根本不需要在推理时恢复高维的 Key 向量，直接在低维潜空间中便能完成注意力权重的计算**！类似的吸收技巧也应用在了 Value 的融合计算中。

#### 核心机制 ③：解耦旋转位置编码 (Decoupled RoPE)
主流 LLM 使用 RoPE（旋转位置编码）来引入位置信息，但 RoPE 具有位置敏感的旋转矩阵，破坏了矩阵结合律，导致上述的“矩阵吸收”无法直接应用。

为了解决这个“RoPE 悖论”，MLA 采用了**解耦设计**：
1.  **Content 向量**：$\mathbf{q}\_{t,i}^C$ 和 $\mathbf{k}\_{t,i}^C$，通过低秩压缩和矩阵吸收来计算，**不施加 RoPE**。
2.  **Position 向量**：单独分配一个极小维度的多头 $Q$ 和共享单头 $K$ 用于携带位置信息（记为 $\mathbf{q}\_{t,i}^R \in \mathbb{R}^{d_R}$ 和 $\mathbf{k}\_{t}^{R} \in \mathbb{R}^{d_R}$），**在它们上面施加 RoPE**。
3.  最终的注意力分数由两部分直接相加：
    $$\text{Score}\_{t,s,i} = (\mathbf{q}\_{t,i}^C)^T \mathbf{k}\_{s,i}^C + (\mathbf{q}\_{t,i}^R)^T \mathbf{k}\_{s}^{R}$$

因此，除了低维潜变量 $\mathbf{c}_t^{KV}$，MLA 还需要在 KV Cache 中额外保存一份非常小且带有位置信息的 $\mathbf{k}\_s^R$。
以 DeepSeek-V2 为例，单层单 Token 的 KV Cache 占用仅为：
$$\text{Size}\_{\text{MLA}} = d_c + d_R = 512 + 64 = 576 \quad (\text{参数量})$$
这远远小于 MHA 的 $2 \times 128 \times 128 = 32768$，甚至比配置为 8 组的 GQA 还要小。

---

### 四种注意力机制的对比总结

| 特征 / 机制 | **MHA (Multi-Head)** | **MQA (Multi-Query)** | **GQA (Grouped-Query)** | **MLA (Multi-head Latent)** |
| :--- | :--- | :--- | :--- | :--- |
| **KV 头数量** | 与 Query 头相同（如 $h=128$） | 仅 1 个组共享 | 分组共享（如 $g=8$ 组） | **不直接压缩头数**，而是压缩通道维度 |
| **单层单 Token Cache 占用** | $2 \times h \times d_h$ | $2 \times 1 \times d_h$ | $2 \times g \times d_h$ | $d_c + d_R$ (潜在维度 + 旋转维度) |
| **显存占用示例** *(以 DeepSeek-V2 规格：h=128, dh=128 为例)* | **32,768** 标量 | **256** 标量 | **2,048** 标量 (8 组) | **576** 标量 ($d_c=512, d_R=64$) |
| **精度与表达能力** | **极高** (基准) | **较低** (丢失了多头关联信息) | **中等偏高** (主流折中方案) | **极高** (媲美甚至优于 MHA) |
| **推理延迟 / 显存带宽瓶颈** | **严重** (受显存读取限制) | **极轻微** (吞吐量极大) | **轻微** (目前最主流的平衡手段) | **极轻微** (计算量稍增，但读取带宽占用极低) |
| **典型应用模型** | GPT-3, LLaMA-1 | Falcon, MPT | LLaMA-2/3, Mistral, Qwen2 | DeepSeek-V2 / V3 |

### 结语
*   **MHA** 提供了最完整的表示能力，但在推理阶段显存开销难以承受。
*   **MQA** 和 **GQA** 通过在“空间（头数）”上进行剪枝来降低显存，虽然在一定程度上解决了显存问题，但在大模型规模化（Scaling）时难以避免精度的损耗。
*   **MLA** 则通过低秩投影将注意力信息压缩进一个潜在空间，并在推理时使用矩阵吸收和解耦 RoPE 规避重构开销，巧妙地做到了“既要（MHA 的高精度）又要（MQA 的超低显存占用）”，代表了当前大语言模型在注意力机制工程优化上的高水平实践。