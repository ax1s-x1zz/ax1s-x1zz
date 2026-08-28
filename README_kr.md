# ax1s-x1zz

**한국 고등학교 2학년 학생입니다. 데이터 파이프라인 DSL, 컴파일러, AI/ML 도구를 Rust로 개발하고 있습니다.**

[English Document](./README.md)

---

## 소개

데이터 파이프라인 언어와 그 주변 도구를 직접 설계하고 구현합니다. 주요 작업은 `.xzz` 스크립트를 최적화된 Polars 실행 계획으로 컴파일하는 일련의 DSL이며, 컴파일러는 Rust로 처음부터 작성합니다. 언어 자체와 함께 비주얼 에디터, 소스 간 변환기(transpiler), LLM 효율성에 대한 응용 연구를 병행합니다.

핵심 설계 방향은 다음과 같습니다: 오류를 런타임이 아닌 컴파일 타임으로 이동시키고, CLI는 서브프로세스 경계 뒤에 무거운 의존성을 격리하여 가볍게 유지하며, CSV에서 학습 완료 모델까지의 전체 경로를 단일 스크립트로 표현하는 것입니다.

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

### [x1zzLang](https://github.com/x1zzdev/x1zzLang) — 데이터 파이프라인 언어
접근 가능한 데이터 분석을 탐구하는 Rust 기반 DSL로, `.xzz` 스크립트를 최적화된 Polars LazyFrame 실행 계획으로 컴파일합니다.

- **현재 스택**: Rust, Polars, clap, serde
- **주요 구현 특징**:
  - Rust로 구현한 전체 컴파일러 파이프라인 (lexer/parser/codegen/emitter)
  - `fillNull` 연산자를 포함한 null-안전 `Option<T>` 타입 시스템
  - `x1zz import`가 CSV 스키마를 자동 추론(EUC-KR/CP949 디코딩 포함)하고 타입 선언 생성
  - `x1zz emit rust`는 `.xzz`를 독립적인 Polars LazyFrame Rust 소스로 변환
  - CLI는 Polars를 절대 링크하지 않고, 실행은 서브프로세스로 위임하는 의존성 격리
  - 이후 Xazz로 발전한 기반 프로젝트

### [py2xzz](https://github.com/x1zzdev/py2xzz) — Python → `.xzz` 변환기
Pandas / PyTorch로 작성된 Python 데이터·딥러닝 파이프라인을 `.xzz` DSL 스크립트로 변환하는 Rust CLI 도구입니다.

- **현재 스택**: Rust, serde
- **주요 구현 특징**:
  - Python 3 `ast` 모듈 스펙을 반영한 자체 lexer/parser로 AST 생성
  - Pandas 체인은 `PipelineOp` 체인으로, `nn.Module` 클래스는 `ModelDecl`/`LayerKind`로 변환하는 매퍼
  - CSV 헤더와 샘플값에서 열 타입을 추론하며, null이 있을 때 `Option<...>`로 감싸서 처리
  - 원본 Python 줄/열 위치를 생성된 코드로 추적하는 span map 기반 진단
  - 출력은 `xazz-core` AST와 1:1 대응하며 `xazz check` 통과

### [x1zzLang Visual IDE](https://github.com/x1zzdev/x1zzLang-visual-ide)
x1zzLang용 그래픽 파이프라인 편집기로, DAG 워크플로를 설계하고 `.xzz` 코드를 생성하여 네이티브로 실행합니다.

- **현재 스택**: React 18, Vite, @xyflow/react, i18next
- **주요 구현 특징**:
  - 9개의 내장 파이프라인 연산자를 갖춘 드래그 앤 드롭 DAG 빌더
  - 전용 transpiler 엔진을 통한 시각적 그래프 → `.xzz` 소스 실시간 변환
  - 백엔드 대상 원클릭 실행 및 테이블 결과 표시
  - 멀티 워크플로 탭, 실행 취소/재실행, 자동 저장, 컨테이너 그룹, 한/영 UI 지원

### [LLM PCAG 연구](https://github.com/ax1s-x1zz/llm-pcag-research) — LLM 양자화의 파워 월
LLM 가중치 양자화의 에너지 절감 효율과 그리드 Jevons 역설을 계량화하는 연구입니다.

- **현재 스택**: Python (NumPy, pandas, SciPy, SymPy, Matplotlib)
- **주요 구현 특징**:
  - 양자화 효율이 정확도 손실보다 빠르게 붕괴하는 지점을 측정하는 PCAG 메트릭(출력 이득 당 전력 비용) 정의
  - 세 경로(empirical PCHIP, Monte Carlo, 해석 모델)로 INT4→INT3 'Power Wall' 검증
  - 변곡점 루트가 진폭에 독립적임을 증명
  - 수요 탄력성 E_d > 1일 때만 그리드 부하가 증가함을 SymPy로 상징적으로 증명, Jevons 역설 폐쇄형 정식화
  - 데이터 소스 라벨링(문헌 기반 vs GPU 측정)을 엄격하게 지키는 재현 가능한 실험 파이프라인

---

## 기술 스택 요약

| 영역 | 기술 |
|---|---|
| 시스템 / DSL | Rust (edition 2024), Cargo workspace, clap, serde |
| 데이터 엔진 | Polars (LazyFrame), Apache Arrow |
| 딥러닝 | Burn, zero-copy 텐서 전달 |
| 웹 / API | Axum, Tokio, React 18, Vite, @xyflow/react |
| 백엔드 통합 | Rust REST API, SHA-256 감사 로깅 |
| 연구 / 분석 | Python, NumPy, pandas, SciPy, SymPy, Matplotlib |

---

## 소개

이 프로젝트들은 타입 안전하고 컴파일되는 데이터 파이프라인을 향한 지속적인 탐구의 일부입니다. 컴파일러 설계, 데이터 도구, ML 인프라스트럭처에서 비슷한 문제를 다루고 있다면 언제든 연락주세요.