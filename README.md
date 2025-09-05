# Prompt‑Engineer: IMO‑style Multi‑Agent Prompts for Structural Analysis

> **Status:** research prototype · **Scope:** LLM multi‑agent prompts for structural engineering (frame analysis, pressure‑vessel FEA)  
> **Disclaimer:** The “IMO‑style pipeline” below is *reconstructed from public statements; not an official spec*. Use with human review for safety‑critical work.

This repository provides **multi‑agent prompts** that emulate an **IMO‑style reasoning pipeline**—
**Solver → Self‑Improvement → Verifier → Correction → Synthesis**—so that large language models (LLMs) can perform **explicit calculation, verification, and reporting** for structural‑engineering tasks.

Two reference cases are included in the accompanying article:
- **Case A:** Indeterminate frame (reactions, member forces, moment diagram) — *LLM‑only reasoning*  
- **Case B:** Thin‑walled pressure vessel — *LLM + MCP‑mediated FEA calls* (axisymmetric model, mesh‑convergence, energy checks)

The prompts are vendor‑agnostic. You can run them with any capable LLM (e.g., Gemini 2.5 Pro, etc.) and optional external tools through **Model Context Protocol (MCP)**.

---

## Contents
- [What this repo is](#what-this-repo-is)
- [Pipeline at a glance](#pipeline-at-a-glance)
- [Prompt files & layout](#prompt-files--layout)
- [Prompt lint (quality checklist)](#prompt-lint-quality-checklist)
- [Case references](#case-references)
- [How to run](#how-to-run)
- [Reproducibility & logging](#reproducibility--logging)
- [Safety & scope](#safety--scope)
- [References](#references)
- [Korean summary](#korean-summary)

---

## What this repo is
**Goal:** Make LLM reasoning *traceable and checkable* for engineering problems by forcing staged outputs. Each stage emits **(i) a short human‑readable summary** and **(ii) a strict JSON block** consumed by the next stage. This mitigates long‑context loss and enables downstream automation.

**Design principles**
- Stage‑specific roles (solver, critic, verifier, fixer, synthesizer)
- Explicit units (SI with thin spaces) and sign‑conventions
- Independent double‑checks for critical scalars
- Machine‑readable hand‑offs (JSON schema shared across stages)
- Minimal edits at the **Correction** stage; no “rewrite from scratch” unless required by the verifier

---

## Pipeline at a glance

| Stage | Purpose | Must consume | Must produce (JSON) | Hard checks |
|---|---|---|---|---|
| **1. Solver** | Parse problem, choose method (e.g., displacement method; 2D axisymmetry) | Problem text/images; prior constraints | `{assumptions[], variables{}, equations[], plan[]}` | Units present; sign conventions and axes declared |
| **2. Self‑Improvement** | Critique Solver; patch weak assumptions | Solver JSON | `{fixes[], revised_plan[], risks[]}` | Catch missing boundary conditions/material data |
| **3. Verifier** | Independent validation (equilibrium, compatibility, code checks) | Solver+SI JSON | `{tests[], pass_rate, deltas{}, flags[]}` | *Double‑calculation* for key scalars |
| **4. Correction** | Apply Verifier feedback; re‑solve minimal set | Verifier JSON | `{patched_equations[], updated_values{}, residuals{}}` | Residual < tolerance; dimensional consistency |
| **5. Synthesis** | Final answer + report formatting | All prior JSON | `{tables[], figures[], citations[], final_text}` | No TODOs/“maybe”; numbers are traceable |

> **Note:** Stages must quote prior key numbers to avoid “lost‑in‑the‑middle”.

---

## Prompt files & layout

> You can keep your current root files. The following layout helps automation and testing.

```
/prompts
  01_solver.md
  02_self_improvement.md
  03_verifier.md
  04_correction.md
  05_synthesis.md
  final_answer_template.md
/examples
  caseA_frame/
    input.pdf          # problem statement/drawing
    expected.csv       # reactions, member forces, extrema coordinates
  caseB_pressure_vessel/
    config.yaml        # geometry, material, loads
    expected.csv       # σθ=100 MPa, σz=50 MPa, σv≈88.34 MPa, n≈2.83
    mcp/               # tool schemas, sample JSON I/O, mesh study
/docs
  pipeline_overview.md
  reproducibility.md
/logs
  run_YYYYMMDD_HHMM.jsonl  # per‑stage summaries + tool calls
```

### Suggested naming (map existing files)
- `SolverPrompt.md` → **`01_solver.md`**
- `Self-Improvement Prompt.md` → **`02_self_improvement.md`**
- `Verifier Prompt.md` → **`03_verifier.md`**
- `Correction Prompt.md` → **`04_correction.md`**
- `SYNTHESISPROMPT.md` → **`05_synthesis.md`**
- `FinalAnswer.md` → **`final_answer_template.md`**

*(Remove spaces in filenames to simplify scripting.)*

---

## Prompt lint (quality checklist)

Add a small YAML header to each prompt and keep a single JSON schema across stages.

```yaml
id: solver|self_improvement|verifier|correction|synthesis
version: 0.2
inputs: [problem, prior_json]
outputs: [stage_json, stage_summary_md]
invariants:
  - use SI units with thin space (e.g., 580 kN, 2 MPa, 500 mm)
  - declare sign conventions and coordinate axes
stop_signals: ["FINAL_JSON:", "END_OF_STAGE"]
failure_modes: ["hallucinated code", "silent unit change", "skipped check"]
```

**Rules to enforce**
- Every stage must **echo** prior key numbers (context retention).
- **Verifier** performs **double‑calculation** via independent routes (e.g., equilibrium vs energy method).
- **Correction** applies **minimal edits** and emits `residuals`; if tolerance not met, loop once more.
- **Synthesis** emits **tables with units** and **error columns** (abs/rel) and lists generated figures (SVG).  
- Prohibit vague language (“likely”, “probably”) in final answers.
- Keep a `/prompts/schema.json` that defines the stage JSON for all five stages.

---

## Case references

### Case A — Indeterminate frame (exam‑style)
- Parse text+drawing, classify as CALC, extract geometry & loads.
- Solve using the displacement method; output **reactions**, **member forces**, and **moment diagram**.
- Report **extrema values with coordinates** and an **error table** against a baseline (if available).

### Case B — Thin‑walled pressure vessel (FEA via MCP)
- Model as **2D axisymmetric**.
- Baseline theory: \(\sigma_\theta=pD/2t=100\,\text{MPa}\), \(\sigma_z=pD/4t=50\,\text{MPa}\).  
  Include radial stress \(\sigma_r=-p\) for von Mises \(\approx88.34\,\text{MPa}\).  
  With \(\sigma_y=250\,\text{MPa}\), safety factor \(n\approx2.83\).
- Run **mesh‑convergence** (coarse/medium/fine) and **energy check** (internal strain energy vs external work).  
- Save **tool I/O JSON**, **mesh table**, and **energy numbers** under `/examples/caseB_pressure_vessel/mcp/`.

---

## How to run

1. Choose a model and set conservative decoding (`temperature≈0.2`, generous `max_output_tokens`).  
2. Execute the five prompts in order. On each step, parse the `FINAL_JSON:` block and feed it to the next stage.  
3. **Case A:** export an SVG moment diagram and a table of extrema (value, x‑coordinate).  
4. **Case B:** call the FEA tool via **MCP** only when needed; record all tool calls and results. Provide a short **sensitivity study** (e.g., ±1% pressure).

> The repo is runner‑agnostic; you can wire these prompts into any agent framework or call them sequentially from a small script.

---

## Reproducibility & logging

- Persist all stage JSONs + tool calls in `/logs/*.jsonl` (one line per stage).  
- Pin prompt versions and keep a short **CHANGELOG** in each file header.  
- Publish example inputs and **expected outputs** so others can verify.  
- If releasing FEA scripts/tooling, document **solver + version**, **element type/integration order**, **BCs**, and **convergence criteria**.

---

## Safety & scope

This project assists with **drafting and analysis**. For **safety‑critical** tasks (building codes, pressure vessels), outputs **must** be reviewed by a qualified engineer; the final responsibility rests with the reviewer.

---

## References

Authoritative sources to cite in your docs/runs:

- OpenAI, *Learning to reason with LLMs (o1)* — reasoning and evaluation details.  
- Google DeepMind, blog posts on IMO‑level performance (2024–2025).  
- Anthropic, *Model Context Protocol (MCP)* — protocol overview and spec.  
- Google AI for Developers — **Gemini 2.5 Pro** model docs.  
- FEniCSx Documentation — examples for axisymmetric analysis.  
- NAFEMS — guidance on **mesh convergence** best practices.  
- ASME BPVC Sec. VIII‑1 (UG‑27) and **Roark’s** for thin‑walled pressure‑vessel theory.

(Use official sources; avoid secondary summaries.)

---

## Korean summary

이 저장소는 **IMO‑식 5단계 멀티에이전트 프롬프트**(풀이→자체개선→검증→수정→종합)를 제공하여  
**사례 A(부정정 프레임)**과 **사례 B(박벽 압력용기 FEA)**를 재현할 수 있게 합니다.  
각 프롬프트 상단에 **YAML 헤더**와 **공통 JSON 스키마**를 두고, **메시 수렴·에너지 검증** 같은 필수 점검을 JSON으로 남기세요.  
안전‑중요 업무에서는 반드시 **전문가 검토**를 거치십시오.

---

*Copyright © 2025. Released as research materials; see repository license for details.*
