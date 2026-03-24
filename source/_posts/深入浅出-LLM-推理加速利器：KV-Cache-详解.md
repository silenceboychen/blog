---
title: 深入浅出 LLM 推理加速利器：KV Cache 详解
toc: true
date: 2026-03-24 17:14:42
categories: AI
tags: AI
mathjax: true
---

在大型语言模型（LLM）如火如荼的今天，不管是 ChatGPT 还是开源的 Llama、Qwen，“推理速度”和“显存占用”始终是绕不开的痛点。如果你曾尝试部署过大模型，或者研究过 vLLM、TGI 等推理框架，你一定频繁听到过一个词——**KV Cache**。

什么是 KV Cache？为什么大模型推理离不开它？它又是如何吃掉你昂贵的 GPU 显存的？今天，我们就来扒一扒 KV Cache 的底层逻辑与优化方案。

---

## 为什么我们需要 KV Cache？

### 大模型的“自回归”生成机制
绝大多数文本大模型（如 GPT 系列、Llama 系列）都基于 Transformer 的 Decoder-Only 架构。它们生成文本的方式是 **自回归（Autoregressive）** 的：即根据输入（Prompt）和已经生成的词，一个个地预测下一个词（Token）。

假设输入是 `"今天天气"`，模型生成 `"真"`，接着输入变成 `"今天天气真"`，模型再生成 `"好"`，以此类推。

### 注意力机制中的重复计算
在 Transformer 中，最核心的机制是**自注意力（Self-Attention）**。计算公式如下：
$$ \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V $$

在没有 KV Cache 的情况下，当我们为了生成第 $N+1$ 个 Token，需要将前 $N$ 个 Token 再重新过一遍模型，计算出它们的 $Q$（Query）、$K$（Key）、$V$（Value）。
这就带来了一个严重的问题：**为了生成第 $N+1$ 个词，前 $N$ 个词的 $K$ 和 $V$ 被重复计算了！** 随着生成序列越来越长，这种冗余计算会呈指数级增长，导致推理速度越来越慢。

### 破局之道：KV Cache
为了避免这种毫无意义的重复劳动，**KV Cache（键值缓存）** 应运而生。
它的核心思想非常简单：**空间换时间**。我们在每一步生成时，把当前 Token 每一层的 $K$ 和 $V$ 矩阵保存到显存里。当生成下一个 Token 时，只需计算最新 Token 的 $Q, K, V$，然后将新的 $K, V$ 追加到之前的缓存中，直接参与 Attention 计算。

---

## KV Cache 的工作流程：Prefill 与 Decode

理解 KV Cache，就必须理解大模型推理的两个经典阶段：**Prefill（预填充）** 和 **Decode（解码）**。

1. **Prefill 阶段（首字响应）**：
   - 此时用户输入了 Prompt（比如一段 1000 字的问题）。
   - 模型会**并行**处理这 1000 个 Token，一次性计算出它们所有的 $Q, K, V$。
   - 算出结果后，不仅输出了第一个生成的 Token，还会把这 1000 个 Token 对应的 $K$ 和 $V$ 保存到显存中。这就是最初的 KV Cache。
   - *此阶段是“计算密集型”（Compute-Bound）。*

2. **Decode 阶段（逐字生成）**：
   - 模型拿着最新生成的一个 Token 放进网络。
   - 此时只需要计算这 **1 个 Token** 的 $Q, K, V$。
   - 将新计算的 $K, V$ 追加到上一步的 KV Cache 中。
   - 用这 1 个 Token 的 $Q$ 去和 KV Cache 中所有的 $K$ 点乘算注意力权重，再和所有的 $V$ 加权求和。
   - *此阶段是“显存带宽密集型”（Memory-Bandwidth-Bound），因为每生成一个词，都需要把庞大的 KV Cache 从显存搬运到计算单元。*

---

## KV Cache 到底有多大？（显存刺客）

很多初学者认为，部署一个 7B 的模型只需要 14GB 的显存（FP16）。但在实际并发推理时，系统往往会 Out Of Memory (OOM)，罪魁祸首就是 KV Cache。

让我们来算一笔账，**一个 Token 的 KV Cache 到底占多少显存？**

**公式：**
`单 Token 显存占用 = 2 (K和V) × 层数 (Layers) × 注意力头数 (Heads) × 头维度 (Head Dim) × 数据类型字节数`

以 **Llama-2-7B** 为例：
*   层数 (Layers) = 32
*   注意力头数 (Heads) = 32
*   头维度 (Head Dim) = 128
*   数据类型：FP16（占用 2 Bytes）

单 Token 的 KV Cache = $2 \times 32 \times 32 \times 128 \times 2 = 524,288$ Bytes $\approx 0.5$ MB。

看起来不大对吧？但别忘了这只是一个 Token！
*   如果上下文长度（Sequence Length）达到 **2048**，一个句子的 KV Cache 就是 **1 GB**。
*   如果服务部署时 Batch Size 设置为 **16**，那么仅 KV Cache 就会吃掉 **16 GB** 的显存！这就超过了模型本身的权重大小。

---

## 业界如何优化 KV Cache？

因为 KV Cache 既占显存容量，又吃显存带宽，业界提出了多种“魔改”方案来对其进行压榨。目前主流的优化技术包括以下四个方向：

### 模型架构层面：MQA 与 GQA
既然 $K$ 和 $V$ 太大，能不能减少它们的数量？
*   **MHA (Multi-Head Attention)**：标准做法，每个 Q 头都有对应专属的 K 头和 V 头。
*   **MQA (Multi-Query Attention)**：所有 Q 头**共享同一组** K 头和 V 头。显存直接锐减到原本的 1/32（假设 32 头的模型），但可能会牺牲一点模型能力。
*   **GQA (Grouped-Query Attention)**：折中方案。将 Q 头分组（比如 8 个一组），每组共享一个 K 头和 V 头。这也是目前 Llama-3、Qwen-2 等主流模型标配的架构，能在保证效果的同时，大幅降低 KV Cache 大小。

### 显存管理层面：PagedAttention (vLLM)
在早期的实现中，KV Cache 需要在显存中分配一段**连续的空间**。因为序列长度是动态变化的，往往会预先分配最大长度，导致产生严重的**内部碎片**和**外部碎片**，显存浪费率高达 60%。

**vLLM** 提出了破圈的 **PagedAttention** 技术。它借鉴了操作系统的虚拟内存和分页管理：
把 KV Cache 切分成固定大小的块（Block，如包含 16 个 Token）。在物理显存中，这些块是不连续的，通过一张“页表”（Block Table）来记录逻辑 Token 到物理 Block 的映射。这几乎消灭了显存碎片，让同等显存下的并发量提升了 2-4 倍。

### 量化层面：KV Cache Quantization
不仅可以对模型权重做量化，也可以对 KV Cache 做量化。
在将 $K, V$ 写入显存前，将其从 FP16/BF16 压缩成 **INT8 甚至 INT4/FP8**。在计算 Attention 前再反量化。配合特定的硬件指令，这不仅能缩小一半以上的显存占用，还能极大缓解 Decode 阶段的显存带宽瓶颈。目前 TensorRT-LLM 和 vLLM 都原生支持了 KV Cache 的量化。

### 缓存淘汰策略：Sliding Window 与 StreamingLLM
对于无限长文本的生成，显存总有爆掉的一天。
*   **Sliding Window Attention (SWA)**：例如 Mistral 模型，只保留最近 N 个 Token 的 KV Cache（类似滑动窗口），因为通常最近的词对预测下一个词最重要。
*   **StreamingLLM**：研究发现 Attention 中存在“注意力陷阱”（Attention Sinks），即第一句话的首几个 Token 总是分配到很高的注意力权重。因此，只要保留**前几个 Token + 最近一段窗口的 Token** 的 KV Cache，就能让 LLM 处理无限长度的流式文本而不崩溃。

---

## 总结

KV Cache 是 LLM 走向工程化、商业化落地的核心技术之一。
*   **本质**：用空间换时间，避免自回归生成中的重复计算。
*   **代价**：造成了巨大的显存容量压力和内存带宽压力（Memory-Bound）。
*   **演进**：通过 GQA 减少体积，通过 PagedAttention 提升利用率，通过量化缓解带宽，通过窗口机制突破长度限制。

理解了 KV Cache，你也就拿到了大模型推理优化（Inference Optimization）的半张门票。在未来，随着长文本（Long Context）模型越来越普及（如 100k, 1M 上下文），如何更加优雅、高效地管理数十甚至数百 GB 的 KV Cache，依然会是 AI Infra 领域的黄金赛道。
