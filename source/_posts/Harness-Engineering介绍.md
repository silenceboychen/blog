---
title: Harness Engineering介绍
toc: true
date: 2026-04-01 09:58:03
categories: AI
tags: AI
---


在2026年初的AI与软件工程领域，**Harness Engineering（驾驭工程/护栏工程）** 无疑是最具突破性和统治力的新范式。

它标志着AI辅助开发从早期的“对话式生成（Vibe Coding）”正式迈入了“生产级大规模应用”的成熟阶段。Harness Engineering 并不是一个空洞的流行语，而是决定 AI 智能体（AI Agents）能否在真实生产环境中稳定运行的核心架构层。

---

## 概念的诞生与核心定义

### 词源与诞生背景
“Harness”一词本意为马具（如缰绳、马鞍、马嚼子）。AI 模型就像一匹肌肉发达、速度极快的骏马，但如果没有马具的约束和引导，它只会横冲直撞。**马具并不削弱马的力量，而是将其转化为生产力。**

这一概念在2026年2月迎来了爆发式普及：
*   **Mitchell Hashimoto（HashiCorp联合创始人）**：在2月5日的博客《My AI Adoption Journey》中首创此词。他提出一个核心理念：*“当 AI 智能体犯错时，你不要去手动改代码，而是花时间去构建一个工程方案（更新规则或工具），确保它永远不再犯同样的错。我将此称为 Harness Engineering。”*
*   **OpenAI 官方下场**：2月11日，OpenAI 发布了轰动业界的工程博客《Harness engineering: leveraging Codex in an agent-first world》。他们披露了一个历时5个月的内部实验：一个初始仅3人的工程师团队，通过 Codex 智能体从零构建了**100万行级别**的生产代码，且**人类没有手写哪怕一行代码**。
*   **Martin Fowler 与 Thoughtworks**：软件工程泰斗 Martin Fowler 的团队随后发文总结了这一模式，正式确立了其在现代软件架构中的地位。

### 核心定义
**Agent = Model + Harness** （智能体 = 模型 + 驾驭环境）

Harness Engineering 指的是：**设计一套包含工具、约束、上下文反馈循环和可观测性的系统，将 AI 智能体包裹其中，使其能够在生产环境中可靠运行的工程学科。**
它解决了一个致命问题：大模型本身没有持续的记忆，且会自信地犯错；而 Harness 就是大模型的“操作系统”，负责管理上下文、执行标准工具调用并拦截错误。

---

## 核心方法论：从“在环内”到“在环上”

传统开发者修复 Bug 的方式是直接修改代码（Working **in** the loop）。
Harness Engineering 要求工程师进行思维转换：将精力放在改进系统上（Working **on** the loop）。

当 Claude Code 或 Codex 生成了错误的代码时：
*   **旧思路**：工程师接管键盘，重写那段代码。结果：下一次 AI 遇到同类问题依然会错。
*   **Harness 思路**：工程师去检查为什么 AI 会错。是缺少 API 文档？是缺少某个自动化测试工具？还是 `AGENTS.md` 里的系统提示词不够清晰？工程师通过完善这些“环境要素（Harness）”来修复问题，然后让 AI 重新生成。

---

## Harness Engineering 的三大支柱

根据 Thoughtworks 专家 Birgitta Böckeler 对 OpenAI 实践的总结，Harness Engineering 包含三个核心支柱：

### 上下文工程 (Context Engineering)
模型能输出什么，取决于它“看”到了什么。Harness 工程师需要构建动态的信息喂给体系。
*   **隐式提示词与 AGENTS.md**：放弃长篇大论的 Prompt，维护一个类似“目录”的 `AGENTS.md`（或 `CLAUDE.md`）文件。记录 AI 过去犯过的错和系统边界，通过持续迭代该文件来约束 AI。例如，不要每次都在对话框里跟模型抱怨“你又把时间格式写错了”，而是在 `AGENTS.md` 中写入规则：*“所有时间均默认使用 UTC 日期格式，并且调用 `DateUtil.ts`，禁用原生 Date 对象”*。一旦写入，这条 Harness 将永久约束模型。
*   **动态上下文注入**：为 AI 提供运行时的可观测性数据、甚至浏览器自动化测试的截图。让 AI 不仅能看懂静态代码，还能看到代码运行的动态结果。

### 架构约束与工具化 (Architectural Constraints & Tooling)
不能指望用自然语言求 AI 写出完美代码，必须用确定性的工具（Deterministic Tools）建立护栏。
*   **强硬的 Linter 与结构化测试**：为 AI 专门编写极度严格的代码检查工具。AI 拥有随心所欲生成代码的自由，但 Harness 系统拥有绝对的“否决权”。
*   **专门为 AI 准备的工具 API**：例如，提供带过滤器的测试运行器（只返回失败的测试日志，防止长日志撑爆 Context Window）。

### 熵管理与“垃圾回收” (Entropy Management)
AI 生成代码的速度极快，如果不加控制，代码库会迅速堆积技术债，陷入“软件熵增”。
*   **清洁工智能体**：在 Harness 中，设计专门在后台定期巡检的“垃圾回收”智能体。它们不负责写新功能，专门负责寻找文档与代码的不一致、检查架构分层违规、清理死代码，从而抵抗系统的腐化退化。

---

## 现代 Harness 系统的七大组件

根据 Cobus Greyling 等 AI 架构专家的梳理，一个生产级的 Harness 系统通常包含以下层级：

1.  **工具集成层 (Tool Integration Layer)**：连接模型与外部 API、数据库、文件系统（如 MCP - Model Context Protocol）。
2.  **记忆与状态管理 (Memory & State Management)**：管理多层记忆（短期工作上下文、会话状态、长期记忆），解决模型跨会话失忆的问题。
3.  **验证与护栏 (Verification and Guardrails)**：在 AI 提交代码/执行操作前，强制执行的确定性验证逻辑。
4.  **隔离与沙箱运行环境 (Sandboxed Environments)**：赋予了智能体调用工具和重构代码的权利后，为防止幻觉导致危险命令执行（如不可逆的删除或泄露密钥），Harness 系统必须提供轻量级的隔离层（如基于 Docker 或 WebAssembly 的沙箱），确保破坏性操作仅在受控环境中发生。
5.  **上下文控制的子智能体 (Sub-agents for Context Control)**：通过派生子智能体来完成特定任务，以防止主智能体的上下文窗口被中间步骤的噪音污染。
6.  **规划与分解 (Planning and Decomposition)**：引导大模型按结构化步骤执行，而非试图在一次推理中完成大系统。
7.  **人类审批流 (Human-in-the-loop)**：在关键破坏性操作前的高效人类拦截机制。

---

## 软件工程师的终极演进

正如 20 年前云计算的普及催生了 **DevOps** 工程师，2026年 AI 智能体的大规模落地催生了 **Harness 工程师**。

在未来，普通工程师与顶级工程师的分水岭在于：
*   **普通开发者**：像带实习生一样，每一步都盯着 AI，用自然语言微调代码，陷入疲惫的“人机结对编程”。
*   **Harness 工程师**：像架构师一样，构建坚不可摧的开发流水线、验证闭环和规则体系。他们信仰**“模型出错不是模型的问题，是 Harness 没建好”**。

**总结：**
Harness Engineering 彻底戳破了“AI 替代程序员”的低级叙事。人类不再是流水线上的打字机，而是系统环境的设计者。驾驭强大的 AI 不靠祈祷和魔法般的提示词，而是靠严密的工程约束。这就是 2026 年软件开发最硬核的事实。
