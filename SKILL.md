---
name: slm-eval
description: Finds, compares and ranks the 5 best small language models (SLMs) to run locally on a laptop or PC. It asks multiple-choice questions about the use case, benchmark needs (MMLU, GSM8K, MATH, HumanEval, MBPP, IFEval, BFCL), speed, commercial use and max memory. It then researches current SLMs on Hugging Face and leaderboards and writes a sourced report with each SLM's minimum system requirements and real benchmark scores. Use it whenever the user types "slm-eval" or asks to find, pick, compare, evaluate or shortlist SLMs, such as "find me a SLM for X", "which SLM should I run locally?", "recommend an SLM for my laptop", "best SLM for 8GB RAM", "compare SLMs for coding", "what SLM's can my PC run?", "SLM for my game/app/agent", "evaluate SLMs for my needs", or "help me choose an SLM", even when phrased casually.
---

# SLM Eval

Turn "I need a local SLM for X" into a sourced shortlist of the 5 small models that best fit the user's hardware and needs.

Run these steps in order: **questions → research → filter & rank → report**.

## Step 1: Ask the questions

Ask with the `AskUserQuestion` tool. It takes at most 4 questions per call, so use **three calls**. Ask the context questions first, because their answers decide which benchmark levels to mark "(Recommended)" in calls 2 and 3.

### Call 1: Context

1. **Use case.** Ask this only if the prompt doesn't already say what the model is for.
   - Header: `Use case`. Question: "What do you want the local SLM to do? Pick one, or choose Other and describe it, including must-haves like language."
   - Options: `Coding helper` · `Chat / Q&A assistant` · `Summarize documents` · `Tool-using agent`
2. **Speed.**
   - Header: `Speed`. Question: "Does the SLM need to reply fast, e.g. real-time in an app or game loop?"
   - `Yes, fast replies` (description: "Prefer models that answer without a long thinking step")
   - `No, quality first` (description: "Thinking/reasoning models are fine")
3. **Commercial use.**
   - Header: `Commercial`. Question: "Will you use it commercially (sell or ship a product with it)?"
   - Options: `Yes` · `No` · `Not sure`
   - The answer decides which licenses are allowed. Many strong small models are non-commercial or have revenue caps.
4. **Max memory.**
   - Header: `Max memory`. Question: "How much memory can the SLM use? GPU VRAM if you have a dedicated GPU, otherwise system RAM. If you only know your total RAM, pick about half to two-thirds of it, since the OS and other apps need the rest."
   - Options: `4 GB` · `8 GB` · `16 GB` · `32 GB`. "Other" is added automatically for custom values.

### Calls 2 and 3: Benchmarks

- Call 2: MMLU, GSM8K, MATH, HumanEval
- Call 3: MBPP, IFEval, BFCL

Each question has three options: **Low**, **Medium**, **High**. Put the plain-language explanation below in the question text, so the user can choose without knowing the benchmark. Mark the level that fits the use case "(Recommended)" and put it first. For example, recommend High for HumanEval for a coding helper, and High for BFCL for a tool-using agent.

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

If `AskUserQuestion` isn't available (e.g. on claude.ai), ask the same questions as one numbered multiple-choice list in chat and wait for the answers.

## Step 2: Research candidates

The SLM landscape changes month to month, so **research fresh. Don't rely on memory** for which models exist or what they score. Use WebSearch / WebFetch, or whatever web tools you have.

**Reusing research from this session.** Full research costs many tool calls, so you can reuse what's already in this conversation, but only when all of these hold:
- it was done within the last ~24 hours
- it covered a memory budget at least as large as the current one
- it collected the fields this run needs, including thinking-mode info if the user wants fast replies

When you reuse:
- Fill only the gaps the new answers create, e.g. non-thinking scores, a new license filter, or a larger budget.
- Re-check on the model card any top-5 entry whose data is vendor-only, uncertain, or was never checked directly.
- Say in the report's Caveats that research was reused and what was re-checked.

Memory from earlier conversations doesn't count. Research fresh in that case.

**If you can run subagents, run them in parallel:**
- one per model family group, e.g. Qwen + DeepSeek; Gemma + Phi + Granite; Llama + Mistral + gpt-oss + OLMo + SmolLM
- one for releases from the last ~6 months plus the official BFCL leaderboard data

Give each agent the user's memory budget, use case, and the fields listed below. Tell each one to cite a URL for every number, and to write "n/r" instead of estimating.

1. **Build a candidate pool of about 15–25 models.**
   - Search for current small instruct models (roughly 0.5B–32B parameters, whatever the memory budget allows).
   - Starting families include Qwen, Gemma, Llama, Phi, Mistral/Ministral, Granite, SmolLM, OLMo, gpt-oss, Liquid LFM, MiniCPM, NVIDIA Nemotron and EXAONE.
   - Always check for newer releases: "best small LLM <current month/year>", r/LocalLLaMA, Hugging Face trending, and each org's HF model list sorted by date.
   - Use **instruct/chat** versions, not base models.
2. **For each candidate, collect scores** from these sources, roughly most to least authoritative:
   - **Official Hugging Face model card** (`huggingface.co/<org>/<model>`): the benchmark table, license, context length, languages, and whether it thinks by default.
   - **Technical report / paper**, usually on arXiv. It has fuller tables and often both thinking and non-thinking scores.
   - **Official BFCL leaderboard.** The page is JavaScript-rendered; read the data file `https://gorilla.cs.berkeley.edu/data_overall.csv` instead.
   - **Independent evaluations:** other labs' model cards that compare competitors, papers, Artificial Analysis, and the archived Open LLM Leaderboard.
3. **Collect memory and runtime data:**
   - The Q4_K_M GGUF file size, from an official GGUF repo, `bartowski`, `unsloth` or `ollama.com/library/<model>`.
   - The Q8_0 size.
   - Total and active parameters (for mixture-of-experts models).
   - Native context length.
   - Thinking mode: none / on by default (can be turned off) / always on.
   - Ollama tag.

**Rules for recording scores.** They exist so the report can be checked and compared fairly:
- **Every number needs a source URL.** A score without one is a guess.
- **Label every score with its variant.** That means shot count, strict/loose, thinking/non-thinking, and benchmark version.
- **Base-model scores** (a "-Base" model, or a "base" table in a report) are shown in the report labelled "(base)" but **never counted** in the ranking. The instruct model the user will run can score quite differently.
- **Replacement benchmarks.** Many 2025–2026 model cards dropped the classic benchmarks. When the classic one isn't reported, use a replacement and label it, e.g. "MMLU-Redux 91.1". The thresholds in step 3 list which replacements are accepted.
- **When speed matters** (the user answered "Yes, fast replies"): use the **non-thinking** score when the card publishes both. If only thinking-mode scores exist, use them and mark them "(thinking)".
- **BFCL comes in kinds that can't be compared:**
  - **v3 / v4 reported runs:** BFCL v3 or v4 overall scores reported on a model card or in a paper. They are either the model maker's own run (**vendor**) or another lab's (**independent**). Both use the same scale, where small models score about 50–70.
  - **official V4**: the Berkeley leaderboard, where the best small models score only about 40–55%

  Label every BFCL number with its kind, e.g. "v3 vendor", "v4 independent" or "official V4".

## Step 3: Filter and rank

**Hard filters.** Drop a model if any of these is true:
- **Memory:** `Q4_K_M file size + 1.5 GB` (runtime overhead plus about 8k context of KV cache) is more than the user's max. If no GGUF size is published, estimate it as `params (B) × 0.6 GB` and mark it "est.".
- **License:** the user answered **Yes** to commercial use and the license is non-commercial (CC-BY-NC, "research only", EXAONE NC and similar). For "Not sure", keep the model but flag the license.
- **Constraints** the user stated (language, context length, OS).

Licenses with conditions (revenue caps, required attribution, user limits) are kept but flagged ⚠️, with the condition spelled out.

**Benchmark thresholds.** These are rough bars for small models:

| Benchmark | Medium ≥ | High ≥ | Accepted replacements (label them) |
|---|---|---|---|
| MMLU | 60 | 70 | MMLU-Redux (same bars); MMLU-Pro (harder: Medium ≥ 50, High ≥ 60) |
| GSM8K | 70 | 85 | GSM-Plus, MGSM (same bars) |
| MATH | 35 | 55 | MATH-500, Minerva Math (same bars) |
| HumanEval | 50 | 70 | HumanEval+, EvalPlus (same bars) |
| MBPP | 50 | 65 | MBPP+ (same bars) |
| IFEval | 65 | 78 | any IFEval variant (strict/loose, prompt/instruction level); note which |
| BFCL | 45 | 55 | **v3 or v4 overall reported runs only, vendor or independent.** Prefer vendor if both exist and say so if they differ widely. Official V4 leaderboard numbers are a cross-check, never compared against these bars. |

If a model has only an official-V4 BFCL score, judge it against the other models' official-V4 scores and say so in the report.

**Ranking:**
- Weight each benchmark by the user's answer: **High = 3, Medium = 2, Low = 0**.
- For each model, normalize each weighted benchmark against the best in the remaining pool (best = 1.0), then sum.
- A missing score counts as 0 for that benchmark and shows as "n/r". Never fill a gap from memory or guesswork; a made-up number misleads more than an honest gap.
- A model that misses any **High** bar, including through a missing score, ranks below every model that clears them all.
- If the user wants fast replies, a model that is **always** thinking ranks below models that can answer without thinking.
- Break ties toward the smaller memory footprint, then the more permissive license.

**Sanity checks before you finalize:**
- **Vendor-only outliers.** If a much smaller model beats much larger ones only on its own vendor's numbers, keep it but flag it as "vendor-only claim, verify yourself". Where you can, check the numbers on the card yourself, and confirm which column of a comparison table is the model.
- **Contradicted claims.** If independent results contradict a vendor claim by a wide margin (e.g. vendor BFCL 76, independent 29), trust the independent number and say why.
- **Name the cause of a big lead.** If the top models lead mainly because others didn't report a benchmark, say that in the report.

Take the top 5. List the next 2–3 models that cleared every High bar as runners-up. If fewer than 5 fit, report the ones that do and say what would unlock more (e.g. "with 16 GB, X and Y also fit").

## Step 4: Write the report

Save it as `slm-report-<short-use-case>-<YYYY-MM-DD>.md` in the working directory. Then give the user the at-a-glance table and the file link in chat. Use this structure:

```markdown
# Local SLM Recommendation: <use case>
_Researched <date>. Scores are as published by each source; check the links._

## Your requirements
- Use case: ...
- Max memory for the model: X GB
- Fast replies needed: yes/no · Commercial use: yes/no/not sure
- Benchmark priorities: **IFEval High** · GSM8K Medium · MMLU Low · ... (one line)
- Other constraints: language / context ...

## How they were picked
2–4 lines: pool size and families, the memory rule, the bars used, and the weights.

## Top 5 at a glance
| # | Model | Params | Min memory (Q4) | MMLU | GSM8K | MATH | HumanEval | MBPP | IFEval | BFCL | Thinking | License | Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | [Name](HF link) | 8B | 6.9 GB | 73.8 | 92.5 | n/r | 85.4 | 87.3 | **87.1** | **68.3** (v3 vendor) | none | Apache-2.0 | 7.55 |

(Bold the benchmarks marked High. Use "n/r" for not reported. Thinking is none / switchable / on by default / always on. Mark thinking-mode scores with "ᵗ". Put replacements and caveats in numbered footnotes under the table.)

## 1. <Model name>
**Why it fits:** 1–2 sentences tied to the user's priorities.

**Minimum system requirements**
| | Value |
|---|---|
| Min memory (Q4_K_M) | X GB RAM/VRAM |
| Comfortable (Q8_0) | Y GB |
| Download size (Q4_K_M) | Z GB (link) |
| Context length | N tokens |
| Thinking mode | none / on by default (can be turned off) / always on |
| Runs on CPU only? | Yes (~a–b tok/s) / GPU recommended |
| How to run | `ollama run <tag>` / LM Studio search "<name>" |

**Benchmark scores**
| Benchmark | Score | Your need | Meets? | Source |
|---|---|---|---|---|
| IFEval (strict prompt) | 87.1 | High | ✅ | [model card](url) |
| ... |

**Watch out for:** license conditions, thinking-mode latency, chat or tool-call format quirks, vendor-only claims, and weaknesses listed on the card.

(repeat for models 2–5)

## Runners-up
One line each.

## Considered but excluded
One line each, with the reason (over memory, below a High bar, missing score, license, or a contradicted claim).

## Caveats
- The kinds of BFCL scores used, and why they aren't comparable.
- Which scores are thinking-mode only.
- Benchmarks are a filter, not the final answer: try the top 2–3 on 20–50 of your real prompts (e.g. with promptfoo + Ollama) and measure response time on your own machine.
```

**Speed guide for the "Runs on CPU only?" row.** Tokens per second is roughly memory bandwidth divided by the bytes read per token:
- **Dense models:** bytes per token ≈ the Q4 file size.
- **Mixture-of-experts models:** bytes per token ≈ active params (B) × 0.6 GB. They run much faster than their file size suggests.
- **Typical bandwidths:** DDR4 laptop ~40 GB/s, DDR5 ~70 GB/s, RTX 4060 ~270 GB/s, Apple M-series 100–400 GB/s.

End by offering to generate a promptfoo config that tests the top 3 on the user's own prompts.
