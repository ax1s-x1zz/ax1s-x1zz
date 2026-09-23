# ax1s-x1zz

**High school student (2nd year) in South Korea — building data pipeline DSLs, compilers, and AI/ML tooling in Rust.**

[한국어 문서 (Korean)](./README_kr.md) · [**x1zz.com**](https://x1zz.com)

---

## What I build

I write compilers and the languages they compile. My main line of work is a family of DSLs that turn scripts into optimized Polars execution plans, built from scratch in Rust — lexer to codegen. Around that core sit web-framework bridges, a visual editor, a Python transpiler, and applied research on LLM efficiency.

The design direction is constant across all of them: **move errors from runtime to compile time**, keep the CLI small by pushing heavy dependencies behind a subprocess boundary, and make the whole path from CSV to trained model expressible in a single script.

**Currently building:** [Xz](https://github.com/x1zzdev/Xz) — a language for AI-written, human-reviewed code — and bridges that bring it to Next.js and Rails.

---

## Featured Projects

### [Xazz](https://github.com/x1zzdev/Xazz) — AI Pipeline DSL

An AI pipeline DSL in Rust that unifies Polars preprocessing, Burn deep-learning compilation, and static security guardrails in one `.xzz` script.

- **Tech**: Rust (edition 2024), Polars, Burn, Axum / Tokio
- Full compiler toolchain from scratch: lexer → parser → static type checker → Rust/Polars/Burn codegen.
- Compile-time null/type safety via `Option<T>`, with `line:col` diagnostics and did-you-mean suggestions.
- Zero-copy: Apache Arrow buffers handed directly to Burn, skipping the pandas→NumPy→PyTorch copy boundaries.
- Guardrails: Policy-as-Code PII/secret detection, differential privacy with a per-session epsilon budget, SHA-256 append-only audit log.
- Multi-crate workspace keeps the CLI 2–5 MB by isolating heavy engines behind the `xazz-runner` subprocess boundary.

<p align="center">
  <img src="assets/ide_monitor.png" alt="Xazz IDE Monitor" width="600"/>
</p>

### [Xz](https://github.com/x1zzdev/Xz) — A Language for AI-Written Code

An experimental general-purpose language built on one thesis: **AI-written, Human-reviewed** — every behavior explicit, every contract visible, every failure path typed.

- **Tech**: Rust (inkwell / portable LLVM 17), nom / pest, generated Python `ctypes` bindings
- **Intent verification** (the differentiator): `@intent` / `@requires` / `@ensures` / `@effects` claims are structurally paired with `pre` / `post` contracts and checked against the body, with a `@trusted` human-review escape hatch.
- Full front end from scratch, plus an LLVM backend: JIT (`xz run`), native binaries, and C-ABI shared libraries (`libXz.so` + `libXz.h`).
- Structured JSON diagnostics with stable error codes, spans, and confidence-scored fixes — built for an LLM self-correction loop.
- Status: Phases 1–4 implemented, Phase 5 (FFI) underway; `hello.xz` and `contracts.xz` execute.

---

## The Xz Ecosystem

```
[Python Code] --> (py2xzz) ---\
                               --> [.xzz Script] --> (Xazz Compiler) --> [Exec / Burn ML]
[Visual Drag&Drop] (IDE) ----/
```

| Project | What it is | Tech |
|---|---|---|
| [next.xz](https://github.com/x1zzdev/next-xz) | Next.js × Xz bridge — Bun/Node FFI, agent self-correction loop, `/___audit` overlay. *Solo project.* | TypeScript · Bun · Next.js |
| [rails.xz](https://github.com/imrubydev/rails-xz) | Rails × Xz bridge — Ruby FFI, Rails Engine audit dashboard. *With [imrubydev](https://github.com/imrubydev): ax1s owns the bridge/agent, imrubydev owns the Engine & DX.* | Ruby · Rails |
| [x1zzLang](https://github.com/x1zzdev/x1zzLang) | The data-pipeline DSL that grew into Xazz. | Rust · Polars |
| [py2xzz](https://github.com/x1zzdev/py2xzz) | Python (Pandas / PyTorch) → `.xzz` transpiler. | Rust |
| [x1zzLang Visual IDE](https://github.com/x1zzdev/x1zzLang-visual-ide) | Drag-and-drop DAG editor that emits and runs `.xzz`. | React |
| [LLM PCAG Research](https://github.com/ax1s-x1zz/llm-pcag-research) | Energy cost of LLM weight quantization and the macro-grid Jevons paradox it creates. | Python |

<p align="center">
  <img src="assets/fig15_dashboard.png" alt="LLM PCAG Dashboard" width="600"/>
</p>

---

## Open Source Contributions

Upstream contributions to **[tracel-ai/burn](https://github.com/tracel-ai/burn)** and **[apache/arrow-rs](https://github.com/apache/arrow-rs)** — 6 merged PRs; one reworked across review cycles into a shared-layer fix.

**Implement `Min` / `Max` `scatter` / `select_assign` across every backend** — [merged PR #5582](https://github.com/tracel-ai/burn/pull/5582). Closed the last gap left by [#5522](https://github.com/tracel-ai/burn/issues/5522): `Assign` / `Add` / `Mul` were supported, but `Min` / `Max` hit `unimplemented!` on every backend. Delivered end-to-end across ndarray, flex, cubecl, and tch, including autodiff backward passes.

<details>
<summary><b>Details on #5582 and the other 5 merged PRs</b></summary>

- **All four backends**: ndarray primitives, `flex` helpers sharing the existing update walkers, `cubecl` entries reusing the `BinaryMinOp` / `BinaryMaxOp` kernels, and `tch` via `scatter_reduce` / `index_reduce_` (`"amin"` / `"amax"`).
- **Autodiff**: backward passes for `Min` / `Max` `scatter` and `select_assign`, mirroring the `scatter_nd` Min/Max gradient — winner masks from comparisons, ties credited to both operands, unique indices required.
- Dispatch matches now enumerate every `IndexingUpdateOp` variant, so an unsupported combination fails at compile time instead of panicking at runtime.
- Verified with 1839 tensor + 572 autodiff tests (`--features ndarray`) and clippy-clean on `burn-ndarray`, `burn-flex`, `burn-autodiff`, and `burn-cubecl`.
- **[burn #5555](https://github.com/tracel-ai/burn/pull/5555)** — Batch-dimension broadcast validation in `TensorCheck::matmul` (reworked from #5542 onto the shared `TensorCheck` layer).
- **[burn #5564](https://github.com/tracel-ai/burn/pull/5564)** — Applied `TensorCheck` (`binary_ops_ew`) to `remainder`, `powi`, `powf`, `hypot`, and `atan2`, with regression tests.
- **[burn #5580](https://github.com/tracel-ai/burn/pull/5580)** — Rank validation in `TensorCheck::matmul`; ranks < 2 return a clear error instead of backend panics.
- **[burn #5639](https://github.com/tracel-ai/burn/pull/5639)** — Fixed a no-std build break in `burn-std` (missing `use alloc::vec;`).
- **[arrow-rs #11005](https://github.com/apache/arrow-rs/pull/11005)** — Replaced `BufferBuilder` with `Vec` when re-encoding IPC run-ends; merged with two maintainer approvals. Follow-up [#11006](https://github.com/apache/arrow-rs/pull/11006) under review.

</details>

---

## Activities & Leadership

<details>
<summary><b>Trendsetter — Founder & Lead Architect</b> · 2026.03 – Present</summary>

Founded `Trendsetter`, a semiconductor and tech-focused data-analysis club. Designed the onboarding roadmap, competency diagnostics, Markdown/Python guides, and open-source Colab starter-kits ([CPU vs. GPU](https://github.com/ax1s-x1zz/trendsetter-semiconductor-01-cpu-vs-gpu), Moore's/Huang's Law log-scale regression, MOTIE/KOSIS export-data templates).

</details>

<details>
<summary><b>CodeGate AI Startup Hackathon (Xazz / x1zz Guard)</b> · 2026.07 — Team Lead</summary>

Team & IP governance (1/N reward split, pre-existing IP separated from hackathon assets), mid-project crisis recovery, judge edge-case prep, and the architecture of `x1zz Guard` — an AST-based static policy gate on an on-premise sLM (Qwen2.5-Coder) in Rust.

</details>

<details>
<summary><b>GEEKs Hackathon</b> · 2026.08 — Team Planner & Presenter</summary>

Led the strategic pivot from an unfeasible B2B PaaS to a B2C platform (`Woosen-haejo`), co-designed the admin scoring logic with Vision AI + PostGIS open data, and delivered the final 8-minute pitch.

</details>

---

## Tech Stack

| Area | Technologies |
|---|---|
| Systems / DSL | Rust (edition 2024), Cargo workspace, clap, serde |
| Data engine | Polars (LazyFrame), Apache Arrow |
| Deep learning | Burn, zero-copy tensor handoff |
| Web / API | Axum, Tokio, React 18, Next.js, Vite, @xyflow/react |
| Web frameworks | Rails (Hotwire, ViewComponent, Turbo), Ruby FFI |
| Backend integration | Rust REST API, SHA-256 audit logging, TypeScript/Bun FFI (koffi) |
| Research / analysis | Python, NumPy, pandas, SciPy, SymPy, Matplotlib |

---

## About

These projects are part of an ongoing exploration of type-safe, compiled data pipelines. If you're working on compiler design, data tooling, or ML infrastructure, I'd be glad to talk.

- **Email**: [ax1s@x1zz.com](mailto:ax1s@x1zz.com)
