# Local SLM Recommendation: AI for a PC RTS game
_Researched 2026-11-08. Scores are as published by each source; check the links._

## Your requirements
- **Use case:** AI inside a PC RTS game. Assumed to be an AI commander that issues game commands through function/tool calls and replies in a strict format your game code can read.
- **Max memory for the model:** 16 GB
- **Benchmark priorities:** **IFEval High** · **BFCL High** · GSM8K Medium · MMLU Low · MATH Low · HumanEval Low · MBPP Low
- **Other constraints:** none stated. Licenses are flagged in case you ship the game commercially. Response speed matters for a real-time game, so each model's thinking mode is noted.

## How they were picked
- **Pool:** 25+ small instruct models from Qwen, DeepSeek, Gemma, Phi, Granite, Llama, Mistral, OpenAI gpt-oss, OLMo, SmolLM, Liquid, MiniCPM, NVIDIA and EXAONE, plus the official BFCL V4 leaderboard.
- **Memory filter:** min memory = Q4_K_M file + 1.5 GB, and it must be ≤ 16 GB.
- **Bars:** IFEval ≥ 78 and BFCL ≥ 55 (both High), GSM8K ≥ 70 (Medium).
- **Score:** weighted 3 / 3 / 2 and normalized against the best in the pool.
- **Missing scores** count as 0. **Base-model scores** are shown but not counted.

## Top 5 at a glance
| # | Model | Params | Min memory (Q4) | MMLU | GSM8K | MATH | HumanEval | MBPP | **IFEval** | **BFCL** | License | Score |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | [Granite 4.1 8B](https://huggingface.co/ibm-granite/granite-4.1-8b) | 8B | 6.9 GB | 73.8 | 92.5 | n/r¹ | 85.4 | 87.3 | **87.1** | **68.3** (v3) | Apache-2.0 | 7.55 |
| 2 | [gpt-oss-20b](https://huggingface.co/openai/gpt-oss-20b) | 21B MoE (3.6B active) | 13.1 GB | 85.3 | 93.3² | n/r | 94.5² | n/r | **86.8**² | **60.9**² (v3) | Apache-2.0 | 7.25 |
| 3 | [LFM3-1.7B](https://huggingface.co/LiquidAI/LFM3-1.7B) | 1.7B | 2.6 GB | n/r | n/r | n/r | n/r | n/r | **96.7** | **71.4** (v4) | LFM Open v1.0 ⚠️ | 6.00 |
| 4 | [Qwen3.5-9B](https://huggingface.co/Qwen/Qwen3.5-9B) | 9B | 7.2 GB | n/r³ | n/r | n/r | n/r | n/r | **91.5** | **66.1** (v4) | Apache-2.0 | 5.62 |
| 5 | [Qwen3-14B](https://huggingface.co/Qwen/Qwen3-14B) | 14.8B | 10.5 GB | n/r⁴ | n/r⁴ | 96.8 (MATH-500) | n/r⁴ | n/r⁴ | **85.4** | **70.4** (v3) | Apache-2.0 | 5.61 |

¹ Granite reports Minerva Math 80.1 instead of MATH.
² Independent evaluation (EngGPT2 paper), not OpenAI's own numbers.
³ Qwen3.5-9B reports MMLU-Redux 91.1 and MMLU-Pro 82.5 instead of MMLU.
⁴ Qwen3-14B instruct isn't reported; the base-model scores are in its section.

**Why #1 and #2 lead:** they are the only models that publish a GSM8K score, which is your Medium need. #3–#5 are within a few points of them on the two High benchmarks.

---

## 1. Granite 4.1 8B (IBM)
**Why it fits:** It clears every bar and has the most complete published scores in the pool. It's a non-reasoning model, so it answers immediately with no thinking pause, which suits a real-time commander. At 8B it's light enough to run alongside the game.

**Minimum system requirements**
| | Value |
|---|---|
| Min memory (Q4_K_M) | 6.9 GB RAM/VRAM |
| Comfortable (Q8_0) | 10.9 GB |
| Download size (Q4_K_M) | 5.35 GB ([official GGUF](https://huggingface.co/ibm-granite/granite-4.1-8b-GGUF)) |
| Context length | 128K tokens |
| Runs on CPU only? | Yes, about 8–13 tokens/s on a DDR4/DDR5 laptop. A GPU is better for in-game use (RTX 4060: about 50 tokens/s). |
| How to run | `ollama run granite4.1:8b` / LM Studio search "granite-4.1-8b" |

**Benchmark scores** ([model card](https://huggingface.co/ibm-granite/granite-4.1-8b))
| Benchmark | Score | Your need | Meets? | Source |
|---|---|---|---|---|
| IFEval (avg) | 87.06 | High | ✅ | model card |
| BFCL v3 | 68.27 | High | ✅ | model card (IBM's own run) |
| GSM8K (8-shot) | 92.49 | Medium | ✅ | model card |
| MMLU (5-shot) | 73.84 | Low | – | model card |
| Minerva Math | 80.10 | Low | – | model card |
| HumanEval | 85.37 | Low | – | model card (the IBM blog says 87.20) |
| MBPP | 87.30 | Low | – | model card (the IBM blog says 82.54) |

**Watch out for:**
- The BFCL score comes from IBM's own run. Granite 4.1 isn't on the official BFCL leaderboard yet.
- The newer [Granite 4.2 8B](https://huggingface.co/ibm-granite/granite-4.2-8b) is a thinking model. Its BFCL v4 is 52.39 and it reports no IFEval, so stay on 4.1 for speed.
- Tool calls use OpenAI-style function definitions.

## 2. gpt-oss-20b (OpenAI)
**Why it fits:** It has the highest GSM8K in the pool and strong general knowledge (MMLU 85.3). Its IFEval and BFCL scores clear your bars in an independent evaluation, not just the vendor's. It's a mixture-of-experts model: only 3.6B of its 21B parameters are active per token, so it runs faster than its size suggests.

**Minimum system requirements**
| | Value |
|---|---|
| Min memory (Q4_K_M) | 13.1 GB (13.6 GB with the native MXFP4 file) |
| Comfortable | about 14 GB. The model ships natively in MXFP4, so Q8_0 is the same 12.11 GB file. |
| Download size | 11.62 GB Q4_K_M ([unsloth](https://huggingface.co/unsloth/gpt-oss-20b-GGUF)) / 12.11 GB MXFP4 ([ggml-org](https://huggingface.co/ggml-org/gpt-oss-20b-GGUF)) |
| Context length | 131K tokens |
| Runs on CPU only? | Workable thanks to MoE: about 15–30 tokens/s on DDR5. A GPU with 16 GB VRAM is recommended. |
| How to run | `ollama run gpt-oss:20b` / LM Studio search "gpt-oss-20b" |

**Benchmark scores**
| Benchmark | Score | Your need | Meets? | Source |
|---|---|---|---|---|
| IFEval (loose prompt) | 86.8 | High | ✅ | [EngGPT2 paper, Table 8](https://arxiv.org/abs/2603.16430) (third party) |
| BFCL v3 | 60.9 | High | ✅ | EngGPT2 paper, Table 8 |
| GSM8K | 93.3 | Medium | ✅ | EngGPT2 paper, Table 8 |
| MMLU | 80.4 / 84.0 / 85.3 (low / med / high effort) | Low | – | [OpenAI model card](https://arxiv.org/abs/2508.10925) |
| HumanEval | 94.5 | Low | – | EngGPT2 paper |
| MATH, MBPP | n/r | Low | – | – |

**Watch out for:**
- It's a reasoning model. Set reasoning effort to **low** for game ticks, because high effort can produce thousands of thinking tokens.
- It needs OpenAI's "harmony" chat format. The card says it "will not work correctly otherwise".
- At 13+ GB it uses most of your 16 GB, leaving little room for the game itself on the same GPU.
- It isn't on the official BFCL V4 leaderboard.

## 3. LFM3-1.7B (Liquid AI)
**Why it fits:** It has the best IFEval (96.66) and BFCL v4 (71.39) in the pool, at only 1.7B parameters and 2.6 GB. That leaves almost all of your memory for the game, and on a GPU it's very fast. It was trained heavily on agentic and tool-use data.

**Minimum system requirements**
| | Value |
|---|---|
| Min memory (Q4_K_M) | 2.6 GB |
| Comfortable (Q8_0) | n/r (likely about 3.5 GB) |
| Download size (Q4_K_M) | 1.1 GB ([official GGUF](https://huggingface.co/LiquidAI/LFM3-1.7B-GGUF)) |
| Context length | up to 256K tokens |
| Runs on CPU only? | Yes, comfortably: about 35–60 tokens/s on a laptop |
| How to run | llama.cpp / LM Studio with the official GGUF (no Ollama library page found) |

**Benchmark scores** ([model card](https://huggingface.co/LiquidAI/LFM3-1.7B), checked directly)
| Benchmark | Score | Your need | Meets? | Source |
|---|---|---|---|---|
| IFEval | 96.66 | High | ✅ | model card |
| BFCL V4 | 71.39 | High | ✅ | model card (vendor run) |
| IFBench | 73.03 | – | – | model card |
| GSM8K | n/r | Medium | ❌ (not reported) | – |
| MMLU, MATH, HumanEval, MBPP | n/r | Low | – | – |

**Watch out for:**
- **License:** LFM Open License v1.0. Per our research, commercial use is limited to organizations under $10M in annual revenue. That's fine for an indie game, but read the license before shipping.
- **Thinking mode:** on by default, and the card says turning it off "can be inconsistent". Test response times in your game loop.
- **Vendor-only scores:** a 1.7B model beating 9–14B models is a vendor claim with no independent check yet. Test it yourself before trusting it.
- **Tool-call format:** calls come out as XML (`<tool_call><function=...>`), so your parser has to handle that.

## 4. Qwen3.5-9B (Alibaba)
**Why it fits:** It has the second-best IFEval in the pool (91.5) and a strong BFCL v4 (66.1). It's a recent release (Feb 2026) with a very long context and wide tooling support (Ollama, vLLM, llama.cpp).

**Minimum system requirements**
| | Value |
|---|---|
| Min memory (Q4_K_M) | 7.2 GB |
| Comfortable (Q8_0) | 11.0 GB |
| Download size (Q4_K_M) | 5.68 GB ([unsloth](https://huggingface.co/unsloth/Qwen3.5-9B-GGUF)) |
| Context length | 262K tokens (extendable to about 1M) |
| Runs on CPU only? | Yes, slowly: about 7–12 tokens/s. A GPU is recommended. |
| How to run | `ollama run qwen3.5:9b` / LM Studio search "Qwen3.5-9B" |

**Benchmark scores** ([model card](https://huggingface.co/Qwen/Qwen3.5-9B), thinking mode)
| Benchmark | Score | Your need | Meets? | Source |
|---|---|---|---|---|
| IFEval | 91.5 | High | ✅ | model card |
| BFCL-V4 | 66.1 | High | ✅ | model card (vendor run) |
| GSM8K | n/r | Medium | ❌ (not reported) | – |
| MMLU-Redux / MMLU-Pro | 91.1 / 82.5 | Low | – | model card |
| TAU2-Bench (agentic) | 79.1 | – | – | model card |
| MATH, HumanEval, MBPP | n/r | Low | – | – |

**Watch out for:**
- It thinks by default. Turn thinking off (`enable_thinking=False`) for per-tick commands. The card gives no scores for non-thinking mode.
- It's multimodal, so the GGUF build may need its vision projector file handled separately.

## 5. Qwen3-14B (Alibaba)
**Why it fits:** It has the strongest BFCL in the Qwen family (v3: 70.4 thinking / 61.5 non-thinking) and solid IFEval. It's also the best-documented model here: a full technical report, plus an entry on the **official** BFCL V4 leaderboard.

**Minimum system requirements**
| | Value |
|---|---|
| Min memory (Q4_K_M) | 10.5 GB |
| Comfortable (Q8_0) | 17.2 GB (over your limit, so stay on Q4/Q5) |
| Download size (Q4_K_M) | 9.0 GB ([official GGUF](https://huggingface.co/Qwen/Qwen3-14B-GGUF)) |
| Context length | 32K native, 131K with YaRN |
| Runs on CPU only? | Slow: about 5–8 tokens/s. A GPU is recommended. |
| How to run | `ollama run qwen3:14b` / LM Studio search "Qwen3-14B" |

**Benchmark scores** ([Qwen3 tech report](https://arxiv.org/abs/2505.09388), Tables 15/16 and 5)
| Benchmark | Score | Your need | Meets? | Source |
|---|---|---|---|---|
| IFEval (strict prompt) | 85.4 thinking / 84.8 non-thinking | High | ✅ | tech report |
| BFCL v3 | 70.4 thinking / 61.5 non-thinking | High | ✅ | tech report |
| BFCL V4, official leaderboard | 41.03% (FC) | – | – | [BFCL leaderboard](https://gorilla.cs.berkeley.edu/leaderboard.html) |
| GSM8K | 92.49 (base model only) | Medium | ❌ (not counted) | tech report, Table 5 |
| MMLU | 81.05 (base) · MMLU-Redux 88.6 (instruct) | Low | – | tech report |
| MATH-500 | 96.8 thinking | Low | – | tech report |
| MBPP / EvalPlus | 73.40 / 72.23 (base) | Low | – | tech report |

**Watch out for:**
- It's an older release (Apr 2025) with a short native context.
- Thinking is on by default; toggle it per request.

---

## Runners-up (also cleared every High bar)
- **[MiniCPM5-2B](https://huggingface.co/openbmb/MiniCPM5-2B):** IFEval 86.7, BFCL v4 66.6, needs 3.1 GB, Apache-2.0. Score 5.49. A great tiny option with a cleaner license than LFM3.
- **[Qwen3-4B-Instruct-2507](https://huggingface.co/Qwen/Qwen3-4B-Instruct-2507):** IFEval 83.4, BFCL v3 61.9, needs 4.0 GB, Apache-2.0. It never thinks, so replies are instant. Score 5.19.
- **[Granite 4.1 3B](https://huggingface.co/ibm-granite/granite-4.1-3b):** IFEval 82.3, BFCL v3 60.8, needs 3.6 GB, Apache-2.0. Score 5.11.

## Considered but excluded
- **LFM2.5-24B-A2B:** needs 16.03 GB, just over your 16 GB limit. It also has the LFM license cap.
- **Qwen3.5-27B / Qwen3.6-27B / Qwen3.8-27B:** need 18+ GB.
- **Gemma 4 26B-A4B, Granite 4.1/4.2 30B, Gemma 3 27B:** over the memory limit.
- **Gemma 4 12B:** IFEval 97.2, but no published BFCL, so it fails your High tool-calling bar on missing data. Its Tau2 agentic score is 69.0, so it's worth testing anyway. Watch for the reported llama.cpp tool-call template bugs.
- **Mistral Small 3.2 24B:** IFEval 77.8 (third party), just below the High bar. It also needs 15.8 GB.
- **Ministral 3 14B:** IFEval 67–77 and BFCL v3 52.7 (third party), both below the bars.
- **Llama 3.1 8B:** Meta claims BFCL 76.1, but independent runs get 29–37 (v3) and 25.8% (official V4). The claim doesn't hold up.
- **OLMo 3 7B:** BFCL 49.8, below the bar.
- **SmolLM3-3B:** IFEval 76.7, below the bar.
- **Phi-4:** IFEval 63, below the bar.
- **Phi-4-mini:** IFEval 70.1, below the bar.
- **Gemma 3 12B:** official BFCL V4 30.4%, below the bar.
- **Granite 4.2 8B:** BFCL v4 52.4 and no IFEval reported.
- **Nemotron-3.5-Bolt-E4B:** no IFEval reported.
- **DeepSeek R1 distills:** no reported tool-use scores, and the chat template has no tool support.
- **EXAONE 4.0, xLAM-2, Hammer 2.1, Command R7B:** non-commercial licenses.

## Caveats
- **BFCL numbers come in three kinds and aren't comparable:**
  - Vendor-run v3 (Granite, Qwen3)
  - Vendor-run v4 (LFM3, Qwen3.5, MiniCPM5)
  - The official V4 leaderboard, where the best small models score only about 40–50%
  
  Most 2026 models aren't on the official board yet (last updated 2026-04-12).
- **Many headline scores are in thinking mode** (Qwen3.5, LFM3, Gemma 4). An RTS needs fast replies, so you'll likely run with thinking off, and those models may score lower that way.
- **Newer model cards have dropped GSM8K, MMLU, HumanEval and MBPP** in favor of MMLU-Pro/Redux, LiveCodeBench, IFBench and Tau2. That's why many cells say n/r.
- **Benchmarks are a filter, not the final answer.** Run your top 2–3 on 20–50 real game situations (for example, "here's the game state as JSON, call the right commands") with promptfoo + Ollama, and measure response time while the game is running.
