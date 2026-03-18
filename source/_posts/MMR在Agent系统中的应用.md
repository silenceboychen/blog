---
title: MMR在Agent系统中的应用
toc: true
date: 2026-03-17 17:43:36
categories: AI
tags: AI
mathjax: true
---

**MMR（Maximal Marginal Relevance，最大边际相关性）**是信息检索领域的经典算法，由 Carbonell 和 Goldstein 于 1998 年在 SIGIR 会议上提出。它的核心作用是**在检索结果中同时兼顾"相关性"（Relevance）和"多样性"（Diversity）**。

在以大语言模型（LLM）为核心的 Agent（智能体）系统中，由于模型的上下文窗口存在限制，如何向模型"喂"入最高质量、信息密度最大的外部知识至关重要。MMR 现已成为 Agent 系统和 RAG（检索增强生成）架构中不可或缺的高阶检索策略。

## 什么是 MMR？它的核心原理是什么？

在传统的向量检索中（如纯余弦相似度搜索），系统往往会返回与用户查询最相似的 Top-K 个文档。但这会导致一个致命问题：**信息冗余**。例如，用户查询"2026年AI的发展"，系统可能会返回 5 篇内容几乎完全相同的报道，虽然它们都高度相关，但没有提供额外的新信息。

**MMR 的目标就是打破这种冗余。** 它不仅要求候选文档与用户的问题相关，还要求候选文档与**已经选出的文档**尽可能不同。

### 核心数学公式

MMR 的本质是一个**贪心选择算子**，每轮从剩余候选集中挑出令下式最大的文档加入结果集。其正式定义为：

$$MMR = \arg\max_{D_i \in R \setminus S} \left[\lambda \cdot Sim_1(D_i, Q) - (1-\lambda) \cdot \max_{D_j \in S} Sim_2(D_i, D_j)\right]$$

各符号含义如下：

*   **$Q$**：用户的查询（Query）。
*   **$R$**：初始粗筛得到的候选文档集合（对应代码中的 `fetch_k`）。
*   **$S$**：已选定并放入最终结果的文档集合（初始为空）。
*   **$R \setminus S$**：$R$ 中尚未被选中的剩余文档，这是每轮迭代的选择范围。
*   **$Sim_1(D_i, Q)$**：**相关性得分**——候选文档与查询的相似度（通常为余弦相似度）。
*   **$Sim_2(D_i, D_j)$**：**冗余度得分**——候选文档与已选文档之间的最大相似度，衡量重复程度。$Sim_1$ 和 $Sim_2$ 可以是不同的相似度函数，实践中通常均采用余弦相似度。
*   **$\lambda$（Lambda）**：**平衡相关性与多样性的超参数（0 到 1 之间）**。
    *   当 $\lambda = 1$ 时，退化为纯相似度检索，不考虑多样性。
    *   当 $\lambda = 0$ 时，完全不考虑查询相关性，只选与已选文档最不相似的文档（极端情况，实际无意义）。
    *   通常在 Agent 系统中，$\lambda$ 默认设为 0.5 到 0.7 之间。

### 算法的迭代执行过程

MMR 并不是对所有文档一次性打分排序，而是一个**逐步贪心迭代**的过程，共执行 $k$ 轮：

1.  **粗筛**：从向量库中按纯相似度检索出 `fetch_k` 个候选文档，构成集合 $R$，初始化 $S = \emptyset$。
2.  **首选**：将 $R$ 中与 Query 余弦相似度最高的文档直接加入 $S$（第一个文档无需考虑多样性惩罚）。
3.  **迭代**：对 $R \setminus S$ 中每个剩余文档，计算其 MMR 得分；选出得分最高者加入 $S$。
4.  **重复步骤 3**，直到 $S$ 中已有 $k$ 个文档为止。
5.  将 $S$ 中的 $k$ 个文档作为最终检索结果交给 LLM。

---

## 为什么 Agent 系统极度需要 MMR？

1.  **上下文窗口（Context Window）寸土寸金**：
    LLM 的上下文长度有限（且越长计算成本越高）。如果检索到的背景知识全都是重复内容，就浪费了宝贵的 Token，导致 Agent 无法全面了解事件全貌，进而产生"幻觉"。

2.  **避免陷入"信息回音室"**：
    当 Agent 执行多步推理（Multi-hop Reasoning）或总结任务时，需要从多维度收集信息。MMR 能确保 Agent 看到一个问题的"正反两面"或"不同侧面"，而不是单方面的重复观点。

3.  **提升长期记忆（Long-term Memory）的召回质量**：
    Agent 拥有记忆库时，传统检索会把过去十次完全一样的历史记录都调出来。MMR 可以帮 Agent 召回那些"相关但包含不同细节"的历史记忆，为推理提供更广泛的参考。

---

## MMR 在 Agent 系统中的具体应用场景

### RAG 中的知识库检索

这是 MMR 最典型的应用场景。当 Agent 需要调用 `Search_KnowledgeBase` 工具时：

*   **传统做法**：从向量数据库中提取 5 个与 Query 距离最近的 Chunk（文本块），结果这 5 个块可能都来自同一篇文章的相邻段落。
*   **MMR 做法**：Agent 首先检索出 Top 20 个相关的 Chunk（`fetch_k`），然后通过 MMR 算法从中贪心筛选出最具差异性的 5 个 Chunk（`k`）交给 LLM。这样 Agent 就能在一轮检索中同时获得定义、案例、优缺点等不同维度的知识。

### 多文档摘要与分析 Agent

当 Agent 被要求"总结今天所有的科技新闻"时，如果使用纯相似度，Agent 可能会提取 10 篇关于同一款手机发布的通稿。引入 MMR 后，Agent 在抓取内容时会刻意避开相似度过高的文章，从而在摘要中同时涵盖手机发布、AI 突破、航空航天等多个不同的科技热点。

### Few-Shot Prompting 的动态样本选择

在高级 Agent 设计中，为了让 LLM 更好地使用外部工具，我们会动态从向量库中检索"历史成功调用工具的样例"（Few-shot examples）放入 Prompt 中。如果只用相似度，选出的可能全都是某种特定参数的调用样例。使用 MMR 可以确保 Prompt 样例在语境和参数上具备多样性，极大提升 Agent 举一反三的泛化能力。（LangChain 中的 `SemanticSimilarityExampleSelector` 也支持 MMR 模式。）

---

## 如何在 Agent 框架中应用 MMR？

在主流的 Agent 与 LLM 编排框架中（如 LangChain、LlamaIndex 等），MMR 已经被封装为开箱即用的模块。以下是基于 **LangChain v0.2+** 的完整示例：

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_core.documents import Document

# 初始化 Embedding 模型
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# 构造示例文档并写入向量库
docs = [
    Document(page_content="电动车锂电池的能量密度正在快速提升，主流乘用车续航已达 800km。"),
    Document(page_content="固态电池是下一代动力电池的主流技术路线，循环寿命和安全性更优。"),
    Document(page_content="宁德时代发布了新一代 4680 圆柱电池，能量密度达到 500Wh/kg。"),
    Document(page_content="电池热管理系统是影响快充速度和电池寿命的关键因素之一。"),
    Document(page_content="锂电池的回收与梯次利用已成为电动车行业可持续发展的重要议题。"),
]
vectorstore = Chroma.from_documents(docs, embedding=embeddings)

# ---- 传统检索（纯相似度）：容易出现冗余结果 ----
similarity_retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

# ---- MMR 检索：兼顾相关性与多样性 ----
mmr_retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 3,             # 最终交给 LLM 的文档数量
        "fetch_k": 15,      # 初始粗筛的候选文档数量（建议为 k 的 3~5 倍）
        "lambda_mult": 0.5  # λ=0.5：相关性与多样性各占一半
    }
)

# 在 RAG Chain 中使用 MMR Retriever
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template(
    "请根据以下背景知识，综合回答问题。\n\n背景知识:\n{context}\n\n问题: {question}"
)

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

rag_chain = (
    {"context": mmr_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | ChatOpenAI(model="gpt-4o")
    | StrOutputParser()
)

answer = rag_chain.invoke("请综合各方观点告诉我当前电动车的电池技术瓶颈")
print(answer)
```

**调参经验法则：**

*   `fetch_k` 建议设为 `k` 的 **3~5 倍**。过小则多样性空间有限，过大则引入过多低质量候选文档并增加计算延迟。
*   如果用户的问题需要**精准的特定事实**（如"某某人的出生日期"），多样性意义不大，将 $\lambda$ 调高（如 0.8~1.0）。
*   如果用户的问题是**开放性、探索性**的（如"分析中东局势的多种原因"），将 $\lambda$ 调低（如 0.3~0.5），引导检索从更多视角收集信息。

---

## MMR 与其他多样性检索方法的对比

| 方法 | 核心思路 | 优点 | 缺点 |
|---|---|---|---|
| **MMR** | 贪心迭代，每步惩罚已选文档的冗余度 | 实现简单，已集成主流框架，效果稳定 | λ 需人工调参，无法动态适应任务类型 |
| **阈值去重** | 过滤余弦相似度超过阈值的文档 | 极其简单，零额外参数 | 只能粗粒度去重，无法精细控制多样性程度 |
| **聚类采样** | 先对候选集聚类，再从每个簇取代表性文档 | 能保证空间分布的全局多样性 | 引入聚类算法的额外复杂度和超参数 |
| **DPP（行列式点过程）** | 用行列式衡量文档集合的体积（多样性的严格数学定义） | 多样性保证最严格，理论最优 | 计算成本远高于 MMR，工程集成复杂 |

在大多数生产级 RAG 场景中，**MMR 是性价比最高的选择**：效果足够好，延迟可控，主流框架开箱即用。

---

## MMR 的局限性

在将 MMR 引入 Agent 系统前，应了解其固有局限：

1.  **计算复杂度随 `fetch_k` 线性增长**：每轮迭代需遍历所有剩余候选文档，总复杂度约为 $O(k \times fetch\_k)$。当 `fetch_k` 超过 200 时，在高并发场景下延迟会显著上升。

2.  **λ 无法自适应调整**：当前实现中 λ 是静态超参数，无法根据用户查询的类型（事实类 vs. 开放类）动态调整。工程上通常需要针对不同任务分别配置。

3.  **效果上限由 Embedding 质量决定**：MMR 依赖余弦相似度判断文档间的冗余度。若所用 Embedding 模型对特定领域语义捕捉不准确，MMR 判断出的"多样"在语义上未必真正多样。

4.  **两阶段漏洞**：若初始粗筛（`fetch_k` 阶段）本身就漏掉了关键文档，MMR 无法将其召回——最终结果的上限由第一阶段的召回率决定。

---

## 总结

在 Agent 系统中，**信息检索的质量直接决定了 Agent 行动和回答的质量**。MMR 扮演了"内容策展人"（Curator）的角色，它在应用层过滤掉了冗余信息，将具有广度的高价值知识拼图交给 LLM。

理解 MMR 的数学本质（贪心 argmax 迭代）、合理配置 `fetch_k`、`k` 和 `λ`，并清楚认识其局限性，是实现从"玩具级 RAG"跨越到"生产级高可靠 Agent"的必备技术手段。

---

**参考文献**

> Carbonell, J., & Goldstein, J. (1998). The use of MMR, diversity-based reranking for reordering documents and producing summaries. *Proceedings of the 21st Annual International ACM SIGIR Conference on Research and Development in Information Retrieval (SIGIR '98)*, 335–336.
