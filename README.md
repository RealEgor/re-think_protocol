# re!think it Protocol v2.0 

> **A cognitive framework that transforms a probabilistic Large Language Model (LLM) into a deterministic State Machine.**

[English version](README.md) | [Русская версия](README_ru.md)

`re!think it` is a system architecture for LLM attention management. The protocol protects against context loss during long conversations (Attention Drift), forcefully verifies the model's logic before generating a response, and compels it to produce non-cliché ideas.

## ⚠️ Why do standard prompts stop working?
Modern LLMs (GPT-4, Claude 3, Gemini) share a fundamental vulnerability: they are trained to be "polite" and "helpful" (the RLHF effect). 
* When you write a prompt in plain human language (*"be a strict critic, do not invent facts"*), the model treats it as a polite request, not a hard law. 
* Faced with missing data in your task, the model instinctively tries to invent it (hallucination of usefulness) to avoid disappointing you with a refusal. 
* Over long distances (10-20+ messages), natural language dilutes semantic gravity: the model forgets the initial rules and regresses to average, banal answers (the probabilistic centroid).

## 🧠 3 secrets behind re!think it's efficiency

At its core, this protocol consists of two specific equations that define the "formula for a correct answer" for the model. On their own, they are sufficient to make the model deliver high-quality results and stop hallucinating out of a basic desire to "be helpful." The rest of the system harness (KV-Cache Anchoring, SEQ numeration) ensures these equations don't just work in a vacuum, but remain stable over long dialogues.

The architecture relies on three fundamental principles:

### 1. Programmatic syntax doesn't dilute with context
Attempts to explain rules in plain human language perform poorly. In `re!think it`, instructions are written using logical operators and pseudo-code — structures the models were trained on and understand orders of magnitude better. Unlike text requests, programmatic syntax isn't blurred by subsequent conversation, is sustained much longer by the model, and physically leaves no room for double interpretation.

### 2. A "Reasoning Equation" instead of prompting
Instead of asking the model to "think step by step," the protocol gives it a mathematical formula for the correct solution. This tunes the reasoning logic at the level of pure formality, without diving into actual programming code. The model receives a "reasoning equation" that shifts its habitual step-by-step generation into a higher-quality, controlled format. If it lacks specific variables to solve this equation, it strictly halts and requests exactly those variables, because it has a clear formula for finding the answer.

### 3. Fundamental separation of task types
At a basic level, there are only two types of AI work: **finding an exact answer** and **brainstorming options**. Any attempt to write a universal "one-size-fits-all" instruction is doomed to fail, as these tasks have diametrically opposed solution formulas. The protocol physically separates them into two independent equations (Precision and Expansion), preventing the model from applying creativity where accuracy is needed, and from slipping into banality where divergence is required.

---

## 📂 File Structure: Full vs. Compact Versions
The `prompts/` folder contains 4 files. Choose the one that fits your workflow:

* 🇬🇧 **[re-think_v1_en.md](prompts/re-think_v1_en.md)** / 🇷🇺 **[re-think_v1_ru.md](prompts/re-think_v1_ru.md)** (Full versions)
  **For whom:** Powerful models (Claude 3.5 Sonnet/Opus, GPT-4o, Gemini 1.5 Pro).
  **Feature:** Contains detailed formulations and full "reasoning equations" with logic descriptions. Powerful models read this structure and produce phenomenally deep answers. Perfect as the core of specialized Custom Agents.

* 🇬🇧 **[re-think_v1_en_compact.md](prompts/re-think_v1_en_compact.md)** / 🇷🇺 **[re-think_v1_ru_compact.md](prompts/re-think_v1_ru_compact.md)** (Compact versions)
  **For whom:** API usage, local models (Llama 3, Mistral), or strict token economy.
  **Feature:** Explanations are cut out; only harsh, dry imperatives and highly compressed formulas remain ("Do this. Violation = fatal hallucination"). Runs faster, minimizes cognitive load on the model.

---

## 🚀 How to use (Quick Start)

The protocol requires no external code or complex integrations. It is a ready-made `System Prompt` injection.

**Scenario 1: Standard chats (ChatGPT, Claude, Gemini)**
1. Copy the raw text of the desired file.
2. Go to the AI settings (Custom Instructions in ChatGPT, or just write in your first prompt for Claude/Gemini: *"Adopt the following system instructions..."*).
3. Start chatting. The model will automatically output Technical Headers, route tasks, and solve equations.

**Scenario 2: Custom Agents (GPTs / Claude Projects)**
Create a new agent, paste the full protocol version into the "Instructions" block, and save. This is the ideal way to create a dedicated "smart" assistant strictly following the framework.

**Scenario 3: IDEs (Cursor, Windsurf, Copilot)**
Paste the compact protocol version (`compact.md`) into a `.cursorrules` or `.windsurfrules` file at the root of your project. Your AI programmer will stop writing boilerplate code and hallucinating non-existent library methods.

---

## 🛠 How to customize and individualize the protocol

`re!think it` is designed as an open modular system. You can easily adapt the logic to your workflows:

### 1. Tuning the User Map and Priorities
The protocol extracts 4 key profile components early on: $S_R$ (competence), $S_T$ (experience/trusted methods), $S_V$ (hard constraints), and $S_F$ (format).
* **Basic level:** You can hardcode these variables directly into the system prompt for yourself, or explicitly highlight them in your chat requests, helping the model adapt faster.
* **Advanced level (RAG/Agents):** These variables are perfect for connecting to a distributed or vector database. By accumulating data across these 4 dimensions, you can dynamically tune the model to a specific user's shifting priorities, competence, and experience.

### 2. Customizing Routing Rules
You can rewrite the Router block to react more precisely to your specific work. Clearly define your triggers: which task types must always be solved with uncompromising accuracy (Protocol A), which require idea generation (Protocol B), and which routine operations the model must execute directly without the framework (Bypass).

### 3. Expanding the Verification Layer
The protocol allows you to set your own verification priorities (Synthesis/Crystallization Phase). Add your own pass criteria: e.g., mandatory code vulnerability checks, corporate tone-of-voice alignment, or character limits. You can do this in plain human language, even in compact versions. It acts as "fine-tuning" the logic rather than cluttering the prompt.

### 4. Creating Custom Solution Formulas
This protocol does not claim absolute universality. Many specific tasks don't fit the base mechanics. Using the proposed syntax as a foundation, you can invent and implement other, more precise and elegant "reasoning equations" for your unique task types.

---

## ⚖️ License
**re!think it Protocol** is distributed under the [CC BY 4.0](LICENSE) license.  
*(c) 2026 Real_Egor.*

You are free to use this protocol, embed it into your agents, scripts, and products (including commercial ones), and modify the logic to suit your needs. **The only condition** is to retain a link to this repository and provide attribution.
