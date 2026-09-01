# ax1s-x1zz

**High school student (2nd year) in South Korea — building data pipeline DSLs, compilers, and AI/ML tooling in Rust.**

[한국어 문서 (Korean)](./README_kr.md) · [**x1zz.com**](https://x1zz.com)

---

## Overview

**Building type-safe, compiled data pipeline DSLs and high-performance ML infrastructure in Rust.**

I build data pipeline languages and the tooling around them. My main line of work is a family of domain-specific languages that compile `.xzz` scripts into optimized Polars execution plans, backed by compilers written from scratch in Rust. Alongside the language itself, I work on visual editors, source-to-source transpilers, and applied research on LLM efficiency.

The core design direction: move errors from runtime to compile time, keep the CLI lightweight by isolating heavy dependencies behind a subprocess boundary, and make the whole path from CSV to trained model expressible in a single script.

---

## Ecosystem

```
[Python Code] --> (py2xzz) ---\
                               --> [.xzz Script] --> (Xazz Compiler) --> [Exec / Burn ML]
[Visual Drag&Drop] (IDE) ----/
```

x1zzLang is the foundation that grew into Xazz; py2xzz and the Visual IDE feed `.xzz` scripts into the Xazz compiler.

---

## Pinned Projects

### [Xazz](https://github.com/x1zzdev/Xazz) — AI Pipeline DSL

An AI pipeline DSL in Rust that unifies Polars preprocessing, Burn deep-learning compilation, and static security guardrails in one `.xzz` script.

- **Tech**: Rust (edition 2024), Polars, Burn, Axum / Tokio
- **Implementation highlights**:
  - Full compiler toolchain built from scratch: lexer → parser → static type checker → Rust/Polars/Burn codegen
  - Compile-time null and type safety via an `Option<T>` type system, with `line:col` diagnostics and did-you-mean suggestions
  - Zero-copy path: Apache Arrow buffers handed directly to Burn, avoiding the pandas→NumPy→PyTorch copy boundaries
  - Policy-as-Code guardrails (PII/secret detection), differential privacy with a per-session epsilon budget, and a SHA-256 append-only audit log
  - Multi-crate Cargo workspace that keeps the CLI binary 2–5 MB by isolating heavy engines behind the `xazz-runner` subprocess boundary

### [x1zzLang](https://github.com/x1zzdev/x1zzLang) — Data Pipeline Language
A Rust-based DSL for exploring accessible data analysis, compiling `.xzz` scripts into optimized Polars LazyFrame execution plans.

- **Tech stack**: Rust, Polars, clap, serde
- **Implementation highlights**:
  - Full compiler pipeline (lexer/parser/codegen/emitter) in Rust
  - Null-safe `Option<T>` type system with a `fillNull` operator
  - `x1zz import` auto-infers CSV schemas (including EUC-KR/CP949 decoding) and generates type declarations
  - `x1zz emit rust` transpiles `.xzz` into standalone Polars LazyFrame Rust source
  - Dependency isolation: the CLI never links Polars — execution is delegated to a spawned subprocess
  - The foundation that later grew into Xazz

### [py2xzz](https://github.com/x1zzdev/py2xzz) — Python → `.xzz` Transpiler
A Rust CLI tool that converts Python data and deep-learning pipelines (Pandas / PyTorch) into `.xzz` DSL scripts.

- **Tech stack**: Rust, serde
- **Implementation highlights**:
  - Self-contained Python 3 lexer/parser producing an AST mirroring the Python `ast` module spec
  - A mapper that turns Pandas chains into `PipelineOp` chains and `nn.Module` classes into `ModelDecl`/`LayerKind`
  - Column types inferred from CSV headers and sample values, wrapped in `Option<...>` when nulls are present
  - A span map traces original Python line/column positions to emitted statements for diagnostic reporting
  - Output maps 1:1 to the `xazz-core` AST and passes `xazz check`

### [x1zzLang Visual IDE](https://github.com/x1zzdev/x1zzLang-visual-ide)
A graphical pipeline editor for x1zzLang that designs DAG workflows and runs them natively.

- **Tech stack**: React 18, Vite, @xyflow/react, i18next
- **Implementation highlights**:
  - Drag-and-drop DAG builder with 9 built-in pipeline operators
  - Real-time transpilation from the visual graph to `.xzz` source via a dedicated transpiler engine
  - One-click execution against the backend with tabular results
  - Multi-workflow tabs, undo/redo, auto-save, container grouping, and Korean/English UI

### [LLM PCAG Research](https://github.com/ax1s-x1zz/llm-pcag-research) — The Power Wall of LLM Quantization
Research quantifying the energy-saving efficiency of LLM weight quantization and the macro-grid Jevons paradox.

- **Tech stack**: Python (NumPy, pandas, SciPy, SymPy, Matplotlib)
- **Implementation highlights**:
  - Defines the PCAG metric (Power Cost per Accuracy Gain) to measure when quantization efficiency collapses faster than accuracy loss
  - Identifies the INT4→INT3 "Power Wall" via three independent paths: empirical PCHIP, Monte Carlo, and an analytic model
  - Proves the inflection root is structurally independent of amplitude
  - Formalizes the Jevons Paradox in closed form with a symbolic proof in SymPy (grid load increases iff demand elasticity E_d > 1)
  - Reproducible experiment pipeline with strict data-source labeling (reference-literature vs GPU-measured)

---
### Activities & Leadership

#### **CodeGate AI Startup Hackathon & Project (`Xazz / x1zz Guard`) (2026.07)**
> **Role:** Team Leader, Project Manager, Core Toolchain Architect
- **Proactive Team Management & IP/Reward Alignment:** Set clear onboarding guidelines, establishing transparent rules for equal prize distribution (1/N) and explicitly separating pre-existing core IP (`x1zzLang` toolchain) from new co-developed assets (guardrail rulesets, PPTs) to foster psychological safety.
- **Agile Crisis Recovery & Scope Re-engineering:** When a key team member departed right before submission, swiftly restructured team roles—balancing core architecture refinement (governance/data bypass scenarios) and submission polishing (2-page summary, BM narrative) to hit the deadline.
- **Strategic Technical Defense:** Preemptively identified potential judge objections (e.g., query correlation overhead, DP noise-induced accuracy loss) and devised defensive arguments, including window-based privacy filtering and adaptive noise injection strategies.
- **Core DSL & Security Control Layer Engineering:** Designed and implemented **`x1zzLang`**, an isolated AI DSL written in Rust. Built an end-to-end toolchain combining Polars LazyFrame acceleration, Burn deep learning execution, and `x1zz Guard` (an AST-based static policy gate with an on-premise Qwen2.5-Coder sLM for dynamic code remediation).

#### **GEEKs Hackathon (2026.08.04 - 2026.08.05)**
> **Role:** Team Presenter, Strategic Pivot Lead, Product/UX Validator
- **Technical Pragmatism & Strategic Pivot:** During an intensive 2-day overnight hackathon under the "Sustainable Infrastructure (SDG 9)" theme, evaluated the newly formed team's tech stack constraints. Recognized that a complex B2B PaaS item was unfeasible within the timeframe, leading a decisive pivot toward a user-centric B2C civil service platform (`Woosen-haejo` / '우선해줘').
- **Inclusive Team Collaboration & Objective Alignment:** Embraced an unfamiliar target domain, driving team alignment by focusing on user experience (UX) validation and co-designing administrative justification scoring logic (Claude Opus 5 Vision AI + PostGIS open data integration).
- **Presentation & Outcome Delivery:** Fully owned the pitch preparation and successfully delivered the final 8-minute presentation, demonstrating strong adaptability, cross-functional alignment, and leadership under pressure.

---
## Tech Stack Summary

| Area | Technologies |
|---|---|
| Systems / DSL | Rust (edition 2024), Cargo workspace, clap, serde |
| Data engine | Polars (LazyFrame), Apache Arrow |
| Deep learning | Burn, zero-copy tensor handoff |
| Web / API | Axum, Tokio, React 18, Vite, @xyflow/react |
| Backend integration | Rust REST API, SHA-256 audit logging |
| Research / analysis | Python, NumPy, pandas, SciPy, SymPy, Matplotlib |

---

## About

These projects are part of an ongoing exploration of type-safe, compiled data pipelines. If you're working on similar problems in compiler design, data tooling, or ML infrastructure, I'd be glad to talk.

- **Email**: [ax1s@x1zz.com](mailto:ax1s@x1zz.com)
