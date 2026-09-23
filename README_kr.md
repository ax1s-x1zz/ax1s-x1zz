# ax1s-x1zz

**한국 고등학교 2학년 학생입니다. 데이터 파이프라인 DSL, 컴파일러, AI/ML 도구를 Rust로 개발하고 있습니다.**

[English Document](./README.md)

---

## 무엇을 만드나

컴파일러와 그 언어를 직접 작성합니다. 주요 작업은 스크립트를 최적화된 Polars 실행 계획으로 바꾸는 일련의 DSL이며, lexer부터 codegen까지 Rust로 처음부터 구축합니다. 그 핵심 주변에는 웹 프레임워크 브리지, 비주얼 에디터, Python 변환기(transpiler), LLM 효율성 응용 연구가 있습니다.

설계 방향은 모든 프로젝트에서 동일합니다: **오류를 런타임이 아닌 컴파일 타임으로 이동**시키고, 무거운 의존성을 서브프로세스 경계 뒤에 격리해 CLI를 가볍게 유지하며, CSV에서 학습 완료 모델까지의 전체 경로를 단일 스크립트로 표현합니다.

**현재 개발 중:** [Xz](https://github.com/x1zzdev/Xz) — AI가 쓴, 사람이 검토하는 코드를 위한 언어 — 와 이를 Next.js·Rails로 가져오는 브리지.

---

## 주요 프로젝트 (Featured)

### [Xazz](https://github.com/x1zzdev/Xazz) — AI 파이프라인 DSL

Polars 전처리, Burn 딥러닝 컴파일, 정적 보안 가드레일을 단일 `.xzz` 스크립트에 통합하는 Rust 기반 AI 파이프라인 DSL입니다.

- **기술 스택**: Rust (edition 2024), Polars, Burn, Axum / Tokio
- lexer → parser → 정적 타입 체커 → Rust/Polars/Burn codegen으로 이어지는 컴파일러 툴체인을 처음부터 구축.
- `Option<T>` 기반 컴파일 타임 null/타입 안전성, `line:col` 진단 및 did-you-mean 제안.
- Zero-copy: Apache Arrow 버퍼를 Burn에 직접 전달해 pandas→NumPy→PyTorch 복사 경계 제거.
- 가드레일: Policy-as-Code PII/시크릿 탐지, 세션별 엡실론 예산의 차등 프라이버시, SHA-256 append-only 감사 로그.
- `xazz-runner` 서브프로세스 경계 뒤에 무거운 엔진을 격리해 CLI 바이너리를 2–5MB로 유지하는 멀티 크레이트 워크스페이스.

<p align="center">
  <img src="assets/ide_monitor.png" alt="Xazz IDE 모니터" width="600"/>
</p>

### [Xz](https://github.com/x1zzdev/Xz) — AI가 쓴 코드를 위한 언어

**AI-written, Human-reviewed**라는 하나의 명제를 중심으로 설계한 실험적 범용 프로그래밍 언어입니다 — 모든 동작은 명시적이고, 모든 계약은 가시적이며, 모든 실패 경로는 타입으로 표현됩니다.

- **기술 스택**: Rust (inkwell / 이식형 LLVM 17), nom / pest, 생성된 Python `ctypes` 바인딩
- **인텐트 검증** (핵심 차별점): `@intent` / `@requires` / `@ensures` / `@effects` 주장(claim)을 `pre` / `post` 계약과 구조적으로 짝지어 본문과 대조 검증하고, `@trusted` 사람 검토 escape hatch를 제공.
- 처음부터 직접 구현한 프런트엔드 + LLVM 백엔드: JIT(`xz run`), 네이티브 바이너리, C ABI 공유 라이브러리(`libXz.so` + `libXz.h`).
- 안정적 에러 코드·스팬·신뢰도 점수 기반 수정안을 담은 구조적 JSON 진단 — LLM 자기수정 루프를 목표로 설계.
- 상태: Phase 1–4 구현 완료, Phase 5(FFI) 진행 중, `hello.xz`·`contracts.xz` 실행 가능.

---

## Xz 생태계

```
[Python 코드] --> (py2xzz) ---\
                               --> [.xzz 스크립트] --> (Xazz 컴파일러) --> [실행 / Burn ML]
[비주얼 드래그앤드롭] (IDE) ----/
```

| 프로젝트 | 설명 | 기술 |
|---|---|---|
| [next.xz](https://github.com/x1zzdev/next-xz) | Next.js × Xz 브리지 — Bun/Node FFI, Agent 자가수정 루프, `/___audit` 오버레이. *단독 프로젝트.* | TypeScript · Bun · Next.js |
| [rails.xz](https://github.com/imrubydev/rails-xz) | Rails × Xz 브리지 — Ruby FFI, Rails Engine 감사 대시보드. *[imrubydev](https://github.com/imrubydev)와 공동: ax1s는 브리지/에이전트, imrubydev는 Engine·DX 담당.* | Ruby · Rails |
| [x1zzLang](https://github.com/x1zzdev/x1zzLang) | Xazz로 성장한 데이터 파이프라인 DSL. | Rust · Polars |
| [py2xzz](https://github.com/x1zzdev/py2xzz) | Python (Pandas / PyTorch) → `.xzz` 변환기. | Rust |
| [x1zzLang Visual IDE](https://github.com/x1zzdev/x1zzLang-visual-ide) | `.xzz`를 생성·실행하는 드래그 앤 드롭 DAG 에디터. | React |
| [LLM PCAG 연구](https://github.com/ax1s-x1zz/llm-pcag-research) | LLM 가중치 양자화의 에너지 비용과 그로 인한 그리드 Jevons 역설. | Python |

<p align="center">
  <img src="assets/fig15_dashboard.png" alt="LLM PCAG 대시보드" width="600"/>
</p>

---

## 오픈소스 기여 (Open Source Contributions)

**[tracel-ai/burn](https://github.com/tracel-ai/burn)**과 **[apache/arrow-rs](https://github.com/apache/arrow-rs)**에 업스트림 기여 — 머지된 PR 6개. 그중 하나는 리뷰 과정에서 공통 레이어 수정으로 재작업.

**모든 백엔드에 `Min` / `Max` `scatter` / `select_assign` 구현** — [머지된 PR #5582](https://github.com/tracel-ai/burn/pull/5582). [#5522](https://github.com/tracel-ai/burn/issues/5522)가 남긴 마지막 공백을 채웠습니다: `Assign` / `Add` / `Mul`은 지원됐지만 `Min` / `Max`는 모든 백엔드에서 `unimplemented!`였습니다. ndarray, flex, cubecl, tch 전반은 물론 autodiff backward까지 end-to-end로 구현했습니다.

<details>
<summary><b>#5582 상세와 나머지 머지 PR 5개</b></summary>

- **네 백엔드 전부**: ndarray 프리미티브, 기존 업데이트 워커를 공유하는 `flex` 헬퍼, `BinaryMinOp` / `BinaryMaxOp` 커널을 재사용하는 `cubecl` 항목, `scatter_reduce` / `index_reduce_`(`"amin"` / `"amax"`)를 쓰는 `tch`.
- **Autodiff**: `Min` / `Max` `scatter`와 `select_assign`의 backward 패스를 `scatter_nd` Min/Max 그래디언트와 동일하게 구현 — 비교 기반 winner 마스크, 동률은 양쪽 입력에 귀속, unique index 요구.
- 이제 dispatch match가 모든 `IndexingUpdateOp` 변형을 열거하므로, 지원되지 않는 조합은 런타임 panic 대신 컴파일 타임에 실패합니다.
- `--features ndarray`로 텐서 1839개 + autodiff 572개 테스트 통과, `burn-ndarray`·`burn-flex`·`burn-autodiff`·`burn-cubecl` clippy 클린.
- **[burn #5555](https://github.com/tracel-ai/burn/pull/5555)** — `TensorCheck::matmul`에 배치 차원 브로드캐스트 검증 추가 (메인테이너 방향에 따라 #5542를 공통 `TensorCheck` 레이어로 재작업).
- **[burn #5564](https://github.com/tracel-ai/burn/pull/5564)** — `remainder`, `powi`, `powf`, `hypot`, `atan2`에 `TensorCheck`(`binary_ops_ew`) 적용, 각각 회귀 테스트 추가.
- **[burn #5580](https://github.com/tracel-ai/burn/pull/5580)** — `TensorCheck::matmul`에 rank 검증 추가; rank < 2가 백엔드별 panic 대신 명확한 에러를 반환.
- **[burn #5639](https://github.com/tracel-ai/burn/pull/5639)** — `burn-std`의 no-std 빌드 실패 수정 (누락된 `use alloc::vec;` 추가).
- **[arrow-rs #11005](https://github.com/apache/arrow-rs/pull/11005)** — IPC run-ends 재인코딩에서 `BufferBuilder`를 `Vec`으로 교체; 메인테이너 2명 승인 후 머지. 후속 [#11006](https://github.com/apache/arrow-rs/pull/11006)은 리뷰 중.

</details>

---

## 대외 활동 및 리더십 (Activities & Leadership)

<details>
<summary><b>Trendsetter — 창립자 및 리드 아키텍트</b> · 2026.03 – 현재</summary>

반도체 및 최신 기술 중심의 데이터 분석 동아리 `Trendsetter`를 창설했습니다. 온보딩 로드맵, 역량 진단 양식, 마크다운/Python 가이드, 오픈소스 Colab 스타터 키트([CPU vs. GPU](https://github.com/ax1s-x1zz/trendsetter-semiconductor-01-cpu-vs-gpu), 무어의 법칙·황의 법칙 로그 스케일 회귀, 산업통상자원부/통계청 수출 데이터 템플릿)를 직접 설계·배포했습니다.

</details>

<details>
<summary><b>코드게이트 AI 스타트업 해커톤 (Xazz / x1zz Guard)</b> · 2026.07 — 팀 리더</summary>

팀 & IP 거버넌스(상금 1/N 정산, 기존 개인 자산과 대회 산출물 IP 분리), 팀원 이탈 시 위기 대응, 심사위원 예상 질의 대응책 마련, 그리고 Rust·온프레미스 sLM(Qwen2.5-Coder) 기반 AST 정적 보안 통제 계층 `x1zz Guard` 설계를 담당했습니다.

</details>

<details>
<summary><b>GEEKs 해커톤</b> · 2026.08 — 팀 기획자 및 발표자</summary>

실현 불가능한 B2B PaaS에서 B2C 플랫폼(`우선해줘`)으로 전략적 피봇을 주도하고, Vision AI + PostGIS 공공데이터 기반 행정 정당성 스코어링 로직을 공동 기획했으며, 최종 8분 발표를 전담했습니다.

</details>

---

## 기술 스택 요약

| 영역 | 기술 |
|---|---|
| 시스템 / DSL | Rust (edition 2024), Cargo workspace, clap, serde |
| 데이터 엔진 | Polars (LazyFrame), Apache Arrow |
| 딥러닝 | Burn, zero-copy 텐서 전달 |
| 웹 / API | Axum, Tokio, React 18, Next.js, Vite, @xyflow/react |
| 웹 프레임워크 | Rails (Hotwire, ViewComponent, Turbo), Ruby FFI |
| 백엔드 통합 | Rust REST API, SHA-256 감사 로깅, TypeScript/Bun FFI (koffi) |
| 연구 / 분석 | Python, NumPy, pandas, SciPy, SymPy, Matplotlib |

---

## 소개

이 프로젝트들은 타입 안전하고 컴파일되는 데이터 파이프라인을 향한 지속적인 탐구의 일부입니다. 컴파일러 설계, 데이터 도구, ML 인프라스트럭처에서 비슷한 문제를 다루고 있다면 언제든 연락주세요.

- **이메일**: [ax1s@x1zz.com](mailto:ax1s@x1zz.com)
