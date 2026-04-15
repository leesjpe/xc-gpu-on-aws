# 모델별 GPU 사이징 계산 가이드

> LLM 모델을 vLLM으로 서빙할 때 필요한 GPU 수량과 VRAM을 계산하는 실무 가이드  
> 대상 인스턴스: **p5en.48xlarge (H200 × 8, 총 가용 VRAM: 1,128 GB)**

---

## ⚠️ 본 계산의 한계 및 주의사항

본 문서의 VRAM 수치는 **모델 가중치 기준 이론값**이며, 아래 항목은 반영되지 않습니다.

| 항목 | 설명 |
|------|------|
| KV Cache 메모리 | 시퀀스 길이, 배치 크기, 동시 사용자 수에 따라 수십~수백 GB 추가 발생 가능 |
| vLLM 프레임워크 최적화 | Prefix Caching, PagedAttention 등 실제 메모리 절감 미반영 |
| CUDA 오버헤드 | 커널 메모리, 드라이버 메모리, CUDA 그래프 등 미반영 |
| Activation 메모리 | 배치 크기 및 시퀀스 길이에 따라 달라짐 |
| 성능(TPS) 예측 | 실제 환경에서는 하드웨어 특성, 배치 전략에 따라 크게 다를 수 있음 |

> **본 계산은 인스턴스 선택을 위한 초기 방향 설정(ballpark estimation) 목적입니다.**  
> 정확한 사이징이 필요하신 경우 담당자에게 문의해 주세요.

---

## 1. VRAM 계산 방법론

### 1.1 모델 가중치 메모리

```
가중치 VRAM = 파라미터 수(B) × 정밀도별 바이트

정밀도별 바이트:
  BF16 / FP16  →  2 bytes/param
  FP8          →  1 byte/param
  INT4         →  0.5 bytes/param
```

### 1.2 총 VRAM 추정 (간이 공식)

```
총 필요 VRAM ≈ 가중치 VRAM × 1.2  (오버헤드 20% 포함)

※ 짧은 컨텍스트 / 소수 동시 사용자 기준
※ 긴 컨텍스트 (64K 이상) 또는 다수 동시 요청 시 KV Cache 별도 산정 필요
```

### 1.3 MoE 모델 주의사항

MoE(Mixture-of-Experts) 모델은 **활성 파라미터만 연산**하지만, **전체 파라미터를 VRAM에 상주**시켜야 합니다.

```
예시: GLM-5.1 (753.9B total, ~40B active per token)
  → 연산은 ~40B만 사용
  → 메모리는 753.9B 전체 필요
```

MoE의 실질적 이점은 **메모리가 아닌 연산 효율(TPS 향상)**입니다.

### 1.4 GPU 수량 계산

```
필요 GPU 수 = ⌈ 총 필요 VRAM ÷ (GPU당 VRAM × 0.9) ⌉

※ Tensor Parallelism은 2의 거듭제곱이 효율적 (1, 2, 4, 8)
※ vLLM 기본값: gpu_memory_utilization = 0.9
```

---

## 2. 대상 인스턴스 스펙

| 인스턴스 | GPU | GPU 수 | GPU당 VRAM | 총 VRAM |
|----------|-----|--------|------------|---------|
| p5en.48xlarge | NVIDIA H200 | 8 | 141 GB | **1,128 GB** |

---

## 3. 모델별 정확한 파라미터 및 VRAM 계산

> 파라미터 수는 HuggingFace safetensors 메타데이터 기준 실측값입니다.

### 3.1 모델 스펙 및 가중치 VRAM

| 모델 | 아키텍처 | 총 파라미터 (실측) | 활성 파라미터 | 라이선스 |
|------|---------|-------------------|--------------|---------|
| Gemma 4 27B-IT | Dense | ~27B | 27B | Gemma ToS |
| Gemma 4 31B | Dense | ~31B | 31B | Gemma ToS |
| Qwen3.5-35B-A3B | MoE | **35.95B** | ~3B | Apache 2.0 |
| Qwen3.5-397B-A17B | MoE | **403.4B** | ~17B | Apache 2.0 |
| MiniMax-M2.5 | MoE | **228.7B** | ~45B | Modified MIT |
| GLM-5.1 | MoE | **753.9B** | ~40B | MIT |
| Kimi K2.5 | MoE | **1,058.6B** | ~32B | Modified MIT |

> ※ Gemma 4 27B/31B는 HuggingFace 접근 제한으로 공개 자료 기준 추정값

### 3.2 VRAM 계산표 (가중치 기준, 오버헤드 미포함)

| 모델 | 총 파라미터 | BF16/FP16 | FP8 | INT4 |
|------|-----------|-----------|-----|------|
| Gemma 4 27B-IT | ~27B | ~54 GB | ~27 GB | ~14 GB |
| Gemma 4 31B | ~31B | ~62 GB | ~31 GB | ~16 GB |
| Qwen3.5-35B-A3B | 35.95B | **~72 GB** | **~36 GB** | **~18 GB** |
| Qwen3.5-397B-A17B | 403.4B | **~807 GB** | **~403 GB** | **~202 GB** |
| MiniMax-M2.5 | 228.7B | **~457 GB** | **~229 GB** | **~114 GB** |
| GLM-5.1 | 753.9B | **~1,508 GB** | **~754 GB** | **~377 GB** |
| Kimi K2.5 | 1,058.6B | **~2,117 GB** | **~1,059 GB** | **~529 GB** |

### 3.3 오버헤드 포함 총 필요 VRAM 추정 (×1.2)

| 모델 | BF16/FP16 | FP8 | INT4 |
|------|-----------|-----|------|
| Gemma 4 27B-IT | ~65 GB | ~32 GB | ~17 GB |
| Gemma 4 31B | ~74 GB | ~37 GB | ~19 GB |
| Qwen3.5-35B-A3B | ~86 GB | ~43 GB | ~22 GB |
| Qwen3.5-397B-A17B | ~968 GB | ~484 GB | ~242 GB |
| MiniMax-M2.5 | ~549 GB | ~275 GB | ~137 GB |
| GLM-5.1 | ~1,810 GB | ~905 GB | ~452 GB |
| Kimi K2.5 | ~2,540 GB | ~1,271 GB | ~635 GB |

---

## 4. p5en.48xlarge 단일 노드 탑재 가능 여부

> 기준: p5en.48xlarge 총 가용 VRAM **1,128 GB**  
> 판정: ✅ 가능 / ⚠️ 조건부 가능 / ❌ 불가

| 모델 | BF16/FP16 | FP8 | INT4 | 비고 |
|------|-----------|-----|------|------|
| Gemma 4 27B-IT | ✅ | ✅ | ✅ | 여유 있음 |
| Gemma 4 31B | ✅ | ✅ | ✅ | 여유 있음 |
| Qwen3.5-35B-A3B | ✅ | ✅ | ✅ | 여유 있음 |
| Qwen3.5-397B-A17B | ❌ | ✅ | ✅ | FP8 이하 권장 |
| MiniMax-M2.5 | ⚠️ | ✅ | ✅ | BF16은 컨텍스트 제한 필요 |
| **GLM-5.1** | ❌ | ⚠️ | ✅ | **FP8: 컨텍스트 단축 필요, INT4 권장** |
| Kimi K2.5 | ❌ | ❌ | ⚠️ | INT4도 여유 없음, 멀티 노드 권장 |

### GLM-5.1 상세

GLM-5.1은 고객이 주요 검토 중인 모델로 별도 상세 분석을 제공합니다.

```
모델명:         GLM-5.1 (zai-org/GLM-5.1)
총 파라미터:    753,864,119,552 params  ≈  753.9B  (HuggingFace 실측)
활성 파라미터:  ~40B per token
아키텍처:      MoE (GlmMoeDsaForCausalLM)
원본 정밀도:    BF16
라이선스:      MIT
컨텍스트:      최대 200K tokens
```

| 정밀도 | 가중치 VRAM | 오버헤드 포함 | p5en.48xlarge (1,128GB) |
|--------|------------|-------------|------------------------|
| BF16   | 1,508 GB   | ~1,810 GB   | ❌ 불가 |
| FP8    | 754 GB     | ~905 GB     | ⚠️ 가능 (KV Cache 최소화 필요) |
| INT4   | 377 GB     | ~452 GB     | ✅ 여유 있음 |

**권장 구성:**

```bash
# FP8 서빙 (컨텍스트 제한 필요)
python -m vllm.entrypoints.openai.api_server \
  --model ./glm-5.1 \
  --tensor-parallel-size 8 \
  --quantization fp8 \
  --kv-cache-dtype fp8 \
  --max-model-len 32768 \
  --gpu-memory-utilization 0.90 \
  --trust-remote-code \
  --port 8000

# INT4 서빙 (여유 있음, 긴 컨텍스트 가능)
python -m vllm.entrypoints.openai.api_server \
  --model ./glm-5.1 \
  --tensor-parallel-size 8 \
  --quantization awq \
  --max-model-len 131072 \
  --gpu-memory-utilization 0.90 \
  --trust-remote-code \
  --port 8000
```

---

## 5. 빠른 참조

### 정밀도별 바이트

| 정밀도 | bytes/param | 비고 |
|--------|-------------|------|
| BF16 / FP16 | 2 | 기본 정밀도 |
| FP8 | 1 | H100/H200 네이티브 지원 |
| INT8 | 1 | FP8보다 정확도 낮을 수 있음 |
| INT4 | 0.5 | 메모리 최소화, 품질 저하 가능 |

### 단순 계산 공식

```
필요 VRAM (GB) ≈ 파라미터(B) × 정밀도 바이트 × 1.2

예시: GLM-5.1 FP8
  753.9 × 1 × 1.2 = 904.7 GB  →  p5en.48xlarge 탑재 가능 (컨텍스트 제한 필요)
```

---

## 6. 문의

본 계산은 이론적 추정값이며, 실제 서빙 환경 구성, vLLM 최적화, 성능 벤치마크 등에 대한 지원이 필요하신 경우 담당자에게 문의해 주세요.

---

> 본 문서의 파라미터 수치는 HuggingFace safetensors 메타데이터 기준 실측값입니다.  
> VRAM 수치는 가중치 기준 이론값으로, vLLM 최적화 및 런타임 변수는 미반영되어 있습니다.  
> 최종 업데이트: 2026년 4월
