# ax1s-x1zz

**한국 고등학교 2학년 학생입니다. 데이터 파이프라인 DSL, 컴파일러, AI/ML 도구를 Rust로 개발하고 있습니다.**

[English Document](./README.md)

---

## 소개

**Rust로 타입 안전하고 컴파일되는 데이터 파이프라인 DSL과 고성능 ML 인프라를 구축합니다.**

데이터 파이프라인 언어와 그 주변 도구를 직접 설계하고 구현합니다. 주요 작업은 `.xzz` 스크립트를 최적화된 Polars 실행 계획으로 컴파일하는 일련의 DSL이며, 컴파일러는 Rust로 처음부터 작성합니다. 언어 자체와 함께 비주얼 에디터, 소스 간 변환기(transpiler), LLM 효율성에 대한 응용 연구를 병행합니다.

핵심 설계 방향은 다음과 같습니다: 오류를 런타임이 아닌 컴파일 타임으로 이동시키고, CLI는 서브프로세스 경계 뒤에 무거운 의존성을 격리하여 가볍게 유지하며, CSV에서 학습 완료 모델까지의 전체 경로를 단일 스크립트로 표현하는 것입니다.

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
### 대외 활동 및 리더십 (Activities & Leadership)

#### **코드게이트 AI 스타트업 해커톤 (`Xazz / x1zz Guard`) (2026.07)**
> **역할:** 팀 리더, 프로젝트 총괄, 코어 툴체인 아키텍트
- **투명한 권한 정의 및 팀 빌딩 리더십:** 팀 결성 직후 상금 동일 분배(1/N) 원칙을 명시하고, 기존 개인 자산(`x1zzLang` 코어)과 대회 중 신규 산출물(가드레일 규칙, 발표자료)의 지적재산권(IP) 귀속 범위를 명확히 정리하여 팀원 간 신뢰와 주도적 참여 환경을 조성했습니다.
- **위기 관리 및 스코프 재설계:** 팀원 이탈이라는 돌발 상황 발생 시, 팀원들의 전문성에 맞춰 '아키텍처/시나리오 고도화'와 '2페이지 요약본/BM 서사 정제'로 역할을 즉시 재배치하고 세부 플랜을 제시하여 마감 기한 내 완성도 높게 프로젝트를 완수했습니다.
- **선제적 기술 방어 논리 수립:** 대량 쿼리 연산 오버헤드 및 차등 프라이버시(DP) 노이즈로 인한 모델 성능 저하 등 심사위원의 예상 태클 포인트를 미리 도출하고, '윈도우 기반 프라이버시 필터' 및 '적응형 노이즈 주입' 기술 대응책을 기획서에 반영했습니다.
- **독자적 DSL 및 보안 통제 계층 설계:** Rust 기반 격리형 AI DSL인 **`x1zzLang`**의 전체 툴체인을 설계 및 구현했습니다. Polars LazyFrame 가속, Burn 딥러닝 연동과 함께, 실행 전 AST 정적 분석 및 온프레미스 sLM(Qwen2.5-Coder) 기반 코드 자동 보정 파이프라인인 `x1zz Guard` 코어를 구축했습니다.

#### **GEEKs Hackathon (2026.08.04 - 2026.08.05)**
> **역할:** 팀 발표자, 전략적 피봇 주도, 제품/UX 기획 및 검증
- **기술적 현실 판단 및 전략적 피봇 주도:** '지속가능한 인프라(SDG 9)' 주제의 무박 2일 해커톤에서, 본인이 구상한 B2B PaaS 아이템이 팀원들의 기술 스택과 단기 해커톤 특성에 맞지 않음을 빠르게 파악했습니다. 이에 팀 전원이 몰입할 수 있는 B2C 민원 행정 플랫폼('우선해줘')으로의 신속한 피봇팅을 이끌었습니다.
- **사용자 경험(UX) 중심 검증 및 팀 목표 일치:** 생소한 B2C 도메인 및 미경험 타겟층을 대상으로 팀원들과 사용자 경험 중심 아이디어를 검증했습니다. 비전 AI(Claude Opus 5)와 PostGIS/공공데이터를 결합한 행정 정당성 스코어링 논리 기획을 주도하여 팀의 역량을 합쳤습니다.
- **주도적 역할 재정의 및 최종 발표 전담:** 피봇된 기획 특성상 개인 개발 지분율이 낮아지자, 발표 준비 및 실전 피칭을 전담하는 역할로 신속히 전환하여 8분간의 최종 발표를 무사히 마쳤습니다.

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

- **이메일**: [ax1s@x1zz.com](mailto:ax1s@x1zz.com)
