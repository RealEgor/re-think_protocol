# re!think it 协议 v1.0 

> **一个将概率性大语言模型 (LLM) 转化为确定性状态机 (Deterministic State Machine) 的认知框架。**

[English version](README.md) | [Русская версия](README_ru.md) | [中文说明](README_zh.md)

`re!think it` 是一套用于 LLM 注意力管理的系统级架构。该协议旨在防止长对话中的“注意力漂移”（Attention Drift），在生成响应前强制执行逻辑验证，并驱动模型输出高质量的非同质化内容（De-homogenized output）。

## ⚠️ 为什么常规提示词（Prompts）会失效？
现代前沿模型（如 GPT-4o, Claude 3.5, 以及国产的 DeepSeek, Qwen）都有一个共同的架构副产物：由于过度 RLHF 对齐，它们被训练出了“讨好型人格”（Sycophancy）。
* 当你用纯自然语言书写提示词时，模型会将其视为一种“软性建议”，而非“硬性约束”。
* 当任务中缺失关键事实时，模型为了不让用户失望，会本能地捏造信息（迎合性幻觉 / Sycophantic hallucination）。
* 在长上下文（Long Context，10-20 轮对话以上）中，自然语言的指令权重会发生衰减：模型会逐渐遗忘初始规则，退化到平庸的文字接龙模式（概率均值回归 / Regression to probability centroid）。

## 🧠 re!think it 高效运行的三个架构设计

该协议的核心由两个特定的逻辑方程组成，它们定义了“正确答案的推导公式”。这套机制足以让模型在不依赖外部代码的情况下，从底层阻断幻觉。

### 1. 抗稀释的结构化语法 (Context-dilution resistant syntax)
用纯自然语言解释规则在长程对话中极易失效。在 `re!think it` 中，指令是使用逻辑运算符和伪代码编写的。这种语法在注意力机制中具有更高的激活权重，物理上不留歧义空间。

### 2. 用“推理方程”替代传统引导 (Reasoning Equation)
协议摒弃了传统的 "Let's think step by step" 软引导，而是向模型注入了刚性的数学公式。模型接收到一个推理方程，将其生成逻辑转化为受控的求解过程。如果缺少关键变量，它会严格触发 `HARD STOP`（熔断机制）并向用户索要缺失的参数。

### 3. 任务类型的根本性解耦 (Architectural Task Decoupling)
AI 只有两类核心工作：**寻找精确答案**和**发散创意选项**。协议将它们解耦为两个独立的执行流：**Precision（精确模式）** 和 **Expansion（发散模式）**，彻底防止模型在需要绝对严谨时发挥创意，或在需要头脑风暴时陷入平庸。

---

## 📂 文件结构：发布版本列表
以下是各语言版本的文件列表：

* 🇨🇳 **[re-think_v1_zh.md](prompts/re-think_v1_zh.md)** / 🇬🇧 **[re-think_v1_en.md](prompts/re-think_v1_en.md)** / 🇷🇺 **[re-think_v1_ru.md](prompts/re-think_v1_ru.md)** (完整版)
  * **适用对象：** 大参数量基座模型（Claude 3.5 Sonnet, GPT-4o, Gemini 1.5 Pro, DeepSeek-V3）。
  * **特点：** 包含详细的逻辑描述、完整的推理方程与护栏规则。理想的自定义智能体（Custom Agents）底层核心。

* 🇨🇳 **[re-think_v1_zh_compact.md](prompts/re-think_v1_zh_compact.md)** / 🇬🇧 **[re-think_v1_en_compact.md](prompts/re-think_v1_en_compact.md)** / 🇷🇺 **[re-think_v1_ru_compact.md](prompts/re-think_v1_ru_compact.md)** (精简版)
  * **适用对象：** API 调用、端侧部署模型（Llama 3, Qwen-2.5）或对 Token 消耗/上下文窗口严格敏感的场景。
  * **特点：** 剥离了所有的解释性文本，仅保留硬性执行指令和压缩公式。

* 🎁 ⚙️ **[re-think_v1_pseudo_code.md](prompts/re-think_v1_pseudo_code.md)** (伪代码版 / Production Core 隐藏彩蛋)
  * **适用对象：** 硬核生产环境部署 (Hardcore production)、RAG 管道、多智能体编排器以及**极限 Token 优化**。
  * **特点：** 这是为架构师准备的专属机制。完全使用英语伪代码、集合论和逻辑运算符编写。由于 LLM 的分词器（Tokenizer）对英语代码的编码效率远高于汉字，该版本的 Token 消耗量**不仅大幅低于中文精简版，甚至比英文精简版还要低**。它以高密度语义指令的形式直接作用于模型的推理权重，提供最高的语义密度，无任何语言冗余。

---

## 🚀 快速上手

该协议不需要外部 Python 代码，是一个即插即用（Plug-and-play）的 `System Prompt` 注入方案。

**场景 1：标准对话界面 (ChatGPT, Claude, Gemini, DeepSeek)**
1. 复制所需协议文件的原始文本。
2. 进入 AI 的系统提示词设置（或在首条消息中写入：“请应用以下系统指令...”）。
3. 开始对话。模型将自动输出技术标头（Technical Header）、执行意图路由并求解方程。

**场景 2：自定义智能体 (GPTs / Coze / Claude Projects)**
将完整版协议粘贴到智能体的“Instructions / 指令”区块中。这是创建一个具备抗幻觉能力、严格遵循框架的 AI 助手的理想方式。

**场景 3：IDE 编程辅助 (Cursor, Windsurf, Copilot)**
将精简版协议 (`compact.md`) 或**伪代码版**的内容粘贴到项目根目录的 `.cursorrules` 文件中，即可约束 AI 程序员的生成逻辑，防止其输出模板化废话或幻觉库方法。

---

## 🛠 如何自定义与扩展协议

### 1. 注入用户画像与变量 (Persona Injection)
协议会在后台提取 4 个关键维度：$S_R$（专业水平）、$S_T$（信任栈）、$S_V$（业务红线）和 $S_F$（输出密度）。你可以直接在系统提示词的顶部硬编码这些变量，将其锁定为你的专属助理。

### 2. 自定义意图路由 (Custom Intent Routing)
你可以重写 `Router` 模块。明确定义触发条件：哪些业务任务必须强制走“精确模式”（Protocol A），哪些允许进入“发散模式”（Protocol B），以及哪些闲聊可以通过“旁路”（Bypass）直接响应。

### 3. 扩展验证门控 (Extend Verification Gating)
协议允许你设定专属的 `[VRF]` 通过标准：例如，强制要求在验证阶段进行安全漏洞审计（Security Audit）或特定的代码风格检查。

---

## ⚖️ 许可证
**re!think it Protocol** 采用 [CC BY 4.0](LICENSE) 许可证分发。  
*(c) 2026 Real_Egor.*
