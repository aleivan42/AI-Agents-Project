# AI Agents Project

Multi-agent LLM benchmarking pipeline built with **AGNO** and self-hosted via **Ollama**, comparing four open-source language models on a prompt-driven text-rewriting task. Includes structured output tracing (JSONL), automated evaluation, and manual qualitative analysis of model behaviour and failure modes.

## Overview

The project evaluates how different LLMs handle a controlled text-rewriting task under a fixed set of prompt guidelines, measuring both rule compliance (via automated metrics) and qualitative fidelity to the source text (via manual review).

**Models compared:** Llama 3.1 8B · Mistral 7B · Qwen 2.5 7B · Falcon 7B
**Framework:** AGNO (agent framework), models served locally via Ollama
**Compute:** Google Colab, GPU runtime (T4, then A100)

> **Note on environment:** Colab notebooks aren't standard practice for production ML work, but were used here specifically to access powerful GPUs (T4/A100) and to run models locally within the runtime via a self-hosted Ollama server — natively compatible with AGNO — removing dependence on external API rate limits (e.g. HuggingFace Inference API).

## Project Structure

| Notebook | Purpose |
|---|---|
| `AI_Agents_Project_Data_Treatment.ipynb` | Preliminary dataset processing (Module 3) |
| `AI_Agents_Project_Data_Analysis.ipynb` | Preliminary dataset exploration, cleaning of guideline-violating rows, and automated results analysis (Module 5) |
| `AGNO_Pipeline.ipynb` | The core agent framework (Modules 1, 2, 4) — self-contained, includes the relevant definitions from the Data Treatment notebook |

## Deliverables

**1. Processed data** — available in `data_evaluated/`.

**2. Prompt effectiveness** — the provided prompts worked reliably for 3 of 4 models. **Falcon 7B failed consistently across every test**, regardless of input text — including a minimal sanity-check ("Are you online?") run on the Test Agent within `AGNO_Pipeline`. Failure modes included irrelevant output, stray/garbled text, and empty generations. Likely explanation: Falcon 7B is poorly suited to conversational/instruction-following tasks without highly specific prompt engineering. No test across any model achieved a perfect score (7/7 metrics, including length compliance). Data in `Agents_Project_Traces/`.

**3. Structured traces** — JSONL output files, split by dataset and model, in `Agents_Project_Traces/`. Note: additional informal tests were run beyond what's documented here, before the JSONL tracing system was implemented.

**4. Qualitative analysis** — manual review of the first 12 rows of the `adv_ele` dataset across all 4 models. Summary below (full detail condensed from per-scenario notes):

| # | Topic | Falcon 7B | Llama 3.1 8B | Mistral 7B | Qwen 2.5 7B |
|---|---|---|---|---|---|
| 1 | .amazon domain dispute | Incoherent — degenerates into a repeated unrelated sentence | Coherent but expanded 4× with sophisticated language | Coherent — adds complexity without altering facts | Very faithful — more formal, structure preserved |
| 2 | Tourism & liberalism in Amsterdam | Critical failure — leaked code artifacts | High fidelity — expands "tolerance" into a fuller analysis | Coherent — preserves irony/contrast | Concise — brief, factually accurate |
| 3 | Google Maps & isolated communities | Critical failure — empty/repeated token | Moderate drift — ~6× expansion, adds unstated concepts | Coherent — adds logistics detail, stays on topic | Faithful — more formal, no invention |
| 4 | Banksy mural auction | Partially coherent — ends in leaked system text | Very detailed — expands into a short essay, facts preserved | Coherent — adds plausible financial detail | Accurate — precise, faithful reporting |
| 5 | Global wealth inequality | Critical failure — repetition loop | Thematic drift — shifts into a "manifesto" framing | Coherent — stylistic addition, correct stats preserved | Precise — preserves exact figures |
| 6 | BlackBerry's decline | Incoherent — repeats instruction text | Evocative — dramatizes tone, historical accuracy kept | Coherent — expands political/social context correctly | Faithful — preserves Q&A structure and tone |
| 7 | Indigenous rights disputes | Critical failure — repetition loop | Expansive — links to global themes, dilutes case specifics | Meta-textual — erroneously includes prompt instructions in output | Excellent — synthesises position with precise terms |
| 8 | Gender pay gap | Incoherent — repeats rule text | Positive drift — adds relevant external context | Coherent — retains informative core | Factual — cites specific figures accurately |
| 9 | Brazil protests | Critical failure — repetition loop | Rhetorical — shifts tone toward political commentary | Coherent — stays aligned with source | Analytical — correctly links causal context |
| 10 | Tobacco / e-cigarette regulation | Incoherent — repeats system text | Speculative — coherent, discusses future implications | Coherent — balances both sides of the debate | Highly expanded (5×) — detailed but verbose |
| 11 | Climate change & poor countries | Critical failure — repetition loop | Generic — correct but clichéd phrasing | Partial hallucination — invents a specific (unverified) report date/location | Urgent tone — maintains original urgency |
| 12 | Coal vs. oil | Partially coherent — ends in leaked system text | Solution-drift — shifts focus from problem to solutions | Analytical — correctly introduces shale gas as a counter-trend | Factual — stays focused on efficiency benefits |

**5. Metrics & full traces** — available in `Agents_Project_Traces/`.

## Key Findings

- **Falcon 7B** failed systematically across all 12 scenarios — not a data issue, but a model/task mismatch requiring far more specific prompt engineering than the other three models.
- **Qwen 2.5 7B** was consistently the most faithful to source content and instructions.
- **Llama 3.1 8B** tended to expand and embellish, occasionally drifting thematically from the source.
- **Mistral 7B** was generally balanced and coherent, with one notable partial hallucination (an invented but plausible-sounding factual detail).
- No model achieved a perfect score on all 7 automated compliance metrics across any test.
