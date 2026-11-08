---
name: slm-eval
description: Recommends the 5 best small language models (SLMs) to run locally on a laptop or PC for a given use case. It asks the user, as multiple-choice questions, how much they need each key benchmark (MMLU, GSM8K, MATH, HumanEval, MBPP, IFEval, BFCL) and how much memory the model may use. It then researches current models on Hugging Face and other sources, drops the ones that won't fit, and writes a report giving each model's minimum system requirements and its actual benchmark scores. Use this skill whenever the user wants to pick, compare, shortlist or get a recommendation for a local, offline, on-device or self-hosted LLM/SLM, asks "which small model should I run on my laptop/PC for X", mentions Ollama / LM Studio / llama.cpp / GGUF model choice, or wants models filtered by RAM/VRAM and benchmark needs, even if they don't say "SLM".
---

# SLM Eval

Turn "I need a local model for X" into a sourced shortlist of the 5 best small models that fit the user's hardware and needs.

Run these steps in order: **use case → questions → research → filter & rank → report**.

## Step 1: Understand the use case

Read the user's prompt for what the model is for (e.g. "summarize support tickets", "coding helper", "Hebrew chatbot", "agent that calls my APIs"), plus any hard constraints they mention: language, license (commercial use?), context length, CPU-only, OS.

If the use case is missing, ask for it in one short question before going on. Don't ask about anything the questions in step 2 already cover.

## Step 2: Ask the benchmark and memory questions

Ask with the `AskUserQuestion` tool. It takes at most 4 questions per call, so use **two calls**:
- Call 1: MMLU, GSM8K, MATH, HumanEval
- Call 2: MBPP, IFEval, BFCL, Max memory

Each benchmark question uses three options: **Low**, **Medium**, **High**. Put the plain-language explanation below in the question text, so the user can choose without knowing the benchmark. Before asking, look at the use case and add "(Recommended)" to the level you'd suggest, putting that option first. For a coding helper, for example, recommend High for HumanEval.

| Header | Question text (plain-language) |
|---|---|
| MMLU | "MMLU = a big multiple-choice quiz across 57 school/university subjects. It measures how much the model *knows*. How important is general knowledge for you?" |
| GSM8K | "GSM8K = grade-school math word problems. It measures simple step-by-step reasoning. How important is that?" |
| MATH | "MATH = hard competition-level math (algebra, geometry, number theory). How important is advanced math?" |
| HumanEval | "HumanEval = the model writes short Python functions that are checked by tests. It measures basic coding. How important is coding?" |
| MBPP | "MBPP = like HumanEval, with more (easier) beginner Python tasks. How important is everyday simple coding?" |
| IFEval | "IFEval = instructions with checkable rules ('under 50 words', '3 bullets', 'no commas'). It measures whether the model does *exactly* what you asked. How important is that?" |
| BFCL | "BFCL = the model is given tools/functions and must call the right one with the right inputs. It measures agent / tool-use ability. How important is that?" |

Option descriptions:
- **Low**: "Not important. Don't filter or rank on this."
- **Medium**: "Should be decent."
- **High**: "Must be strong. This drives the ranking."

**Max memory** question: "How much memory can the model use? (GPU VRAM if you have a dedicated GPU, otherwise system RAM. If you only know your total RAM, pick about half to two-thirds of it, since the OS and apps need the rest.)" Options: `4 GB`, `8 GB`, `16 GB`, `32 GB`. The "Other" choice is added automatically for custom values.

If `AskUserQuestion` isn't available (for example on claude.ai), ask the same questions as one numbered multiple-choice list in chat and wait for the answers.

## Step 3: Research candidates

The model landscape changes month to month, so **research fresh. Don't rely on memory** for which models exist or what they score. Use WebSearch / WebFetch (or whatever web tools you have). If you can run subagents, research candidates in parallel, one per model family, and have each return the fields in step 5.

1. **Build a candidate pool of about 10–20 models.** Search for current small instruct models (roughly 0.5B–32B parameters, whatever the memory budget allows). Good starting families include Qwen, Gemma, Llama, Phi, Mistral/Ministral, Granite, SmolLM, OLMo, DeepSeek-R1 distills and EXAONE, but always check for newer releases ("best small LLM <current month/year>", r/LocalLLaMA, Hugging Face trending). Use **instruct/chat** versions, not base models.
2. **For each candidate, collect scores** from the sources below, roughly most to least authoritative:
   - **Official Hugging Face model card** (`huggingface.co/<org>/<model>`): the benchmark table, license, context length, languages and knowledge cutoff.
   - **Technical report / paper** (usually linked from the card, often on arXiv). It has fuller benchmark tables.
   - **BFCL leaderboard** (`gorilla.cs.berkeley.edu/leaderboard.html`) for tool calling. Note which BFCL version.
   - **Open LLM Leaderboard** (Hugging Face; archived/retired in 2025, but still useful for older models' MMLU-Pro, IFEval, MATH).
   - **Independent aggregators** such as Artificial Analysis, LMArena and Papers with Code, as a cross-check.
3. **Collect memory/size data:**
   - The **Q4_K_M GGUF file size**, from a GGUF repo on Hugging Face (official, `bartowski`, `unsloth`) or the Ollama library page (`ollama.com/library/<model>`).
   - The **Q8_0** size, if listed.
   - Native context length.

Record a source URL for every number. A score with no source is a guess, and the whole value of this report is that it can be checked.

## Step 4: Filter and rank

**Memory filter (hard rule):**
`Min memory = Q4_K_M file size + 1.5 GB` (runtime overhead plus a KV cache for about 8k context).
Drop any model whose min memory is above the user's max. If no GGUF size is published, estimate it as `params (B) × 0.6 GB` and mark it "est.".

**Benchmark thresholds.** These are rough bars for small models; scores vary with shot count and benchmark version, so treat them as guides:

| Benchmark | Medium ≥ | High ≥ |
|---|---|---|
| MMLU | 60 | 70 |
| GSM8K | 70 | 85 |
| MATH | 35 | 55 |
| HumanEval | 50 | 70 |
| MBPP | 50 | 65 |
| IFEval | 65 | 78 |
| BFCL (overall) | 45 | 55 |

**Ranking:**
- Weight each benchmark by the user's answer: **High = 3, Medium = 2, Low = 0**.
- For each model, normalize each weighted benchmark across the remaining pool (best in pool = 1.0) and sum the weighted values.
- A model that misses a **High** threshold drops below every model that meets them all.
- A missing score counts as 0 for that benchmark and is shown as "not reported". Never fill it in from memory or by guessing, because a made-up number misleads more than an honest gap.
- Break ties toward the smaller memory footprint and the more permissive license.
- Then apply the hard constraints from step 1 (license, language, context length). These exclude models outright.

Take the top 5. If fewer than 5 fit, report the ones that do and say what would unlock more (e.g. "with 16 GB, X and Y also fit").

## Step 5: Write the report

Save the report as `slm-report-<short-use-case>-<YYYY-MM-DD>.md` in the working directory. Then give the user the summary table and the file link in chat. Use this structure:

```markdown
# Local SLM Recommendation: <use case>
_Researched <date>. Scores are as published by each source; check the links._

## Your requirements
- Use case: ...
- Max memory for the model: X GB
- Benchmark priorities: MMLU High · GSM8K Low · ... (one line)
- Other constraints: license / language / context ...

## Top 5 at a glance
| # | Model | Params | Min memory (Q4) | MMLU | GSM8K | MATH | HumanEval | MBPP | IFEval | BFCL | License |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | [Name](HF link) | 7B | 6.2 GB | 74.2 | 91.6 | ... | ... | n/r | ... | ... | Apache-2.0 |

(Bold the benchmarks marked High. Use "n/r" for not reported.)

## 1. <Model name>
**Why it fits:** 1–2 sentences tied to the user's priorities.

**Minimum system requirements**
| | Value |
|---|---|
| Min memory (Q4_K_M) | X GB RAM/VRAM |
| Comfortable (Q8_0) | Y GB |
| Download size (Q4_K_M) | Z GB |
| Context length | N tokens |
| Runs on CPU only? | Yes (slow, ~a–b tok/s) / GPU recommended |
| How to run | `ollama run <tag>` / LM Studio search "<name>" |

**Benchmark scores**
| Benchmark | Score | Your need | Meets? | Source |
|---|---|---|---|---|
| MMLU (5-shot) | 74.2 | High | ✅ | [model card](url) |
| ... |

**Watch out for:** limitations from the card (languages, license limits, known weaknesses).

(repeat for models 2–5)

## Considered but excluded
- Model A: needs 9.5 GB (over your 8 GB limit)
- Model B: IFEval 58, below your High bar
(short list, one line each)

## Caveats
- Vendors choose which benchmarks they publish, and shot counts or versions differ between sources, so compare with care.
- Benchmarks are a filter, not the final answer: try the top 2–3 on 20–50 of your real prompts (e.g. with promptfoo + Ollama) before committing.
```

Rough CPU speed guide for the "Runs on CPU" row: tokens/s ≈ memory bandwidth ÷ model file size (DDR4 laptop ~40 GB/s, DDR5 ~70 GB/s, RTX 4060 ~270 GB/s, Apple M-series 100–400 GB/s).

End by offering to generate a promptfoo config that tests the top 3 on the user's own prompts.
