# ax1s-x1zz

**한국 고등학교 2학년 학생입니다. 데이터 파이프라인 DSL, 컴파일러, AI/ML 도구를 Rust로 개발하고 있습니다.**

[English Document](./README.md)

---

## 소개

컴파일러와 그 언어를 직접 작성합니다. 주요 작업은 `.xzz` 스크립트를 최적화된 Polars 실행 계획으로 바꾸는 일련의 DSL이며, lexer부터 codegen까지 Rust로 처음부터 구축합니다. 그 핵심 주변에는 비주얼 에디터, Python→`.xzz` 변환기(transpiler), LLM 효율성에 대한 응용 연구가 있습니다.

설계 방향은 모든 프로젝트에서 동일합니다: 오류를 런타임이 아닌 컴파일 타임으로 이동시키고, 무거운 의존성을 서브프로세스 경계 뒤에 격리해 CLI를 가볍게 유지하며, CSV에서 학습 완료 모델까지의 전체 경로를 단일 스크립트로 표현하는 것입니다.

---

## 생태계

```
[Python 코드] --> (py2xzz) ---\
                               --> [.xzz 스크립트] --> (Xazz 컴파일러) --> [실행 / Burn ML]
[비주얼 드래그앤드롭] (IDE) ----/
```

x1zzLang은 Xazz로 성장한 기반 프로젝트이며, py2xzz와 비주얼 IDE는 `.xzz` 스크립트를 Xazz 컴파일러로 공급합니다.

---

## 핀 프로젝트 (Pinned Projects)

### [Xazz](https://github.com/x1zzdev/Xazz) — AI 파이프라인 DSL

Polars 전처리, Burn 딥러닝 컴파일, 정적 보안 가드레일을 단일 `.xzz` 스크립트에 통합하는 Rust 기반 AI 파이프라인 DSL입니다.

- **기술 스택**: Rust (edition 2024), Polars, Burn, Axum / Tokio
- **주요 구현 특징**:
  - lexer → parser → 정적 타입 체커 → Rust/Polars/Burn codegen으로 이어지는 컴파일러 툴체인을 처음부터 구축
  - `Option<T>` 타입 시스템 기반 컴파일 타임 null/타입 안전성, `line:col` 진단 및 did-you-mean 제안 제공
  - pandas→NumPy→PyTorch 경계의 복사 비용 없이 Apache Arrow 버퍼를 직접 Burn에 전달하는 zero-copy 경로
  - PII/시크릿 탐지 기반 Policy-as-Code 가드레일, 세션별 엡실론 예산을 갖는 차등 프라이버시, SHA-256 append-only 감사 로그
  - `xazz-runner` 서브프로세스 경계 뒤에 무거운 엔진을 격리해 CLI 바이너리를 2–5MB로 유지하는 멀티 크레이트 워크스페이스

<p align="center">
  <img src="assets/ide_monitor.png" alt="Xazz IDE 모니터" width="600"/>
</p>

### [Xz](https://github.com/x1zzdev/Xz) — AI가 쓴 코드를 위한 언어

**AI-written, Human-reviewed**라는 하나의 명제를 중심으로 설계한 실험적 범용 프로그래밍 언어입니다. 기계가 생성한 코드를 사람이 읽고, 검증하고, 신뢰할 수 있도록 — 모든 동작을 명시적이고, 모든 계약을 가시적이며, 모든 실패 경로를 타입으로 만듭니다.

- **기술 스택**: Rust (inkwell / 이식형 LLVM 17), nom / pest, 생성된 Python `ctypes` 바인딩
- **주요 구현 특징**:
  - **인텐트 검증** — 핵심 차별점: 문서 주석의 `@intent` / `@requires` / `@ensures` / `@effects` 주장(claim)을 `pre` / `post` 계약과 순서대로 구조적으로 짝지어 본문과 대조 검증하고, `@trusted` 사람 검토 escape hatch를 제공 ("no unverified claims")
  - 처음부터 직접 구현한 전체 프런트엔드: lexer → 들여쓰기 파서 → 이름 해석 → 타입 추론 → 계약 검사 → 인텐트 검증
  - LLVM 백엔드: JIT 실행(`xz run`), 네이티브 바이너리(`xz build-native`), 공유 라이브러리(`xz build --shared` → `libXz.so` + `libXz.h`)
  - FFI 우선 상호운용: C ABI 브리지, `@cstruct` 레코드, `Ptr` 핸들, 생성된 Python `ctypes` 래퍼(`xz bind --lang python`)
  - 안정적인 에러 코드·스팬·신뢰도 점수 기반 수정안을 담은 구조적 JSON 진단 — LLM 자기수정 루프를 목표로 설계
  - 현재 상태: Phase 1–4 구현 완료, Phase 5(FFI) 진행 중, `hello.xz`·`contracts.xz` 실행 가능

### [next.xz](https://github.com/x1zzdev/next-xz) — Next.js × Xz 하이브리드 툴킷

**Next.js(TypeScript)** 와 **Xz 언어**를 연결하는 하이브리드 웹 프레임워크 겸 오케스트레이션 툴킷입니다 — 단독으로 시작한 프로젝트입니다. UI와 라우팅 셸은 Next.js가, 백엔드 비즈니스 로직은 Xz가 담당합니다. Xz가 모든 동작을 명시적으로 만들기 때문에, AI Agent가 타입 계약에 맞춰 백엔드 코드를 생성하고 `xz check-json`으로 스스로 교정하며, 여러 파일의 TS diff 대신 한눈에 검토 가능한 감사 결과를 사람에게 넘깁니다.

- **기술 스택**: TypeScript / Bun (`bun:ffi`) / Node (`koffi`), Next.js App Router, Xz (LLVM 17, C ABI)
- **주요 구현 특징**:
  - `@xz-lang/bridge`, `@xz-lang/agent`, `@xz-lang/audit` 세 패키지 모노레포
  - `@xz-lang/bridge` — `.xzint` 파서, Xz↔TS 타입 매핑, Bun FFI·Node koffi 로더, `Str` / `Bytes` / `@cstruct` 마샬링, `Result` → 타입드 에러 매핑, TypeScript 바인딩 제너레이터
  - `@xz-lang/agent` — 진단 파서를 갖춘 `xz check-json` 실행기, 프롬프트 빌더, KPI 계측(first-pass / self-correction / escalation)을 포함한 N=3 자가수정 루프
  - `@xz-lang/audit` — `/___audit` 라우트에서 Audit Card와 이펙트 배지를 렌더링해 사람이 승인하는 Next.js 개발 오버레이
  - 바인딩 레이어에 소유권을 모델링: borrow와 `transfer` 버퍼 수명을 구분해 `.xzint`부터 Server Action까지 zero-copy 전달을 명시적으로 유지

### [rails.xz](https://github.com/imrubydev/rails-xz) — Rails × Xz 하이브리드 툴킷

**Ruby on Rails** 와 **Xz 언어**를 연결하는 하이브리드 Rails 툴킷입니다 — Rails를 다루는 [**imrubydev**](https://github.com/imrubydev)와 함께 개발합니다. 웹 레이어(ActiveRecord, 라우팅, 컨트롤러, Hotwire)는 Rails가, 빠르고 격리되며 검증 가능해야 하는 백엔드 로직은 Xz가 담당합니다.

- **기술 스택**: Ruby / Rails (Engine, Hotwire, ViewComponent, Turbo), Ruby FFI (Fiddle), Xz (LLVM 17, C ABI)
- **역할 분담** (2인 협업):
  - **ax1s-x1zz** — `rails-xz-bridge`(FFI 브리지 + 바인딩 생성 + 컴파일러 통합)와 `rails-xz-agent`(자가수정 루프)
  - **imrubydev** — `rails-xz` Engine(`Xz::Module` 서비스 DSL, `/xz_audit` 대시보드)와 개발자 경험; 저장소 소유자
- **주요 구현 특징**:
  - 브리지가 `.xzint` 인터페이스에서 Ruby 바인딩을 생성하고 Xz 공유 라이브러리를 `Fiddle`/`ffi`로 로드합니다. `@export` `.xz` 모듈은 C ABI 라이브러리로 컴파일되므로 유지보수할 C 확장이 없습니다
  - 생성 로직은 `*.xz` 파일에 완전히 격리됩니다 — 선언된 effects와 파생된 effects가 다르면 컴파일 에러(`I0020`)이며, Agent가 Rails의 암묵적 컨텍스트를 건드리지 않습니다
  - Engine이 사람 검토 경로를 담당합니다: ViewComponent 감사 카드, 이펙트 배지, `AuditCard` 모델 기반 Turbo Stream 승인/거절

### [x1zzLang](https://github.com/x1zzdev/x1zzLang) — 데이터 파이프라인 언어
데이터 분석을 쉽게 접근하도록 만드는 Rust DSL로, `.xzz` 스크립트를 최적화된 Polars LazyFrame 실행 계획으로 컴파일합니다.

- **현재 스택**: Rust, Polars, clap, serde
- **주요 구현 특징**:
  - Rust로 구현한 전체 컴파일러 파이프라인 (lexer/parser/codegen/emitter)
  - `fillNull` 연산자를 포함한 null-안전 `Option<T>` 타입 시스템
  - `x1zz import`가 CSV 스키마를 자동 추론(EUC-KR/CP949 디코딩 포함)하고 타입 선언 생성
  - `x1zz emit rust`는 `.xzz`를 독립적인 Polars LazyFrame Rust 소스로 변환
  - CLI는 Polars를 절대 링크하지 않고, 실행은 서브프로세스로 위임하는 의존성 격리
  - 이후 Xazz로 발전한 기반 프로젝트

### [py2xzz](https://github.com/x1zzdev/py2xzz) — Python → `.xzz` 변환기
Pandas / PyTorch로 작성된 Python 데이터·딥러닝 파이프라인을 `.xzz` DSL 스크립트로 변환하는 Rust CLI입니다.

- **현재 스택**: Rust, serde
- **주요 구현 특징**:
  - Python 3 `ast` 모듈 스펙을 반영한 자체 lexer/parser로 AST 생성
  - Pandas 체인은 `PipelineOp` 체인으로, `nn.Module` 클래스는 `ModelDecl`/`LayerKind`로 변환하는 매퍼
  - CSV 헤더와 샘플값에서 열 타입을 추론하며, null이 있을 때 `Option<...>`로 감싸서 처리
  - 원본 Python 줄/열 위치를 생성된 코드로 추적하는 span map 기반 진단
  - 출력은 `xazz-core` AST와 1:1 대응하며 `xazz check` 통과

### [x1zzLang Visual IDE](https://github.com/x1zzdev/x1zzLang-visual-ide)
x1zzLang용 그래픽 파이프라인 편집기 — DAG 워크플로를 시각적으로 설계하고 `.xzz` 코드를 생성하여 네이티브로 실행합니다.

- **현재 스택**: React 18, Vite, @xyflow/react, i18next
- **주요 구현 특징**:
  - 9개의 내장 파이프라인 연산자를 갖춘 드래그 앤 드롭 DAG 빌더
  - 전용 transpiler 엔진을 통한 시각적 그래프 → `.xzz` 소스 실시간 변환
  - 백엔드 대상 원클릭 실행 및 테이블 결과 표시
  - 멀티 워크플로 탭, 실행 취소/재실행, 자동 저장, 컨테이너 그룹, 한/영 UI 지원

### [LLM PCAG 연구](https://github.com/ax1s-x1zz/llm-pcag-research) — LLM 양자화의 파워 월
LLM 가중치 양자화가 실제로 얼마나 에너지를 절감하는지, 그리고 그로 인한 그리드 Jevons 역설을 계량화하는 연구입니다.

- **현재 스택**: Python (NumPy, pandas, SciPy, SymPy, Matplotlib)
- **주요 구현 특징**:
  - 양자화 효율이 정확도 손실보다 빠르게 붕괴하는 지점을 측정하는 PCAG 메트릭(출력 이득 당 전력 비용) 정의
  - 세 경로(empirical PCHIP, Monte Carlo, 해석 모델)로 INT4→INT3 'Power Wall' 검증
  - 변곡점 루트가 진폭에 독립적임을 증명
  - 수요 탄력성 E_d > 1일 때만 그리드 부하가 증가함을 SymPy로 상징적으로 증명, Jevons 역설 폐쇄형 정식화
  - 데이터 소스 라벨링(문헌 기반 vs GPU 측정)을 엄격하게 지키는 재현 가능한 실험 파이프라인

<p align="center">
  <img src="assets/fig15_dashboard.png" alt="LLM PCAG 대시보드" width="600"/>
</p>

---
### 오픈소스 기여 (Open Source Contributions)

**[tracel-ai/burn](https://github.com/tracel-ai/burn)**과 **[apache/arrow-rs](https://github.com/apache/arrow-rs)**에 업스트림 기여 — 머지된 PR 6개. 그중 하나는 리뷰 과정에서 공통 레이어 수정으로 재작업.

**모든 백엔드에 `Min` / `Max` `scatter` / `select_assign` 구현** — [머지된 PR #5582](https://github.com/tracel-ai/burn/pull/5582)

- [#5522](https://github.com/tracel-ai/burn/issues/5522)로 남겨진 요소별 `scatter` / `select_assign` API의 마지막 공백을 채웠습니다. `Assign`, `Add`, `Mul`은 지원됐지만 `Min` / `Max`는 모든 백엔드에서 `unimplemented!`에 걸렸습니다 (`scatter_nd`는 이미 다섯 변형을 모두 구현했는데도). 빠진 변형을 end-to-end로 구현했습니다.
- **네 백엔드 전부**: ndarray 프리미티브, 기존 업데이트 워커를 공유하는 `flex` 헬퍼, `BinaryMinOp` / `BinaryMaxOp` 커널을 재사용하는 `cubecl` 항목, `scatter_reduce` / `index_reduce_`(`"amin"` / `"amax"`)를 쓰는 `tch`.
- **Autodiff**: `Min` / `Max` `scatter`와 `select_assign`의 backward 패스를 `scatter_nd` Min/Max 그래디언트와 동일하게 구현 — 비교 기반 winner 마스크, 동률은 양쪽 입력에 귀속, unique index 요구.
- 이제 dispatch match가 모든 `IndexingUpdateOp` 변형을 열거하므로, 지원되지 않는 조합은 런타임 panic 대신 컴파일 타임에 실패합니다.
- `--features ndarray`로 텐서 1839개 + autodiff 572개 테스트 통과, `burn-ndarray`·`burn-flex`·`burn-autodiff`·`burn-cubecl` clippy 클린. backward를 제거하면 새 autodiff 테스트가 기존 `unimplemented!`로 실패함도 확인했습니다.

<details>
<summary><b>그 외 머지된 기여 (5)</b></summary>

- **[burn #5555](https://github.com/tracel-ai/burn/pull/5555)** — `TensorCheck::matmul`에 배치 차원 브로드캐스트 검증을 추가해, 디스패치 이전에 모든 백엔드가 일관된 `Tensor Operation Error`를 받도록. (메인테이너 방향에 따라 #5542를 공통 `TensorCheck` 레이어로 재작업.)
- **[burn #5564](https://github.com/tracel-ai/burn/pull/5564)** — `remainder`, `powi`, `powf`, `hypot`, `atan2`에 `TensorCheck`(`binary_ops_ew`) 적용, 각각 회귀 테스트 추가.
- **[burn #5580](https://github.com/tracel-ai/burn/pull/5580)** — `TensorCheck::matmul`에 rank 검증 추가; rank < 2가 백엔드별 panic이나 불일치 결과 대신 명확한 에러를 반환.
- **[burn #5639](https://github.com/tracel-ai/burn/pull/5639)** — `burn-std`의 no-std 빌드 실패 수정: `layout.rs`의 `#[cfg(test)]` 모듈이 `vec!` 매크로를 임포트하지 않고 `vec![...]`를 써서 `cargo test --no-default-features -p burn-std`가 컴파일되지 않던 문제. `use alloc::vec;` 추가.
- **[arrow-rs #11005](https://github.com/apache/arrow-rs/pull/11005)** — IPC run-ends 재인코딩에서 `BufferBuilder`를 `Vec`으로 교체(에픽 [#10245](https://github.com/apache/arrow-rs/issues/10245)); 메인테이너 2명 승인 후 머지. 후속 interval 파싱 PR [#11006](https://github.com/apache/arrow-rs/pull/11006)은 리뷰 중.

</details>

---
### 대외 활동 및 리더십 (Activities & Leadership)

#### **Trendsetter — 창립자 및 리드 아키텍트 (2026.03 – 현재)**
> **역할:** 동아리 창설자, 파이프라인 및 커리큘럼 설계자, 플랫폼 엔지니어

- **동아리 설계 및 온보딩 프레임워크 구축**
  - 반도체 및 최신 기술 중심의 데이터 분석 동아리 `Trendsetter`를 직접 창설하고, 부원들의 파이썬 및 데이터 분석 진입장벽을 낮추기 위한 단계별 탐구 로드맵 설계.
  - 1차 역량 진단 제출 양식, 마크다운 작성 가이드, 파이썬(Pandas, Matplotlib, Plotly) 가이드라인을 직접 작성하여 배포.

- **실습 환경 및 교육용 스타터 키트(Starter-Kit) 엔지니어링**
  - **CPU vs. GPU 스펙 비교 분석**: Google Colab 노트북, 엄선된 하드웨어 데이터셋, 과제 제출용 구글 폼이 포함된 오픈소스 GitHub 리포지토리([ax1s-x1zz/trendsetter-semiconductor-01-cpu-vs-gpu](https://github.com/ax1s-x1zz/trendsetter-semiconductor-01-cpu-vs-gpu)) 구축.
  - **무어의 법칙 & 황의 법칙 검증**: 반도체 트랜지스터 집적도 데이터를 로그 스케일($\log$) 변환 및 선형 회귀 분석(Linear Regression)으로 수학적·통계적 검증을 수행할 수 있는 Colab 실습 환경 제작.
  - **산업 데이터 분석**: 산업통상자원부(MOTIE) 및 통계청(KOSIS)의 공식 반도체 수출액 데이터 기반 데이터 전처리 템플릿 배포.

#### **코드게이트 AI 스타트업 해커톤 (`Xazz / x1zz Guard`) (2026.07)**
> **역할:** 팀 리더, 프로젝트 총괄, 코어 툴체인 아키텍트
- **팀 & IP 거버넌스:** 상금 1/N 정산을 명확히 하고, 기존 개인 자산(`x1zzLang`)과 대회 신규 산출물 간의 IP 귀속 범위를 정립.
- **위기 대응:** 팀원 이탈 상황에서 역할을 신속히 재조정하여 마감 기한 내 완성도 높은 프로젝트 제출 성공.
- **기술 방어 논리:** 대량 쿼리 오버헤드 및 차등 프라이버시(DP) 성능 저하 등 심사위원이 예상하는 질의에 대한 구체적 대응책 마련.
- **보안 아키텍처:** Rust 및 온프레미스 sLM(Qwen2.5-Coder) 기반의 AST 정적 보안 통제 계층 `x1zz Guard` 설계.

#### **GEEKs Hackathon (2026.08.04 - 2026.08.05)**
> **역할:** 팀 기획자 및 발표자, 전략적 피봇 주도, 제품/UX 검증
- **전략적 피봇:** 제한된 개발 기간과 팀 기술 시너지를 고려해 복잡한 B2B PaaS에서 실현 가능한 B2C 민원 플랫폼('우선해줘')으로 전환.
- **로직 & UX 기획:** Vision AI(Claude Opus 5)와 PostGIS 공공데이터를 결합한 행정 정당성 스코어링 논리 기획.
- **발표 전담:** 기획 전환 후 발표 준비부터 실전 8분 피칭까지 전담하여 프로젝트 완성 및 공유 완료.
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
