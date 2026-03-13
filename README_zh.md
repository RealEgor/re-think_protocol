# re!think it 协议 v1.0 

> **一个将概率性大语言模型 (LLM) 转换为确定性状态机 (State Machine) 的认知框架。**

[English version](README.md) | [Русская версия](README_ru.md) | [中文说明](README_zh.md)

`re!think it` 是一套用于 LLM 注意力管理的系统架构。该协议旨在防止长对话中的“注意力漂移”（Attention Drift），在生成响应前强制验证逻辑，并驱动模型产生非平庸的创意。

## ⚠️ 为什么常规提示词（Prompts）会失效？
现代大模型（如 GPT-4, Claude 3, 以及国产的 DeepSeek, Qwen）都有一个共同的缺陷：它们被训练得过于“礼貌”且“乐于助人”（RLHF 效应）。
* 当你用自然语言书写提示词时，模型会将其视为一种“礼貌的建议”，而非硬性规则。
* 当任务中缺失关键数据时，模型为了不因拒绝而让你失望，会本能地尝试捏造信息（顺从性幻觉）。
* 在长对话（10-20 轮以上）中，自然语言的语义引力会稀释：模型会逐渐忘记初始规则，退化到平庸的回答模式（概率质心回归）。

## 🧠 re!think it 高效运行的三个核心秘密

该协议的核心由两个特定的方程组成，它们定义了“正确答案的公式”。这两个方程足以让模型在不依赖外部代码的情况下停止幻觉。

### 1. 程序化语法不会随上下文稀释
用纯自然语言解释规则的效果很差。在 `re!think it` 中，指令是使用逻辑运算符和伪代码编写的。这种语法不会被后续对话模糊，物理上不留歧义空间。

### 2. 用“推理方程”替代传统引导
协议没有简单地要求模型“一步步思考”，而是给了它一个数学公式。模型接收到一个推理方程，将其生成逻辑转变为受控格式。如果缺少变量，它会严格执行 `HARD STOP`（强制停止）并索要这些变量。

### 3. 任务类型的根本性分离
AI 只有两类工作：**寻找精确答案**和**发散创意选项**。协议将它们分为两个独立的方程：**Precision（精确）** 和 **Expansion（发散）**，防止模型在需要准确时发挥创意，或在需要发散时陷入平庸。

---

## 📂 文件结构：完整版 vs 精简版
`prompts/` 文件夹包含 4 个文件：

* 🇨🇳 **[re-think_v1_zh.md](prompts/re-think_v1_zh.md)** / 🇬🇧 **[re-think_v1_en.md](prompts/re-think_v1_en.md)** / 🇷🇺 **[re-think_v1_ru.md](prompts/re-think_v1_ru.md)** (完整版)
  * **适用对象：** 强力模型（Claude 3.5 Sonnet, GPT-4o, Gemini 1.5 Pro, DeepSeek-V3）。
  * **特点：** 包含详细的逻辑描述和完整的推理方程。

* 🇨🇳 **[re-think_v1_zh_compact.md](prompts/re-think_v1_zh_compact.md)** / 🇬🇧 **[re-think_v1_en_compact.md](prompts/re-think_v1_en_compact.md)** / 🇷🇺 **[re-think_v1_ru_compact.md](prompts/re-think_v1_ru_compact.md)** (精简版)
  * **适用对象：** API 调用、本地模型（Llama 3, Qwen-2.5）或严格的 Token 节省场景。
  * **特点：** 删除了所有解释，仅保留硬性指令和压缩公式。

---

## 🚀 快速上手

该协议不需要外部代码，是一个即插即用的 `System Prompt` 注入方案。

**场景 1：标准对话 (ChatGPT, Claude, Gemini, DeepSeek)**
1. 复制所需文件的原始文本。
2. 进入 AI 设置（或在首条消息中写入：“请采用以下系统指令...”）。
3. 开始对话。模型将自动输出技术页眉、分流任务并求解方程。

**场景 2：自定义 Agent (GPTs / Coze / Claude Projects)**
将完整版协议粘贴到“指令”区块中。这是创建严格遵循框架的智能助手的理想方式。

**场景 3：IDE (Cursor, Windsurf, Copilot)**
将精简版协议 (`compact.md`) 粘贴到项目根目录的 `.cursorrules` 文件中。

---

## 🛠 如何自定义和个性化协议

### 1. 调整用户画像与优先级
协议提取 4 个关键维度：$S_R$（能力）、$S_T$（信任方法）、$S_V$（硬约束）和 $S_F$（格式）。你可以直接在系统提示词中为自己硬编码这些变量。

### 2. 自定义分流规则 (Routing)
你可以重写 `Router` 模块。明确定义触发器：哪些任务必须用“精确模式”（Protocol A），哪些需要“发散模式”（Protocol B），以及哪些任务通过“旁路”（Bypass）直接执行。

### 3. 扩展验证层
协议允许你设定自己的通过标准：例如强制性的代码漏洞检查或特定的语调对齐。

---

## ⚖️ 许可证
**re!think it Protocol** 采用 [CC BY 4.0](LICENSE) 许可证分发。  
*(c) 2026 Real_Egor.*
