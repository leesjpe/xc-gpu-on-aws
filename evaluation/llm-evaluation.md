# LLM 평가 방법론 가이드

> 셀프호스팅 LLM 도입을 위한 모델 평가 프레임워크
> 용도에 맞는 모델을 고르려면 체계적 평가가 필요합니다.

---

## 1. Artificial Analysis Coding Index — 정확히 무엇을 측정하는가

AA Coding Index를 이해 합니다.

출처: [AA Intelligence Benchmarking Methodology](https://artificialanalysis.ai/methodology/intelligence-benchmarking/)

### 1.1 AA Intelligence Index v4.0 전체 구조

AA Intelligence Index는 4개 카테고리 × 25% 균등 가중으로 구성됩니다.
"Coding Index"는 이 중 Coding 카테고리(25%)만 따로 뽑은 것입니다.

```
Intelligence Index v4.0 (총점 0~100)
├── Agents (25%)
│   ├── GDPval-AA (16.7%) — 44개 직업군 실무 태스크, ELO 기반
│   └── 𝜏²-Bench Telecom (8.3%) — 에이전트-사용자 대화 시뮬레이션
│
├── Coding (25%)  ← ★ AA 의 "Coding Index"를 구성하는 카테고리
│   ├── Terminal-Bench Hard (16.7%) — 터미널 기반 에이전트 태스크 44개
│   └── SciCode (8.3%) — Python 코드 생성 능력 평가 (과학 도메인 문제 288개)
│
├── General (25%)
│   ├── AA-Omniscience (12.5%) — 지식 + 환각률 측정 6,000문제
│   ├── AA-LCR (6.25%) — 100K 토큰 긴 문서 추론
│   └── IFBench (6.25%) — 지시 따르기 정확도
│
└── Scientific Reasoning (25%)
    ├── HLE (12.5%) — 대학원급 2,158문제
    ├── GPQA Diamond (6.25%) — 과학 추론 198문제
    └── CritPt (6.25%) — 연구 수준 물리 70문제
```

### 1.2 Coding Index의 실체: Terminal-Bench Hard + SciCode

"Coding Index"는 AA가 별도로 제공하는 코딩 능력 지표입니다.
AA의 [Coding Index 페이지](https://artificialanalysis.ai/models/capabilities/coding)에 따르면
이 지표는 Terminal-Bench Hard와 SciCode, 2개 벤치마크로 구성됩니다.

참고로 이것은 Intelligence Index(10개 벤치마크 종합)와는 별개의 지표입니다.
Intelligence Index 내에서 Coding 카테고리는 전체의 25%를 차지하며,
그 안에서 Terminal-Bench Hard가 16.7%, SciCode가 8.3%의 가중치를 갖습니다.
Coding Index 자체의 내부 가중치는 AA가 별도로 공개하지 않았습니다.

#### Terminal-Bench Hard (Intelligence Index 내 Coding 비중의 2/3)

| 항목 | 내용 |
|------|------|
| 무엇인가 | 터미널(CLI) 환경에서 복잡한 태스크를 자율 수행하는 에이전트 능력 평가 |
| 태스크 수 | 44개 (hard subset) |
| 반복 | 3회 반복, pass@1 |
| 제한 | 최대 100 에피소드, 24시간 타임아웃, 100만 토큰 입력 제한 |
| 에이전트 하네스 | Terminus 2 (AA가 모든 모델에 동일 적용) |

실제 태스크 예시 (44개 중 일부):
```
- cobol-modernization          ← COBOL 코드 현대화
- make-doom-for-mips           ← MIPS용 DOOM 빌드
- make-mips-interpreter        ← MIPS 인터프리터 구현
- swe-bench-astropy-1/2        ← SWE-bench 스타일 버그 수정
- gpt2-codegolf               ← GPT-2 코드골프
- install-windows-xp           ← Windows XP 설치
- play-zork / play-zork-easy   ← 텍스트 어드벤처 게임 플레이
- reverse-engineering          ← 리버스 엔지니어링
- password-recovery            ← 패스워드 복구
- train-fasttext               ← FastText 모델 학습
- word2vec-from-scratch        ← Word2Vec 처음부터 구현
- path-tracing / path-tracing-reverse ← 레이트레이싱
- parallel-particle-simulator  ← 병렬 입자 시뮬레이션
- configure-git-webserver      ← Git 웹서버 설정
- write-compressor             ← 압축기 작성
- cartpole-rl-training         ← RL 학습
- feal-differential/linear-cryptanalysis ← 암호 분석
```

#### SciCode (Intelligence Index 내 Coding 비중의 1/3)

| 항목 | 내용 |
|------|------|
| 무엇인가 | Python 코드 생성 능력 평가 (문제 소재가 과학 도메인) |
| 문제 수 | 288개 서브프로블럼 (테스트셋), 80개 메인 문제에서 분해 |
| 반복 | 3회 반복, pass@1 |
| 평가 방식 | 생성된 Python 코드가 유닛 테스트를 통과하는지 자동 채점 |
| 도구 사용 | 없음 (순수 코드 생성) |

SciCode가 측정하는 코딩 능력:
```
- 함수 시그니처를 보고 올바른 Python 구현을 작성하는 능력
- numpy, scipy, sympy 등 라이브러리 활용 코딩
- 서브프로블럼을 단계적으로 풀어가는 코드 구조화
- 유닛 테스트를 통과하는 정확한 코드 생성 (pass@1)

※ 문제의 소재가 물리/화학/수학일 뿐, 측정 대상은 "코딩 능력"
   AA도 이를 "Code Generation" 필드로 분류하고 Coding 카테고리에 배치
```

### 1.3 핵심 분석: AA Coding Index가 측정하는 것과 측정하지 않는 것

```
AA Coding Index가 측정하는 것:
  ✓ 터미널 환경에서 복잡한 태스크를 자율 수행하는 능력 (Terminal-Bench)
  ✓ Python 코드 생성 능력 — 함수 구현, 라이브러리 활용, 유닛 테스트 통과 (SciCode)
  ✓ 에이전트로서의 계획-실행-검증 루프 (Terminal-Bench)
  ✓ 복잡한 요구사항을 코드로 구조화하는 능력 (SciCode)

AA Coding Index가 측정하지 않는 것:
  ✗ 일반적인 소프트웨어 개발 코드 생성 (HumanEval, MBPP 스타일)
  ✗ 실제 GitHub 이슈 기반 버그 수정 (SWE-bench)
  ✗ 경쟁 프로그래밍 / 알고리즘 (LiveCodeBench) — v4.0에서 제외됨
  ✗ 멀티턴 대화 기반 코드 편집 (Aider Polyglot)
  ✗ Python 외 언어 (Java, TypeScript, Go 등)
  ✗ 코드 리뷰, 리팩토링, 테스트 작성
  ✗ IDE 자동완성 / 인라인 코드 제안
  ✗ 한국어 프롬프트 기반 코딩
```

---

## 2. AA Coding Index가 높으면 그냥 도입해도 되는가?

결론부터 말하면: **아닙니다.** 세 가지 이유가 있습니다.

```
핵심 논지 1: AA가 측정하는 "코딩 능력"과 우리가 필요한 "코딩 능력"은 같은 코딩이 아니다
핵심 논지 2: AA는 풀 정밀도로 평가하지만, 실제 환경에서는 양자화해서 서빙하는 경우가 있음 — 점수가 유지되는지는 자체 평가 전에는 알 수 없음
핵심 논지 3: Coding Index 몇 점 차이가 GPU 비용 수배~수십배 차이를 정당화하는지 검증이 필요하다
```

이 세 가지는 AA Coding Index 자체의 문제가 아닙니다.
AA는 훌륭한 독립 평가 기관이고, Coding Index도 코딩 능력을 측정하는 것이 맞습니다.
문제는 **"AA Coding Index가 높으면 우리 환경에서도 좋을 것이다"라는 가정이 검증되지 않았다**는 것입니다.

### 2.1 핵심 논지 1: 같은 "코딩"이 아니다

Terminal-Bench Hard의 44개 태스크는 "COBOL 현대화", "DOOM을 MIPS로 빌드", "Windows XP 설치" 같은
극단적으로 어려운 시스템 태스크입니다. SciCode는 함수 시그니처 기반 Python 구현으로 코딩 능력을 측정합니다.
둘 다 코딩 능력을 측정하는 건 맞습니다.

하지만 우리가 실제로 하려는 건 "내부 개발자들이 일상적으로 쓰는 코드 어시스턴트"입니다.

```
AA Coding Index가 측정하는 코딩:
  → "MIPS 인터프리터를 처음부터 구현해라" (Terminal-Bench)
  → "이 함수 시그니처에 맞는 Python 구현을 작성해라" (SciCode)

우리에게 필요한 코딩 예시:
  → "이 Spring Boot API에 페이징 추가해줘"
  → "이 React 컴포넌트 리팩토링해줘"
  → "이 PR 코드 리뷰해줘"
  → "이 SQL 쿼리 왜 느린지 분석해줘"
```

AA Coding Index에서 높은 점수를 받는 모델이 위 태스크도 잘할 **가능성은 높습니다**.
코딩 능력의 기반이 되는 역량(코드 이해, 생성, 디버깅)은 공유되니까요.
하지만 **보장되지는 않습니다**. 이것이 내부 평가가 필요한 첫 번째 이유입니다.

#### 언어 커버리지

SciCode는 Python만 평가합니다. Terminal-Bench도 주로 Python/Bash 환경입니다.
팀이 Java, TypeScript, Go, Kotlin 등을 사용한다면 이 지표는 해당 언어 능력을 전혀 반영하지 않습니다.

#### 평가 규모

Coding 카테고리 전체가 Terminal-Bench 44태스크 + SciCode 288서브프로블럼입니다.
SWE-bench Verified(500개 실제 GitHub 이슈)나 LiveCodeBench(315개 경쟁 프로그래밍)에 비해
태스크 다양성과 규모가 제한적입니다.

#### v4.0에서 제외된 벤치마크들

AA는 v4.0에서 MMLU-Pro, LiveCodeBench, AIME 2025를 Intelligence Index에서 제외했습니다.
이유는 "상위 모델들이 포화(saturation)되어 변별력이 없어졌기 때문"입니다.

하지만 이것은 **프론티어 모델 간 변별력** 관점이지, 실무 코딩 능력과 무관합니다.
LiveCodeBench가 빠졌다고 해서 알고리즘 능력이 중요하지 않은 것이 아닙니다.

### 2.2 핵심 논지 2: 평가 환경과 서빙 환경이 다르다

AA Coding Index는 모델 제공사의 API, 즉 FP16/BF16 풀 정밀도로 평가합니다.
하지만 실제 vLLM에서 FP8 또는 INT4로 양자화하여 서빙해야 하는 경우 있습니다.

```
AA 평가 환경:  FP16/BF16 풀 정밀도 → Coding Index 47
실제 서빙 환경: FP8 양자화          → ???
              INT4 양자화          → ???
```

양자화에 의한 품질 저하는 모델마다 다릅니다.
GLM-5.1이 FP8에서 품질을 잘 유지하는지,
Qwen 3.5-397B가 INT4에서 오히려 더 잘 버티는지는
**직접 양자화해서 돌려봐야 알 수 있습니다.**

AA Coding Index 47점은 "풀 정밀도에서의 47점"이며,
"실제 vLLM FP8 환경에서의 47점"이 아닙니다.

### 2.3 핵심 논지 3: 점수 차이가 비용 차이를 정당화하는가

리스트에서 가장 중요한 비교:

```
GLM-5.1 (47점) → 8× H100 80GB 필요 → 클라우드 기준 ~$16/hr
Gemma 4 31B (39점 추정) → 1× A100 80GB 가능 → 클라우드 기준 ~$1.5/hr
```

Coding Index 8점 차이에 GPU 비용이 10배 이상 차이납니다.

이 8점이 실제 내부 태스크에서도 체감되는 차이인지,
아니면 "DOOM을 MIPS로 빌드"하는 능력의 차이일 뿐인지는
**내부 평가 없이는 판단할 수 없습니다.**

참고로 GLM-5.1과 Qwen 3.5-397B의 차이는 4점입니다.
AA는 95% 신뢰구간을 ±1%로 추정하지만, 이는 Intelligence Index 전체 기준이고
개별 벤치마크는 더 넓은 신뢰구간을 가질 수 있다고 명시합니다.
Terminal-Bench Hard는 44개 태스크 × 3회 반복 = 132회 시행이므로,
이 규모에서 4점 차이가 통계적으로 유의미한지도 불확실합니다.

---

## 3. 내부 평가가 필요한 이론적 근거

### 3.1 벤치마크 일반론

위 세 가지 핵심 논지는 LLM 평가 분야에서 학술적으로 잘 알려진 문제들에 기반합니다:

| 문제 | 설명 | 위 논지와의 관계 |
|------|------|-----------------|
| Distribution Shift | 벤치마크 데이터 분포 ≠ 실제 사용 데이터 분포 | **논지 1** — AA 태스크 분포 ≠ 실제 태스크 분포 |
| Evaluation Context Gap | 평가 환경 ≠ 배포 환경 | **논지 2** — 풀 정밀도 평가 ≠ 양자화 서빙 |
| Goodhart's Law | "측정 지표가 목표가 되면, 좋은 지표가 아니게 된다" | **논지 3** — Coding Index 점수 자체가 목적이 되면 안 됨 |
| Task Specificity | 벤치마크는 특정 태스크 유형만 측정 | AA는 44+288개 특정 태스크만 커버 |
| Data Contamination | 학습 데이터에 벤치마크 문제 포함 가능 | 점수가 실제 능력을 과대 추정할 수 있음 |

핵심은 이것입니다:
**벤치마크는 "이 모델이 이 태스크를 잘한다"를 말해줄 뿐,
"이 모델이 우리 환경에서 우리 태스크를 잘할 것이다"를 보장하지 않습니다.**
| Data Contamination | 학습 데이터에 벤치마크 문제 포함 가능 | 점수가 실제 능력을 과대 추정 |
| Evaluation Bias | 평가 방법 자체의 편향 | HLE는 특정 모델로 adversarial 선별 → 해당 모델에 불리 |

### 3.2 세 가지 논지를 실제 상황에 적용하면

```
이유 1: 태스크 분포의 차이
  AA Coding Index → "DOOM을 MIPS로 빌드", "함수 시그니처 기반 Python 구현"
  실제 내부 사용  → "이 Spring Boot API에 페이징 추가해줘",
                    "이 React 컴포넌트 리팩토링해줘",
                    "이 SQL 쿼리 최적화해줘"

이유 2: 언어/프레임워크 특수성
  AA Coding Index → Python + Bash 중심
  실제 내부 사용  → 팀이 쓰는 특정 언어, 프레임워크, 라이브러리

이유 3: 컨텍스트 패턴의 차이
  AA Coding Index → 제로샷, 단일 태스크
  실제 내부 사용  → 긴 코드 컨텍스트, 멀티턴 대화, 프로젝트 구조 이해

이유 4: 품질 기준의 차이
  AA Coding Index → "테스트 통과 여부" (pass/fail)
  실제 내부 사용  → 코드 스타일, 가독성, 팀 컨벤션, 보안, 성능

이유 5: 서빙 환경의 차이
  AA Coding Index → API 호출 (지연 시간 무관)
  실제 내부 사용  → vLLM 셀프호스팅, 양자화, GPU 제약 하에서의 품질
```

### 3.3 정리: 내부 평가 없이 도입하면 생기는 리스크

```
리스크 1: AA Coding Index 1위 모델을 도입했는데,
         정작 팀이 쓰는 Java/TypeScript 태스크에서는 2위 모델보다 못할 수 있다.

리스크 2: 풀 정밀도에서 47점이던 모델이 FP8 양자화 후 실질적으로 43점 수준으로 떨어져,
         더 작고 저렴한 모델과 차이가 없어질 수 있다.

리스크 3: 8× H100을 투자했는데, 1× A100으로 돌아가는 모델과
         실제 개발자 만족도에서 유의미한 차이가 없을 수 있다.
```

이 리스크들은 내부 평가를 통해서만 검증할 수 있습니다.
이것이 이 문서가 존재하는 이유입니다.

---

## 4. 권장 평가 프레임워크: AA + 내부 평가 병행

### 4.1 AA Coding Index의 올바른 활용법

AA Coding Index를 버리라는 것이 아닙니다. 올바른 위치에서 활용해야 합니다.

```
┌─────────────────────────────────────────────────────────┐
│  Phase 1: 스크리닝 (AA Coding Index 활용)                  │
│                                                         │
│  AA Coding Index + Intelligence Index로                 │
│  후보 모델 3~5개를 빠르게 선정                                │
│                                                         │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 2: 보완 벤치마크 (AA가 커버하지 못하는 영역)              │
│                                                         │
│  ├─ SWE-bench Verified/Pro → 실제 버그 수정 능력            │
│  ├─ LiveCodeBench → 알고리즘 능력 (v4.0에서 빠짐)            │
│  ├─ Aider Polyglot → 멀티턴 코드 편집                      │
│  └─ MultiPL-E → Python 외 언어 커버리지                     │
│                                                         │
│  이 벤치마크들은 AA가 "포화"를 이유로 뺐지만,                    │
│  실무 코딩 능력 평가에는 여전히 유효함                           │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 3: 내부 태스크 평가 (가장 중요)                        │
│                                                         │
│  실제 내부 코드베이스 + 실제 사용 시나리오로 평가                  │
│                                                         │
│  테스트 셋 구성 (20~50개):                                  │
│  ├─ 코드 생성: 팀이 실제로 작성하는 유형의 코드                   │
│  ├─ 버그 수정: 과거 실제 버그 티켓 기반                         │
│  ├─ 코드 리뷰: 실제 PR에 대한 리뷰 요청                        │
│  ├─ 리팩토링: 레거시 코드 개선                                │
│  └─ 팀 언어/프레임워크 특화 태스크                             │
│                                                         │
│  ⚠️ 반드시 양자화된 상태(FP8/INT4)에서 평가                     │
│     (AA는 풀 정밀도로 평가하므로 결과가 다를 수 있음)              │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 4: 서빙 성능 + 사용자 파일럿                           │
│                                                         │
│  ├─ vLLM 위에서 TTFT, TPS, Throughput 측정                 │
│  ├─ 동시 사용자 시나리오별 부하 테스트                           │
│  └─ 실제 개발자 5~10명 1~2주 파일럿                           │
└─────────────────────────────────────────────────────────┘
```


### 4.2 후보 모델 종합 비교

리스트 모델을 AA Coding Index + 보완 지표 + GPU 요구사항으로 나란히 비교합니다.

| 모델 | AA Coding Index | AA Intelligence | SWE-bench Pro | 총 파라미터 | 활성 파라미터 | 최소 GPU (FP8) | 비고 |
|------|----------------|-----------------|---------------|-----------|-------------|---------------|------|
| GLM-5.1 | 47 | 51 | 58.4% | 744B MoE | ~40B | 8× H100 80GB | 오픈소스 AA 1위, MIT |
| Qwen 3.5-397B | 43 | - | - | 397B MoE | ~17B | 8× H100 80GB | |
| MiniMax-M2.5 | 42 | - | 56.2% | - | - | - | |
| Kimi K2.5 | 40 | - | 53.8% | 1T MoE | 32B | 8× H200 141GB | 가장 큰 모델 |
| GLM-5 | 47 (추정) | 50 | 55.1% | 744B MoE | ~40B | 8× H100 80GB | GLM-5.1 전세대 |
| Gemma 4 31B | 39 (추정) | - | - | 31B Dense | 31B | 1× A100 80GB | 단일 GPU 가능 |
| Qwen 3.5-35B | 낮음 | - | - | 35B MoE | 3B | 1× RTX 4090 | A3B Coding 특화 |

핵심 관찰:
- GLM-5.1은 AA Coding Index에서 오픈소스 최상위이지만, 8× H100이 필요
- Gemma 4 31B는 Coding Index가 8점 낮지만, GPU 1장으로 서빙 가능
- **Coding Index 47 vs 39의 차이가 8× H100 vs 1× A100의 비용 차이를 정당화하는가?**
- 이 질문에 답하려면 내부 평가가 필수

### 4.3 내부 평가 설계 가이드

```
테스트 셋 구성 원칙:
  1. 실제 업무에서 나온 태스크를 사용할 것 (합성 데이터 X)
  2. 팀이 사용하는 언어/프레임워크로 구성할 것
  3. 난이도를 분산시킬 것 (쉬운 것 ~ 어려운 것)
  4. 정답이 명확한 태스크를 포함할 것 (자동 채점 가능)
  5. 양자화된 모델로 평가할 것 (FP8/INT4)

평가 메트릭:
  ├─ Pass@1: 첫 시도에 정답을 맞추는 비율
  ├─ 코드 품질 점수: 가독성, 스타일, 보안 (사람 평가)
  ├─ 환각률: 존재하지 않는 API/함수를 사용하는 비율
  ├─ 지시 따르기: 요청한 포맷/제약을 정확히 따르는 비율
  └─ 응답 시간: TTFT + 전체 생성 시간

비교 방법:
  ├─ 동일 테스트 셋으로 후보 모델 2~3개 평가
  ├─ 블라인드 평가 (어떤 모델의 출력인지 모르게)
  └─ 가능하면 여러 평가자의 점수 평균
```

---

## 5. 서빙 성능 평가 메트릭

### 5.1 핵심 메트릭

| 메트릭 | 설명 | 의미 | 목표값 (참고) |
|--------|------|------|-------------|
| TTFT (Time to First Token) | 첫 토큰까지 시간 | 사용자 체감 응답 시작 지연 | < 1~2초 |
| TPS (Tokens Per Second) | 초당 토큰 생성 속도 | 응답이 화면에 나타나는 속도 | > 30 t/s |
| Throughput (req/s) | 동시 요청 처리량 | 동시 사용자 수 지원 | 팀 규모 × 1.5 |
| P95 Latency | 95번째 백분위 지연 | 대부분의 요청 완료 시간 | < 10초 |

### 5.2 부하 테스트

```bash
# vLLM 내장 벤치마크 도구
python -m vllm.entrypoints.openai.bench_serving \
  --backend vllm \
  --base-url http://localhost:8000 \
  --model [model-name] \
  --dataset-name sharegpt \
  --num-prompts 100 \
  --request-rate 10
```

모니터링 (vLLM Prometheus):
- `vllm:time_to_first_token_seconds`
- `vllm:time_per_output_token_seconds`
- `vllm:num_requests_running`
- `vllm:gpu_cache_usage_perc`

---

## 6. 결론

```
AA Coding Index의 역할:
  ✓ 모델 후보군을 빠르게 좁히는 스크리닝 도구
  ✓ 프론티어 모델 간 상대적 위치 파악
  ✓ 에이전트형 코딩 + Python 코드 생성 능력의 프록시

AA Coding Index의 한계:
  ✗ 일반적 소프트웨어 개발 코딩 능력의 직접 측정이 아님
  ✗ Python/Bash 외 언어를 평가하지 않음
  ✗ 양자화 환경에서의 품질을 반영하지 않음
  ✗ 44+288개 태스크로 통계적 변별력에 한계
  ✗ 실제 사용 패턴(멀티턴, IDE 통합)을 반영하지 않음

따라서:
  Phase 1 (스크리닝)에서 AA Coding Index 활용 → 이미 완료
  Phase 2~4 (보완 벤치마크 + 내부 평가 + 파일럿)가 반드시 필요
```

---

## 참고 자료

- AA Intelligence Benchmarking 방법론: https://artificialanalysis.ai/methodology/intelligence-benchmarking/
- AA 리더보드: https://artificialanalysis.ai/leaderboards/models
- Terminal-Bench: https://www.tbench.ai/
- SciCode: https://scicode-bench.github.io/
- SWE-bench: https://www.swebench.com
- LiveCodeBench: https://livecodebench.github.io
- HLE: https://arxiv.org/abs/2501.14249v2
- GLM-5.1 공식: https://huggingface.co/zai-org/GLM-5.1

> Content was rephrased for compliance with licensing restrictions.
> 벤치마크 데이터는 2026년 4월 기준이며, 각 모델의 공식 발표 자료 및 AA 방법론 페이지를 기반으로 합니다.
