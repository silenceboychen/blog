---
title: Lora为何能大幅减少训练参数
toc: true
date: 2026-02-12 14:53:30
categories: AI
tags: AI
mathjax: true
---

LoRA (Low-Rank Adaptation，低秩自适应) 能大幅减少可训练参数的核心原因在于它利用了**矩阵分解（Matrix Decomposition）**和**低秩（Low-Rank）假设**。

简单来说，LoRA 不去更新模型原本庞大的所有参数，而是通过训练两个非常小的矩阵来模拟参数的变化。

以下是详细的原理拆解：

## 核心思想：冻结主干 + 旁路更新

在传统的全量微调（Full Fine-tuning）中，如果预训练模型有一个权重矩阵 $W$，我们需要更新这个矩阵中的每一个数字。
*   **全量微调：** $W_{new} = W_{old} + \Delta W$。我们需要学习完整的 $\Delta W$（参数更新量）。

LoRA 的做法是：
1.  **冻结**预训练模型原本的权重 $W_{old}$（不参与训练，不占梯度显存）。
2.  在旁边增加一个“旁路”来模拟变化量 $\Delta W$。

## 数学原理：矩阵分解

这是 LoRA 省参数的关键。

假设模型某一层权重的维度是 $d \times k$（例如 $d=4096, k=4096$）。
*   如果要学习完整的 $\Delta W$，你需要训练 **$d \times k$** 个参数。

LoRA 假设：**参数的更新量 $\Delta W$ 不需要那么高的“秩”（Rank），它可以通过两个小矩阵相乘来近似。**

LoRA 将 $\Delta W$ 分解为两个小矩阵 $A$ 和 $B$ 的乘积：
$$ \Delta W = B \times A $$

其中：
*   $B$ 的维度是 $d \times r$
*   $A$ 的维度是 $r \times k$
*   **$r$ 是秩（Rank）**，这是一个我们可以设定的超参数，通常非常小（例如 4, 8, 16, 64）。

## 直观的算账对比

让我们用具体的数字来感受一下差距。

假设一个 Transformer 层的维度 $d = 4096$，我们也设 $k = 4096$。我们设定 LoRA 的秩 $r = 8$。

*   **全量微调（Full Fine-tuning）：**
    我们需要训练 $\Delta W$，参数量 = $4096 \times 4096 \approx$ **16,777,216 (1600万)** 个参数。

*   **LoRA 微调：**
    我们需要训练矩阵 $A$ 和 $B$。
    *   矩阵 $A$ ($8 \times 4096$) 参数量 = 32,768
    *   矩阵 $B$ ($4096 \times 8$) 参数量 = 32,768
    *   总参数量 = $32,768 + 32,768 =$ **65,536 (6.5万)** 个参数。

**结论：**
$$ \frac{65,536}{16,777,216} \approx 0.0039 $$
**LoRA 的参数量仅为全量微调的 0.4% 左右。** 这就是为什么它能“大幅”减少参数。

## 为什么这样做有效？

你可能会问：*“用这么少的参数去代替原来的大矩阵，效果不会变差吗？”*

这基于一个理论发现（Aghajanyan et al. 2020）：**大型预训练模型是过度参数化的，它们具有很低的“本征维度”（Intrinsic Dimension）。**

意思是说，虽然模型有千亿参数，但当我们为了某个特定任务（比如写代码或把英语翻译成中文）去微调它时，真正起作用、发生变化的参数空间其实非常小。所有的变化都可以被压缩到一个低秩空间里，而不会丢失太多信息。LoRA 正是利用了这个特性。

## LoRA 应用于模型的哪些层？

在实际应用中，LoRA 并不是对模型的所有参数都进行低秩分解，而是有选择性地应用于特定的模块。

**在 Transformer 架构中，LoRA 通常应用于：**

1.  **Attention 层的投影矩阵**：
    *   $W_Q$（Query 投影矩阵）
    *   $W_K$（Key 投影矩阵）
    *   $W_V$（Value 投影矩阵）
    *   $W_O$（Output 投影矩阵）

2.  **前馈神经网络（FFN）层**：
    *   第一个线性层（up projection）
    *   第二个线性层（down projection）

**不同的配置策略：**

*   **最小配置**（节省参数）：只对 $W_Q$ 和 $W_V$ 应用 LoRA
*   **标准配置**（平衡效果）：对所有 Attention 投影矩阵（Q、K、V、O）应用 LoRA
*   **完整配置**（最佳效果）：对 Attention 层和 FFN 层的所有线性层应用 LoRA

选择不同的 `target_modules` 会直接影响：
*   可训练参数的总量
*   微调效果的好坏
*   训练和推理的速度

原论文的实验表明，**只对 Attention 层的 Q 和 V 矩阵应用 LoRA 就能达到很好的效果**，这也是最常用的配置。

## 代码实现示例

理论讲完后，让我们看看如何在实际项目中使用 LoRA。这里使用 Hugging Face 的 PEFT 库（Parameter-Efficient Fine-Tuning）。

### 安装依赖

```bash
pip install transformers peft datasets accelerate
```

### 基础使用示例

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model, TaskType

# 1. 加载预训练模型
model_name = "meta-llama/Llama-2-7b-hf"
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.float16,
    device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained(model_name)

# 2. 配置 LoRA
lora_config = LoraConfig(
    r=8,                                    # LoRA 的秩（Rank）
    lora_alpha=32,                          # Scaling factor，通常设为 r 的 2-4 倍
    target_modules=["q_proj", "v_proj"],    # 应用 LoRA 的目标模块
    lora_dropout=0.1,                       # Dropout 概率，防止过拟合
    bias="none",                            # 是否训练 bias，通常设为 "none"
    task_type=TaskType.CAUSAL_LM            # 任务类型
)

# 3. 将 LoRA 应用到模型
model = get_peft_model(model, lora_config)

# 4. 查看可训练参数
model.print_trainable_parameters()
# 输出示例：trainable params: 4,194,304 || all params: 6,742,609,920 || trainable%: 0.0622
```

### 训练 LoRA 模型

```python
from transformers import Trainer, TrainingArguments

# 训练配置
training_args = TrainingArguments(
    output_dir="./lora_output",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    learning_rate=3e-4,                  # LoRA 可以使用较高的学习率
    logging_steps=10,
    save_strategy="epoch",
    optim="adamw_torch",
)

# 创建 Trainer 并训练
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset,
)

trainer.train()

# 保存 LoRA 权重（只保存 A 和 B 矩阵，通常只有几 MB）
model.save_pretrained("./lora_weights")
```

### 加载和使用 LoRA 权重

```python
from peft import PeftModel

# 加载基础模型
base_model = AutoModelForCausalLM.from_pretrained(model_name)

# 加载 LoRA 权重
model = PeftModel.from_pretrained(base_model, "./lora_weights")

# 推理
inputs = tokenizer("你的输入文本", return_tensors="pt")
outputs = model.generate(**inputs, max_length=100)
print(tokenizer.decode(outputs[0]))

# 如果要合并 LoRA 权重到基础模型（用于部署）
model = model.merge_and_unload()
model.save_pretrained("./merged_model")
```

## 超参数选择指南

选择合适的超参数对 LoRA 的效果至关重要。以下是基于实践经验的推荐配置：

### 核心超参数表

| 参数 | 推荐值 | 说明 | 影响 |
|------|--------|------|------|
| **r (rank)** | 4-64 | 秩的大小，决定了低秩矩阵的维度 | 越大效果越好但参数越多，小任务用 4-8，复杂任务用 16-64 |
| **lora_alpha** | 16-64 | Scaling factor，通常是 r 的 2-4 倍 | 控制 LoRA 更新的强度，alpha/r 越大，LoRA 的影响越大 |
| **lora_dropout** | 0.05-0.1 | Dropout 概率 | 防止过拟合，数据集小时可以设高一点 |
| **target_modules** | ["q_proj", "v_proj"] | 应用 LoRA 的模块 | 越多效果越好但参数越多，最常用的是 Q+V |
| **learning_rate** | 1e-4 ~ 3e-4 | 学习率 | LoRA 可以用比全量微调高 1-2 个数量级的学习率 |

### 根据任务类型选择配置

**小规模任务**（如情感分析、文本分类，数据量 < 10K）：
```python
LoraConfig(
    r=4,
    lora_alpha=16,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.1,
)
```

**中等规模任务**（如指令微调、对话系统，数据量 10K-100K）：
```python
LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
    lora_dropout=0.05,
)
```

**大规模任务**（如领域适应、知识注入，数据量 > 100K）：
```python
LoraConfig(
    r=64,
    lora_alpha=128,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
)
```

### Rank (r) 的选择经验法则

*   **r = 4**：适合简单的风格迁移、格式调整任务
*   **r = 8**：最常用的默认值，适合大多数微调任务
*   **r = 16**：适合需要学习新领域知识的任务
*   **r = 32 或 64**：适合复杂任务或当 r=8/16 效果不理想时

**重要提示**：增大 r 并不总是能带来更好的效果，有时反而会导致过拟合。建议从小的 r 开始实验，逐步增大。

## LoRA 的局限性与适用场景

虽然 LoRA 非常强大，但它并不是万能的。了解它的局限性和适用场景，可以帮助你做出更好的技术选择。

### ✅ LoRA 适合的场景

1.  **领域适应（Domain Adaptation）**
    *   将通用语言模型适配到特定领域（如医疗、法律、金融）
    *   模型已经具备基础能力，只需要学习领域特定的术语和知识

2.  **指令微调（Instruction Tuning）**
    *   教会模型遵循特定的指令格式
    *   调整模型的输出风格和行为模式

3.  **风格迁移**
    *   改变文本生成的风格（如正式/非正式、简洁/详细）
    *   调整语气和表达方式

4.  **多任务学习**
    *   为同一个基础模型训练多个 LoRA 权重，分别对应不同任务
    *   快速切换不同的任务能力

5.  **资源受限环境**
    *   只有单张消费级显卡（如 RTX 3090/4090）
    *   需要同时维护多个任务的模型版本

### ❌ LoRA 不适合的场景

1.  **学习全新能力**
    *   预训练模型完全不具备的能力（如让纯英文模型学习中文）
    *   需要大幅改变模型的知识结构
    *   **为什么？** LoRA 的低秩假设认为变化发生在低维子空间，但学习全新能力可能需要高维空间的变化

2.  **预训练模型与任务差距过大**
    *   使用对话模型去做代码生成
    *   使用文本模型去做图像理解
    *   **建议**：选择与任务更匹配的预训练模型

3.  **需要修改模型架构**
    *   添加新的层或模块
    *   改变注意力机制
    *   **解决方案**：考虑使用 Adapter 或直接修改模型架构

4.  **极小数据集**
    *   训练样本少于 100 条
    *   **为什么？** 即使是 LoRA，也需要足够的数据来学习有意义的模式
    *   **建议**：考虑 Few-shot Learning 或 Prompt Engineering

### 实际效果对比

根据原论文（Hu et al., 2021）在多个任务上的实验结果：

| 任务 | 模型 | 全量微调效果 | LoRA 效果 | LoRA 参数占比 |
|------|------|--------------|-----------|---------------|
| MNLI（自然语言推理） | RoBERTa-base | 90.2% | 90.1% | 0.3% |
| SST-2（情感分析） | RoBERTa-base | 96.4% | 96.3% | 0.3% |
| WikiSQL（文本到SQL） | GPT-3 175B | 73.8% | 73.4% | 0.01% |
| SAMSum（摘要） | GPT-3 175B | 52.0% | 53.8% | 0.01% |

**关键发现**：
*   在大多数任务上，LoRA 的效果与全量微调**几乎相同**
*   在某些生成任务上，LoRA 甚至**超过**全量微调（可能因为全量微调容易过拟合）
*   参数量只有全量微调的 **0.01% - 0.3%**

## 与其他参数高效微调（PEFT）方法的对比

LoRA 是众多参数高效微调方法中的一种。了解不同方法的特点，可以帮助你选择最适合的方案。

### 主流 PEFT 方法对比表

| 方法 | 参数量 | 推理开销 | 训练难度 | 主要优势 | 主要劣势 | 适用场景 |
|------|--------|----------|----------|----------|----------|----------|
| **LoRA** | 0.1-1% | **无**（可合并） | 低 | 参数少、无推理开销、易于切换 | 不适合学习全新能力 | **通用，性价比最高** |
| **Adapter** | 2-4% | 有（增加层） | 中 | 可以插入不同位置、模块化 | 推理时有延迟 | 需要保留多任务切换能力 |
| **Prefix-Tuning** | <1% | 有（占用序列长度） | 高（训练不稳定） | 参数极少 | 训练困难、占用输入长度 | 自然语言生成任务 |
| **P-Tuning v2** | <1% | 有（占用序列长度） | 高 | 改进的 Prefix-Tuning | 同 Prefix-Tuning | NLU + NLG 任务 |
| **Prompt Tuning** | <0.01% | 有（占用序列长度） | 高 | 参数极少 | 需要大模型（>10B）才有效 | 超大模型的轻量适配 |
| **BitFit** | <0.1% | **无** | 低 | 极简单 | 效果通常不如 LoRA | 快速验证想法 |
| **Full Fine-tuning** | **100%** | **无** | 低 | 效果最好（理论上） | 成本高、易过拟合 | 预算充足、数据充足 |

### 各方法的技术细节

#### 1. **LoRA**（本文重点）
*   **核心思想**：低秩矩阵分解 $\Delta W = B \times A$
*   **特点**：可在推理时合并权重，不增加延迟

#### 2. **Adapter**（适配器）
*   **核心思想**：在 Transformer 层之间插入小型的全连接层
*   **结构**：Down-projection（降维）→ 激活函数 → Up-projection（升维）
*   **优势**：模块化，可以针对不同任务插入不同的 Adapter
*   **劣势**：推理时需要经过额外的层，有轻微延迟

#### 3. **Prefix-Tuning / P-Tuning**
*   **核心思想**：在输入序列前添加可学习的"虚拟 token"（前缀）
*   **特点**：不修改模型参数，只学习前缀的嵌入
*   **劣势**：训练不稳定，且前缀会占用输入序列的长度

#### 4. **Prompt Tuning**
*   **核心思想**：只学习 Soft Prompt 的嵌入向量
*   **特点**：参数量极少（只有几千到几万个参数）
*   **限制**：只在超大模型（>10B 参数）上有效

#### 5. **BitFit**
*   **核心思想**：只训练模型的 bias 项，冻结所有其他参数
*   **特点**：极其简单，但效果通常不如 LoRA

### 选择建议

*   **首选 LoRA**：如果没有特殊需求，LoRA 是性价比最高的选择
*   **选择 Adapter**：如果需要在推理时动态切换多个任务，且可以接受轻微的推理延迟
*   **选择 Prefix-Tuning**：如果是纯生成任务（如摘要、翻译），且有调参经验
*   **选择 Full Fine-tuning**：如果有充足的计算资源，且数据集很大（>100K 样本）

## LoRA 的变种与最新进展

LoRA 的成功激发了大量后续研究，出现了多种改进版本。

### **QLoRA**（Quantized LoRA）

**核心创新**：将 4-bit 量化与 LoRA 结合，进一步降低显存需求。

**技术要点**：
*   **NF4（4-bit NormalFloat）量化**：专门为正态分布的权重设计的 4-bit 数据类型
*   **双重量化（Double Quantization）**：连量化参数本身也进行量化
*   **分页优化器（Paged Optimizers）**：使用 CPU 内存作为缓冲，避免显存溢出

**实际效果**：
*   在单张 24GB 显卡（如 RTX 3090）上微调 **65B 参数**的模型
*   在单张 48GB 显卡（如 A6000）上微调 **33B 参数**的模型，效果接近全量微调

**代码示例**：
```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
from peft import prepare_model_for_kbit_training, LoraConfig, get_peft_model

# 4-bit 量化配置
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,
)

# 加载量化模型
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-70b-hf",
    quantization_config=bnb_config,
    device_map="auto"
)

# 准备模型以进行 k-bit 训练
model = prepare_model_for_kbit_training(model)

# 应用 LoRA
lora_config = LoraConfig(r=16, lora_alpha=32, ...)
model = get_peft_model(model, lora_config)
```

### **AdaLoRA**（Adaptive LoRA）

**核心创新**：自适应地为不同层分配不同的 rank，而不是所有层使用相同的 r。

**技术原理**：
*   使用 **奇异值分解（SVD）** 来评估每一层的"重要性"
*   重要的层分配更高的 rank，不重要的层降低 rank
*   训练过程中动态调整 rank 分配

**优势**：
*   参数预算固定的情况下，效果优于固定 rank 的 LoRA
*   自动发现哪些层对任务更重要

**适用场景**：
*   对性能要求极致，愿意付出更多训练时间
*   需要理解模型的哪些部分对特定任务最重要

### **DoRA**（Weight-Decomposed LoRA）

**核心创新**：将权重矩阵分解为 **幅度（Magnitude）** 和 **方向（Direction）** 两个部分。

**数学表示**：
$$ W' = m \frac{W + \Delta W}{||W + \Delta W||} $$

其中：
*   $m$ 是幅度（标量）
*   $\frac{W + \Delta W}{||W + \Delta W||}$ 是方向（单位向量）
*   只对方向部分应用 LoRA

**实验结果**：
*   在 Commonsense Reasoning、视觉指令微调等任务上**超越 LoRA**
*   特别是在需要精细调整的任务上，效果提升明显

**代码示例**：
```python
from peft import LoraConfig

lora_config = LoraConfig(
    r=8,
    lora_alpha=16,
    use_dora=True,  # 启用 DoRA
    target_modules=["q_proj", "v_proj"],
)
```

### **LoRA+**（LoRA with Differential Learning Rates）

**核心创新**：为矩阵 A 和 B 设置**不同的学习率**。

**技术细节**：
*   矩阵 B 使用较高的学习率（如 $\eta_B = 2e-4$）
*   矩阵 A 使用较低的学习率（如 $\eta_A = \eta_B / 16$）
*   **原理**：矩阵 B 在初始化时为零，需要更快地学习；矩阵 A 已有随机初始化，应更保守地更新

**效果**：
*   在相同的训练步数下，收敛更快
*   最终效果略优于标准 LoRA

### 5. **其他变种**

*   **MultiLoRA**：为同一模型的不同层使用不同的 rank
*   **LoRA-FA（Frozen-A）**：训练过程中冻结矩阵 A，只训练矩阵 B
*   **VeRA**（Vector-based LoRA）：用向量代替矩阵，进一步减少参数

### 选择建议

*   **默认选择**：标准 LoRA（简单、稳定、效果好）
*   **显存受限**：QLoRA（在消费级显卡上微调大模型）
*   **追求极致性能**：DoRA 或 AdaLoRA（效果提升，但实现更复杂）
*   **快速验证**：LoRA+（更快收敛，适合快速实验）

## LoRA 的初始化策略与训练技巧

正确的初始化和训练策略对 LoRA 的效果至关重要。以下是原论文和实践中总结的技巧。

### 初始化策略

LoRA 使用了一个巧妙的初始化方式，确保训练开始时模型的行为与预训练模型**完全一致**。

#### 矩阵 A 的初始化
*   **方法**：**高斯随机初始化** $A \sim \mathcal{N}(0, \sigma^2)$
*   **标准差**：通常 $\sigma = 1 / r$（r 是 rank）
*   **代码**：`nn.init.kaiming_uniform_(A, a=math.sqrt(5))`

#### 矩阵 B 的初始化
*   **方法**：**零初始化** $B = 0$
*   **目的**：确保训练开始时 $\Delta W = B \times A = 0$，模型行为与预训练模型一致
*   **代码**：`nn.init.zeros_(B)`

#### Scaling Factor
LoRA 的更新不是直接 $\Delta W = B \times A$，而是：

$$ \Delta W = \frac{\alpha}{r} \times B \times A $$

其中：
*   $\alpha$ 是 `lora_alpha`（超参数，通常设为 16、32 或 64）
*   $r$ 是 rank
*   $\frac{\alpha}{r}$ 是 scaling factor

**为什么需要 Scaling？**
*   保持不同 rank 之间的更新幅度一致
*   类似于学习率的作用，控制 LoRA 对模型的影响程度
*   实验表明，$\alpha = 2r$ 或 $\alpha = 4r$ 通常效果最好

### 训练技巧

#### 1. 学习率设置
*   **LoRA 可以使用比全量微调高 10-100 倍的学习率**
*   推荐范围：$1 \times 10^{-4}$ 到 $5 \times 10^{-4}$
*   全量微调通常用：$1 \times 10^{-5}$ 到 $5 \times 10^{-5}$

**为什么 LoRA 能用更高学习率？**
*   LoRA 只更新很小一部分参数
*   预训练模型的权重被冻结，不会被破坏
*   即使 LoRA 部分过度更新，也可以通过降低 alpha 来减弱影响

#### 2. 避免过拟合

*   **使用 Dropout**：设置 `lora_dropout=0.05` 或 `0.1`
*   **Early Stopping**：监控验证集损失，及时停止训练
*   **数据增强**：对于小数据集，使用数据增强技术
*   **降低 Rank**：如果发现过拟合，尝试降低 r（如从 16 降到 8）

#### 3. 优化器选择

*   **推荐**：AdamW（最常用）
*   **也可尝试**：
    *   Adafactor（节省显存，但收敛可能较慢）
    *   Lion（最新的优化器，在某些任务上表现更好）

**AdamW 配置**：
```python
optimizer = torch.optim.AdamW(
    model.parameters(),
    lr=3e-4,
    betas=(0.9, 0.999),
    eps=1e-8,
    weight_decay=0.01  # 权重衰减有助于防止过拟合
)
```

#### 4. 批次大小与梯度累积

*   **有效批次大小**（Effective Batch Size）= `per_device_batch_size × gradient_accumulation_steps × num_gpus`
*   推荐有效批次大小：32-128
*   如果显存不足，减小 `per_device_batch_size`，增大 `gradient_accumulation_steps`

**示例**：
```python
training_args = TrainingArguments(
    per_device_train_batch_size=4,     # 单卡批次大小
    gradient_accumulation_steps=8,     # 累积 8 步再更新
    # 有效批次大小 = 4 × 8 = 32
)
```

#### 5. 学习率调度器

*   **推荐**：Cosine Annealing with Warmup
*   **Warmup 步数**：总步数的 3-10%
*   **最小学习率**：初始学习率的 0.1 倍

```python
from transformers import get_cosine_schedule_with_warmup

scheduler = get_cosine_schedule_with_warmup(
    optimizer,
    num_warmup_steps=100,
    num_training_steps=1000
)
```

#### 6. 混合精度训练

*   使用 `fp16` 或 `bf16`（BF16 更稳定，但需要较新的 GPU）
*   可以进一步节省显存和加速训练

```python
training_args = TrainingArguments(
    fp16=True,  # 或 bf16=True（对于 A100、H100 等）
    ...
)
```

### 调试技巧

#### 检查 LoRA 是否正常工作

```python
# 1. 检查可训练参数
model.print_trainable_parameters()

# 2. 检查哪些参数被冻结
for name, param in model.named_parameters():
    if param.requires_grad:
        print(f"Trainable: {name}")

# 3. 检查 LoRA 的权重是否在更新
# 在训练前后打印 LoRA 权重的范数
for name, module in model.named_modules():
    if "lora" in name.lower():
        if hasattr(module, 'weight'):
            print(f"{name}: {module.weight.norm().item()}")
```

#### 常见问题排查

1.  **损失不下降**：
    *   检查学习率是否太小（尝试增大 10 倍）
    *   检查是否正确设置了 `target_modules`
    *   检查数据预处理是否正确

2.  **过拟合严重**：
    *   增大 `lora_dropout`
    *   降低 `r`（rank）
    *   增加训练数据或使用数据增强

3.  **显存溢出**：
    *   降低 `per_device_batch_size`
    *   增大 `gradient_accumulation_steps`
    *   使用 `gradient_checkpointing`
    *   考虑使用 QLoRA

## 总结：LoRA 的优势

1.  **训练参数极少**：通常只有原模型的 1/1000 甚至更少。
2.  **显存占用低**：因为不需要为原本的大模型权重存储优化器状态（Optimizer States），只需为那两个小矩阵存储，这使得在单张消费级显卡（如 RTX 3090/4090）上微调大模型成为可能。
3.  **推理无延迟**：在推理（Inference）阶段，可以将训练好的 $B \times A$ 直接加回原权重 $W_{old}$ 中。这样模型架构没有变化，推理速度和原模型一样快。
4.  **易于切换**：因为 LoRA 文件很小（几 MB 到几百 MB），你可以保留一个基础大模型，然后针对不同任务挂载不同的 LoRA 模块，不需要为每个任务都存一个几十 GB 的大模型。
5.  **效果接近全量微调**：在大多数任务上，LoRA 能达到与全量微调相当的效果，在某些任务上甚至更好。
6.  **生态系统成熟**：有完善的库（PEFT、Diffusers）和社区支持，大量预训练的 LoRA 权重可以直接使用。

## 参考资料

*   原论文：[LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685) (Hu et al., 2021)
*   QLoRA 论文：[QLoRA: Efficient Finetuning of Quantized LLMs](https://arxiv.org/abs/2305.14314) (Dettmers et al., 2023)
*   Hugging Face PEFT 库：[https://github.com/huggingface/peft](https://github.com/huggingface/peft)
*   低本征维度论文：[Intrinsic Dimensionality Explains the Effectiveness of Language Model Fine-Tuning](https://arxiv.org/abs/2012.13255) (Aghajanyan et al., 2020)