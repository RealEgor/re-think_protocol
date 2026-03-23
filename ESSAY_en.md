# re!think it: A ~1,300-Token Prompt That Teaches LLMs to Reason — Grounded in Math, Not Words

## Intro: The Over-Engineering Problem

There is a huge contradiction in how the IT industry uses LLMs today. 

Models have million-token context windows. They understand code, math, and logic perfectly. But what do we do? We treat them like dumb text generators. We wrap them in hundreds of thousands of lines of external Python code — LangChain, multi-agent frameworks, RAG checkers, and semantic routers. We build a massive factory just to hammer a nail.

I wanted to test a different approach: **Can we get the same stable results by building the whole logic *inside* the context window?**

It started as an experiment in plain [Russian](prompts/re-think_v1_ru.md) (full and [compact](prompts/re-think_v1_ru_compact.md) versions), then [English](prompts/re-think_v1_en.md) (full and [compact](prompts/re-think_v1_en_compact.md) versions — the English compact came in at around 1,300 tokens).

When I looked under the hood of my own prompt, I realized I had intuitively packed 7 heavy backend mechanics into a single `.md` file. 

Below is what I found inside.

---

## Core Mechanics: Enterprise Scaffolding vs. In-Context Logic

For consistency, every concept below is broken into the same three angles:

- **What it is** — a short description so you know what problem we're talking about
- **How the industry does it** — this part was assembled with LLMs as research partners; they analyzed existing approaches and gave their take, I only adjusted for clarity of presentation
- **How re!think does it** — how this mechanic is implemented inside the protocol, as concisely as possible

And if you want to see how all of this is explained *inside the protocol itself* — with the full reasoning and construction logic baked in — there's a link at the end of the post.

---

### 1. Intent Routing (⬡ ROUTER)
**What it is:** Before doing anything, the system needs to understand what the user wants (e.g., exact search vs. creative brainstorming) and choose the right path.

**How the industry does it:** From what I've read, the standard setup is something like: run the prompt through an embedding model, compare vectors, route to the right LLM chain via Python. I haven't built this stack myself — but it's well documented in LangChain's own guides, so I'll take it at face value.
*(Verdict: slow, heavy, a separate network call before the actual work even starts).*

**How re!think it does it:** A strict `PROT_A / PROT_B / C_BYPASS` IF/THEN block right in the system prompt. The model silently categorizes the request and switches branches instantly.
*   **Main drawback:** It can make a mistake on very confusing prompts. The industrial way is more accurate.
*   **Absolute win:** Zero latency. Zero external code. It has an explicit bypass (`C_BYPASS`) so simple questions don't get processed by heavy reasoning frameworks.

*A side note on routing in general: delegating this step to a lightweight model is like putting a junior manager in charge of a team of specialists — someone who decides who works on what, but doesn't fully understand what anyone actually does. Routing is one of the most consequential steps in the pipeline. Get it wrong, and more than half your trajectories become meaningless from the very start. Real flexibility of thinking begins with a thoughtful choice of tool — and an equally thoughtful decision about how to apply it.*

### 2. Pre-generation Gating (HARD STOP)
**What it is:** If the system is missing critical data, it must stop and ask. It shouldn't guess (hallucinate) just to give a fast answer.

**How the industry does it:** As best I understand it — write validation code, have the LLM output JSON, run an external script (Pydantic seems to be the standard) to check for missing fields. Or do a second LLM pass to review the first one.
*(How it works in practice: the LLM generates garbage, then you try to catch it downstream. At least that's what it looks like from the outside.)*

**How re!think it does it:** The model solves a simple math formula before answering: `Δ = Goal − (Context + Tools)`. If `Δ` is too big — meaning there's a critical gap in what I know about your situation — the model hits a **HARD STOP**. It must ask exactly 1 clear question. Not two. Not a list. One question that closes the most important gap. No second runs, no JSON parsing, no external validator.
*(Main drawback: it relies on the model being honest about what it doesn't know. The code-based check is more reliable on paper. But it's also slower, more brittle, and costs extra calls. The in-context version is much faster — and in practice, it works.)*

### 3. Profiling on the Fly
**What it is:** The system adapts to the user's skill level and limits without asking every time.

**How the industry does it:** RAG, as far as I know. Some service reads the chat, builds a profile, saves it to a Vector DB, pulls it out later. The details vary by stack — I'm simplifying.

**How re!think it does it:** The model extracts variables from the chat on the fly: `S_R` (Role), `S_T` (Trust level), `S_V` (Boundaries). These aren't just for "fixing the tone". They act as strict mathematical limits for the reasoning process.
*(Verdict: Great for one long chat session. But it won't remember you tomorrow. For long-term memory across days, Vector DB wins).*

*The main idea for me here wasn't "profile everything." It was: choose your profile parameters deliberately, and think in advance about exactly how each one will shape the reasoning — not just the tone of the response. `S_R` doesn't just adjust the vocabulary. It narrows the entire solution space the model is allowed to explore. That's what I want you to take from this section.*

### 4. Fighting "Average" Answers (Anti-Centroid Filter)
**What it is:** How to stop the model from giving boring, statistically average, "safe" answers.

**How the industry does it:** Temperature tweaking at the server level, mostly. There are also penalty algorithms that explicitly suppress high-frequency tokens — but I'm honestly not sure how widely those are deployed in production versus just described in papers.

**How re!think it does it:** A set theory rule in the prompt: `M_filtered = M - {P_centroid}`. I explicitly tell the model: "Throw away the first default answer that comes to your mind." Plus, it is forced to generate at least 3 standard paths and 1 completely counter-intuitive path.

*This is closer to controlled imagination than random creativity. If you just crank up the Temperature, you get wild and unfeasible ideas — noise dressed up as novelty. But if you explicitly remove the default answer while still searching within the space of plausible ideas, you get something more useful: non-obvious hypotheses that are actually worth testing — or at least worth running by the user.*

### 5. In-context Garbage Collector (Pointer GC)
**What it is:** Stopping attention drift. In a long chat, the model simply forgets the early rules and variables.

**How the industry does it:** Memory Agents, from what I've seen. They read the history continuously and produce summaries. Or companies just buy 2M-token context windows and throw everything in. Both are expensive. Whether either actually solves the drift problem or just postpones it — I honestly don't know.

**How re!think it does it:** A strict pointer system (`C.0014 := C.0005`). The rule is simple: a variable cannot live longer than 10 messages. On the 11th step, the model MUST rewrite the variable into the new message block. It's manual trash collection. We force the model to refresh the pointer before it falls out of the active attention zone.
*(Bonus: Absolute transparency. Because the model prints these variables in a technical header every time, you can audit exactly what data it is using to think).*

### 6. The "Anchor" (Attention Anchoring)
**What it is:** How to keep the model focused when the dialogue stretches. By the 100th message, the influence of the original system prompt drops to near zero, and the model starts ignoring instructions.

**How the industry does it:** Re-inject the system prompt periodically. Or just trust the native attention mechanism to hold things together. That second option seems more common than I would expect.
*(Verdict: expensive. And from everything I've read — fundamentally unreliable at scale).*

**How re!think it does it:** Every single technical log strictly begins with the trigger phrase `re!think protocol`. I did this originally just to separate logs visually — it had nothing to do with attention mechanics. But it turns out it acts as a cognitive anchor. The model physically has to print these words before doing anything. And printing them drags its attention back to the core rules immediately before it generates the response. It's not a hack. It's how attention actually works — you focus on what you just touched.
*(Cost: 3 tokens. Effect: a non-stop "System 2" wake-up call. Instead of hoping the model stays obedient, we force it to touch the rule-set before every single action.)*

*Full disclosure: this isn't a complete solution. It extends the working life of the system prompt — but it can still break down in very long, digressive conversations. A truly robust fix would require training the model on thousands of protocol-driven dialogues, so that the reasoning logic gets baked into the base weights. Once that happens, anchoring at each step becomes a reflex — the same way you don't think about balance when you walk. Until then, this anchor is a useful patch, not a cure. The real goal is a model that doesn't need to "remember the formula" — because it already knows how to think.*

### 7. Structured Math vs. "Stream of Consciousness"
**What it is:** How to control the actual logic flow, ensuring the model solves the task instead of just talking to itself in endless circles.

**How the industry does it:** Chain-of-Thought. Tell the model to "think step-by-step." It generates a wall of text.
*(How it works in practice: like pouring water on the floor and hoping the puddle magically shapes into a perfect polygon. Completely uncontrolled.)*

**How re!think it does it:** The protocol shatters this "blurry" thinking. It forces the model to treat logic as a chain of discrete, solvable equations: take output A, use it as Tool B, solve Goal C. We don't allow the model to philosophize. We teach it to think algebraically, where every answer becomes the direct input for the next step.
*(Verdict: We kill the generative word soup and turn the reasoning process into a deterministic calculating machine).*

---

## Protocol in Action: Real-World Case Studies

The examples below are documented in a separate file to keep this essay focused on the mechanics. Each case study runs the same request through the same model — with and without the protocol — and breaks down exactly what changed and why.

**Four documented cases:**

| # | Topic | Mechanism demonstrated |
|---|---|---|
| 1 | SaaS Pricing Model | **SOFT STOP** — model catches an empty seed before expanding |
| 2 | Engineer Departure | **Assumptions** — explicit fork declaration instead of hedging both paths |
| 3 | B2B Growth Strategy | **HARD STOP** — mathematical impossibility detected and blocked |
| 4 | Document Processing Pipeline | **HARD STOP** — architecture generation blocked pending design constraints |

In every case, the pattern is the same: a standard model absorbs the ambiguity, generates a plausible-looking answer, and leaves the hard thinking to you. The protocol-equipped model resolves the ambiguity first — by stopping, naming the gap, and requiring the missing variable before generating anything.

📂 **[View all four case studies → examples_en.md](examples_en.md)**  
📂 **[Русские примеры → examples_ru.md](examples_ru.md)** *(HR-кейс с HARD STOP)*



---

## The Boundary Between Orchestration and Reasoning

Let's be clear: I am not saying the industry is completely wrong and my text file solves everything. I haven't run formal benchmarks on this yet — that's planned for v1.1.

But the problem is simpler than it looks. We are confusing two completely different things: **external orchestration** (running code around the model) and **internal reasoning** (what actually happens inside the context window).

Advanced reasoning cannot be strictly "programmed" in the traditional sense. You can tune it. You can constrain it. You can shape it. You can even write its instructions using Python syntax. But in advanced reasoning, **there is no place for a compiler**.

When we try to force a multi-step cognitive process through external Python scripts — stopping the model, parsing its JSON, running a rigid logic check, and feeding it back — we are treating a fluid cognitive engine like a deterministic calculator.

We gave these models massive context windows. At this scale, we need a paradigm shift in how we communicate with them. It is no longer about "prompt engineering." It is about **Neuro-Linguistic Programming (NLP)** — in its original psychological sense.

*(Yes, I know — NLP has a bad reputation in science. But the core idea I took from it is simple: if you control how a thinking system talks to itself, you control how it thinks. That's the whole premise here. Not therapy. Not pseudoscience rituals. Just the basic mechanic — language structure shapes reasoning, and you can engineer that structure deliberately. That idea is what started this whole experiment.)*

Think about humans. Our working memory is famously limited. When we solve complex problems, we don't compile our thoughts; we write things down to not lose the thread. An LLM has a 1-million-token memory, but it STILL needs to write its working state into the chat or it loses focus. My protocol simply forces it to do so systematically.

We don't need to wrap the model in external code to make it think. We need to write better, stricter cognitive contracts with the model itself.

### One Last Thing About Pseudo-Code
The main defense against hallucination in my protocol is the strict structure and logging. But when I built a full [Chinese version](prompts/re-think_v1_zh.md) and its [compact equivalent](prompts/re-think_v1_zh_compact.md), I hit a wall. Even the compact Chinese version clocked in at around 2,100 tokens.

So I asked myself: **What is the cheapest language for an LLM? What happens if you strip out every last word?**
The answer was [pseudo-code](prompts/re-think_v1_pseudo_code.md). Token-wise, it lands at roughly the same ~1,300 as the English compact — but it turned out to be significantly more resistant to attention drift.

And here I found an unexpected bonus: pseudo-code didn't just save space. It killed **attention drift**. Natural language is fluid — models forget or misinterpret words over long chats. But strict math operators and variables? There is zero room for interpretation. The fact that the pseudo-code version became almost immune to hallucination was just a fantastic, unexpected bonus.

### A Note on Optimization — and Where I'm Going Next

After publishing the pseudo-code version, I did a careful token-by-token analysis of it looking for places to cut. The candidates were real: LaTeX notation costs more tokens than Unicode equivalents, `// Reason:` comments add weight, bold markers multiply tokens.

But when I looked at *what each of those "extra" tokens actually does*, I realized there's no real waste here. Every heavy construct serves one of two functions: it either **directs the model's attention** — telling it where to switch modes, where to stop, where exactness is required — or it **explains the mechanics and their necessity**, so the model understands not just *what* to do but *why* the rule exists and what breaks if it doesn't. The LaTeX math context signals "formal logic, not prose." The `// Reason:` comments explain why a rule is non-negotiable. The `**HARD STOP**` bold marks a hard mode switch, not a suggestion.

You can strip all of that once a model is fine-tuned on thousands of protocol-driven dialogues. At that point, the explicit scaffolding becomes redundant — the attention management and the explanation of mechanics are already baked into the weights. The protocol stops being instructions and becomes reflex.

But until that happens, every "extra" token is doing real work.

So: anyone who wants to experiment with a leaner version — go ahead. The candidates are documented. I'm not stopping you. But my own roadmap is different. I'm not going to spend time shaving tokens off v1.

I'm working on the next layer: a **universal standard of reasoning**, where every class of cognitive task has its own vector transformation formula. Not just PROT_A and PROT_B. A full taxonomy — from causal analysis to constraint satisfaction to adversarial stress-testing — each with a precise mathematical description of how input context maps to a verified output. A thinking OS, not just a prompt.

That's the actual problem worth solving.

---
📖 **View the full documentation and usage instructions in the main README** *(available in 3 languages)*:
[English](README.md) | [Русская версия](README_ru.md) | [中文说明](README_zh.md)
