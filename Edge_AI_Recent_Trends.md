# Edge AI 최신 동향: 양자화·가지치기 외 압축 기술과 발전 방향

> 기존 문서 `README.md`에서 다룬 양자화·가지치기 이외의 기술과
> 2025~2026년 최신 동향을 정리한 문서입니다.

## 목차
1. [개요](#1-개요)
2. [양자화·가지치기 외 주요 기술](#2-양자화가지치기-외-주요-기술)
3. [기술별 성격 비교](#3-기술별-성격-비교)
4. [최근 동향 (2025~2026)](#4-최근-동향-20252026)
5. [하이브리드 파이프라인: 실전 순서](#5-하이브리드-파이프라인-실전-순서)
6. [PyTorch 실전 예제](#6-pytorch-실전-예제)
7. [관련 도구 및 프레임워크](#7-관련-도구-및-프레임워크)
8. [결론](#8-결론)

---

## 1. 개요

Edge AI를 위한 모델 최적화는 과거에는 "양자화 1개, 가지치기 1개"를 골라 쓰는
단일 기법 중심이었습니다. 그러나 2025년 이후 트렌드는 확실히 바뀌었습니다.

> **핵심 흐름**: 단일 기법 → **여러 기법의 결합(하이브리드)** → 더 나아가
> **하드웨어까지 고려한 공동 설계(co-design)**

- 연산 리소스(FLOPs, 지연시간) → **가지치기 / 저랭크 분해 / MoE / 희소 커널**
- 메모리 리소스(저장, RAM, 대역폭) → **양자화 / 클러스터링 / KV cache 압축**
- 성능 유지 → **Knowledge Distillation / 미세조정 / NAS**

---

## 2. 양자화·가지치기 외 주요 기술

### 2.1 Knowledge Distillation (지식 증류, KD)

큰 **teacher** 모델이 내는 확률 분포(soft target)를 작은 **student** 모델이
따라 배우게 하는 기법입니다. 단순히 "정답 라벨"이 아니라 teacher의
"판단 방식"까지 물려받기 때문에, 처음부터 작고 똑똑한 모델을 만들 수 있습니다.

- **특징**: 모델 크기가 처음부터 작음 (압축 후 재학습과는 반대 방향)
- **강점**: 정확도 회복력이 뛰어나며, 양자화·가지치기에서 잃은 성능을 되살리는
  마지막 단계로도 활용
- **비용**: teacher 실행이 필요해 학습 비용 2~5배 (일회성)

### 2.2 저랭크 분해 (SVD · LoRA · Tensor Decomposition)

큰 가중치 텐서를 **작은 랭크 행렬 두 개의 곱**으로 근사해 파라미터를 줄입니다.

```
W (1000×1000)  ≈  A (1000×100) × B^T (100×1000)   →  파라미터 100만 → 20만
```

- SVD는 정적 구조 분해, LoRA는 **학습 가능한 저랭크 보정**으로 미세조정에 사용
- 최근에는 "가지치기로 제거된 정보를 저랭크 보상(mergeable low-rank
  compensation)으로 복구"하는 결합 방식이 주목받음

### 2.3 Neural Architecture Search (NAS)

모델 구조 자체를 **자동으로 탐색**하는 기법입니다. 최근에는 정확도뿐 아니라
RAM, MACs, 지연시간, 에너지를 함께 고려하는 **Hardware-Aware NAS (HW-NAS)** 가
표준이 되었습니다.

- TinyML 표준 장비(STM32 MCU 등)에 맞는 초소형 구조를 자동 설계
- 최신 경향: **LLM이 후보 구조를 생성·설명하는 LLM-guided NAS** (LMaNet 등)

### 2.4 가중치 클러스터링 / 코드북 공유

비슷한 값의 가중치를 한 개의 값으로 묶어 **코드북(index)** 로 저장합니다.

- INT8 이하 수준의 메모리를 이유 while also shaving 저장 공간
- MCU처럼 저장 용량이 극도로 작은 환경에서 양자화의 대안으로 사용

### 2.5 KV Cache 압축 (온디바이스 LLM 전용)

언어 모델이 긴 문맥을 처리할 때 키/벨류(KV)를 메모리에 쌓는데, 이것이
온디바이스 LLM의 **최대 메모리 병목**입니다.

- 페이지 단위 관리(paging), 압축, 정리(eviction), 저비트 양자화로 해결
- 대표 연구: ThinK 등 (장문 문맥 오버헤드 감소)

### 2.6 Mixture-of-Experts (MoE)

전체 파라미터를 여러 "전문가" 서브모델로 나누고, 입력마다 **일부만 활성화**해
연산량을 크게 줄이는 구조입니다. 희소 라우팅 때문에 양자화와 결합 시
전용 보정(예: MoEQuant)이 필요합니다.

### 2.7 시스템·컴파일러 최적화 (모델 외적 기법)

모델을 줄이는 것이 아니라 **실행 방식을 최적화**하는 기법들입니다.

- 커널 퓨전(kernel fusion), 연산 재배치, 메모리 재사용
- CMSIS-NN, TVM, TensorRT, ONNX Runtime, TFLM, XNNPACK 등에서 지원
- 하드웨어(micro-Loop) 지원과 결합해야 실측 이득이 나옴

### 2.8 Adaptive / Dynamic Inference

- 여러 해상도·서브네트워크 중 상황에 맞게 선택 (배터리·부하·입력 복잡도에 따라)
- Dynamic Sparse Training: 가지치기와 재생성(regrowth)을 반복해 희소 모델을 학습

---

## 3. 기술별 성격 비교

| 기술 | 줄이는 것 | 성능 유지 방법 | 학습 비용 | 하드웨어 의존 |
|---|---|---|---|---|
| 양자화 | 메모리·대역폭(1/4~1/8) | QAT·보정 | 낮음~중간 | INT8 가속 시 유리 |
| 가지치기 | 연산(FLOPs) | 미세조정 | 중간 | 희소 커널 시 유리 |
| **지식 증류** | 모델 자체(설계부터 작게) | 학생 재학습 | **높음(일회성)** | 낮음 |
| **저랭크 분해** | 파라미터(→작은 행렬곱) | 미세조정/LoRA | 중간 | 낮음 |
| **NAS** | 구조 탐색으로 최적화 | 자동 탐색 | **매우 높음** | HW-Aware 필요 |
| **클러스터링** | 저장(코드북) | 미세조정 선택 | 낮음 | 낮음 |
| **KV cache 압축** | 런타임 문맥 메모리 | 압축·재사용 | 낮음 | 낮음 |
| **MoE** | 연산(전문가 선택) | 게이팅 학습 | 높음 | 희소 라우팅 필요 |

---

## 4. 최근 동향 (2025~2026)

### 4.1 단일 기법 → 복합 파이프라인

여러 기법을 이어 붙이는 **하이브리드 파이프라인**이 주류가 되었습니다.
단순히 "아무렇게나" 붙이는 것이 아니라 **순서가 중요**하다는 것이
실험으로 확인되었습니다. 대표 연구(PQD: Prune-Quantize-Distill) 결과:

```
Prune(가지치기) → INT8 QAT(양자화 인지 학습) → KD(지식 증류)

실측(CPU): 가지치기 단독은 거의 속도↑ 없음(불규칙 메모리 접근)
           INT8 QAT가 대부분의 지연시간 감소(2.45ms → 0.99ms, 약 2.5배)
           KD는 마지막에 정확도만 회복(Deployment 형태 변화 없음)
```

- 가지치기 = 용량 축소 + 후속 저비트 학습 안정화
- QAT = 지연시간 감소의 주역
- KD = 정확도 회복 담당

### 4.2 온디바이스 LLM의 부상

1~4B 크기의 언어 모델을 스마트폰·에지 기기에 배포하는 연구가 폭발적으로
증가했습니다. 핵심 과제:

- 저비트(INT8 → **W4A4**) 안정화: SmoothQuant, OmniQuant, FlatQuant, ResQ 등
- KV cache를 1급 서브시스템으로 취급 (paging·압축·정리)
- 배포 형식 표준화: GGUF(llama.cpp), AWQ, GPTQ
- Dell이 언급한 **"Micro LLMs"**: 작고 특화된 모델 여러 개를 조합하는 방식

### 4.3 Training-free 프루닝

LLM은 재학습 비용이 너무 커 "가지치기 후 미세조정" 패러다임이 현실적이지
못했습니다. 그래서 **1회성·무학습(one-shot, training-free)** 가지치기가 대세:
SparseGPT, Wanda, LLM-Pruner, Probe Pruning, RotPruner 등.

### 4.4 LLM-guided NAS + KD

LLM에게 후보 구조 생성을 맡기고 ViT teacher로 증류하는 방식이
TinyML 표준 MCU(STM32H7, 320KB SRAM, <100M MACs)에서 SOTA를 갱신했습니다.

- LMaNet 계열: CIFAR-100에서 74.5% (기존 SOTA MCUNet 72.86% 대비 우위)
- KD 단계만으로 정확도 +1.3~1.4%p 향상

### 4.5 하드웨어-소프트웨어 공동 설계 (Co-design)

모델 압축만으로는 실측 성능이 보장되지 않는다는 인식이 확산되어,
**러타임/컴파일러(커널 퓨전, 희소 커널)와 모델 구조를 함께 최적화**하는
방향으로 발전하고 있습니다.

- 구조적 가지치기(채널·헤드 제거)는 정형 행렬 연산을 유지 → NPU/DSP 친화
- 비구조적 가지치기는 XNNPACK·cuSPARSE 등 희소 커널 지원이 필수

### 4.6 평가 기준 변화: ALEM

"정확도·모델 크기"만 보던 것에서 **ALEM** 지표로 전환:

| 지표 | 의미 |
|---|---|
| **A**ccuracy | 성능 |
| **L**atency | 지연시간 |
| **E**nergy | 에너지(전력) |
| **M**emory | 메모리 |

- 프록시 지표(파라미터 수, 명목 FLOPs) 대신 **타깃 하드웨어에서 실측**한
  지연시간·에너지로 판정하는 것이 표준적 권장사항이 됨

---

## 5. 하이브리드 파이프라인: 실전 순서

실무에서 검증된 조합 순서를 예시로 들면:

```
1. Knowledge Distillation      → 큰 모델의 능력을 작은 구조로 이전 (최대 압축)
2. Structured Pruning          → 불필요 채널/헤드 제거 (연산 감소)
3. Fine-tuning / LoRA 보정      → 손실 복구 + 저랭크 보상
4. Quantization (PTQ → QAT)    → INT8/INT4로 메모리 압축
5. 컴파일러 최적화 배포         → TensorRT·TFLite·CMSIS-NN으로 실측 검증
```

실제 사례로 본 복합 효과 (개념 수치):

| 단계 | 크기 | 누적 비고 |
|---|---|---|
| 7B 모델 FP16 | 14 GB | 기준 |
| INT4 양자화 | 3.5 GB | 4배 절감, 품질 -1~4%p |
| + 가지치기 30% | 2.5 GB | 품질 추가 -2~3%p |
| + 증류(3B 타깃) | 1.5 GB | 품질 누적 약 -10%p (용도에 따라 허용) |

또한 폐기물 분류 TinyML 사례에서는 **증류 + PTQ**만으로
모델이 16MB → 281KB (약 57배) 줄면서 82% 정확도 유지가 보고된 바 있습니다.

---

## 6. PyTorch 실전 예제

### 6.1 Knowledge Distillation — 손실 함수

```python
import torch
import torch.nn.functional as F

def distillation_loss(student_logits, teacher_logits, labels,
                      temperature=4.0, alpha=0.7):
    """teacher의 soft probability와 실제 라벨을 함께 학습"""
    soft_target = F.softmax(teacher_logits.detach() / temperature, dim=1)
    soft_student = F.log_softmax(student_logits / temperature, dim=1)
    kd_loss = F.kl_div(soft_student, soft_target,
                       reduction="batchmean") * (temperature ** 2)
    ce_loss = F.cross_entropy(student_logits, labels)
    return alpha * kd_loss + (1 - alpha) * ce_loss
```

### 6.2 저랭크 분해 (SVD) — Linear 계층 축소

```python
import torch
import torch.nn as nn

def replace_with_svd(layer, rank):
    """nn.Linear 하나를 랭크 r 두 개의 Linear로 교체 (W ≈ A @ B^T)"""
    U, S, Vh = torch.linalg.svd(layer.weight, full_matrices=False)
    r = rank
    A = U[:, :r] * S[:r].sqrt().view(1, r)   # (out, r)
    B = Vh[:r] * S[:r].sqrt().view(r, 1)     # (r, in)

    low1 = nn.Linear(layer.in_features, r, bias=False)
    low2 = nn.Linear(r, layer.out_features)
    with torch.no_grad():
        low1.weight.copy_(B)
        low2.weight.copy_(A)
        low2.bias.copy_(layer.bias)
    return nn.Sequential(low1, low2)
```

**압축 비율 예시**: `in=out=1000, rank=100`이면
파라미터 100만(`1000×1000`) → 20만(`1000×100 + 100×1000`), 약 **5배 감소**.

### 6.3 가중치 클러스터링 (코드북) — 개념

```python
import torch

def kmeans_codebook(weight, n_clusters=16):
    """가중치를 n_clusters개의 대표값(코드북)으로 클러스터링"""
    w = weight.flatten()
    # 실제로는 k-means(예: torch 클러스터링 라이브러리) 수행
    # 아래는 개념 예시: min-max로 대표값 n개 생성
    edges = torch.linspace(w.min(), w.max(), n_clusters + 1)
    centers = (edges[:-1] + edges[1:]) / 2
    indices = torch.bucketize(w, edges[1:-1])
    return indices, centers
```

### 6.4 하이브리드 파이프라인 (개념 결합)

```python
def hybrid_pipeline(teacher, student, pruner_ratio=0.4, bits=8):
    # 1단계: KD (student를 teacher의 분포로 사전학습)
    #  단, 실제 학습 루프에서 distillation_loss 활용
    # 2단계: 가지치기
    for name, module in student.named_modules():
        if isinstance(module, nn.Conv2d):
            prune.ln_structured(module, name="weight",
                                amount=pruner_ratio, dim=0, n=2)
    # 3단계: QAT (학습 루프에서 양자화 시뮬레이션)
    # 4단계: 컴파일러 최적화 (모델 밖: TensorRT/TFLite)
    pass
```

---

## 7. 관련 도구 및 프레임워크

| 도구/프레임워크 | 역할 |
|---|---|
| TensorRT, ONNX Runtime | 하드웨어 최적화 추론, 저비트 커널 |
| TVM, OpenVINO, TFLM, CMSIS-NN | 컴파일러·커널 최적화 (MCU 포함) |
| XNNPACK, cuSPARSE | 희소 커널 (비구조적 가지치기 지원) |
| llama.cpp (GGUF), AWQ, GPTQ, bitsandbytes | LLM 저비트 배포 |
| Edge Impulse, NanoEdge AI Studio | TinyML 자동 배포 파이프라인 |
| Qualcomm AI Hub, Apple ANE | 하드웨어별 가속 딜리버리 |

---

## 8. 결론

- 미래 Edge AI는 **한 가지 기법이 아니라 "압축 스택"의 문제**입니다.
- 흐름: **양자화·가지치기 단독 → 하이브리드 파이프라인 → HW/SW 공동 설계 →
  온디바이스 LLM 전용 최적화(KV cache, 저비트, 무학습 프루닝)** 로 확장 중.
- 판단의 기준도 "정확도·파라미터 수"에서 **ALEM(정확도·지연시간·에너지·메모리)
  의 실측값**으로 옮겨가고 있습니다.

> **한 줄 요약**: "작게 + 빠르게 + 오래 쓰기"를 위해 가지치기·양자화를 축으로
> 증류·저랭크·NAS·컴파일러 최적화를 **순서에 맞게 결합**하고, 타깃 하드웨어에서
> 실측으로 판정하는 것이 2026년 Edge AI 최적화의 정석입니다.

---

### 참고 자료 (핵심)

- Prune-Quantize-Distill: "An Ordered Pipeline for Efficient Neural Network Compression" (2026)
- "On-device Large Language Models: A Survey of Model Compression and System Optimization" (Artificial Intelligence Review, 2026)
- "Large Models for Small Devices: Recent Advances and Empirical Analysis of Edge AI Deployment" (2026)
- LMaNet: "Can LLMs Revolutionize the Design of Explainable and Efficient TinyML Models?" (IJCNN 2025)
- ENAS, HW-Aware NAS 프레임워크 (IJCAI 2026)
- [ON-device LLM 참조 모음](https://github.com/LumosJiang/Awesome-On-Device-LLMs)