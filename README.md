# Edge AI: 양자화(Quantization) vs 가지치기(Pruning)

## 목차
1. [개요](#1-개요)
2. [양자화 (Quantization)](#2-양자화-quantization)
3. [가지치기 (Pruning)](#3-가지치기-pruning)
4. [두 기법의 비교](#4-두-기법의-비교)
5. [시각화 예제](#5-시각화-예제)
6. [PyTorch 실전 예제](#6-pytorch-실전-예제)
7. [실전 조합 전략](#7-실전-조합-전략)
8. [결론](#8-결론)

---

## 1. 개요

Edge AI(엣지 AI)는 스마트폰, MCU, 라즈베리파이 같은 **제한된 연산 자원**을 가진 디바이스에서
딥러닝 모델을 실행하는 기술입니다. 그러나 대형 모델은 메모리와 연산량이 크기 때문에
그대로 배포하기 어렵습니다.

이 문제를 해결하는 대표적인 기법이 바로 **양자화(Quantization)** 와 **가지치기(Pruning)** 입니다.

| 항목 | 양자화 | 가지치기 |
|---|---|---|
| 핵심 아이디어 | 숫자 정밀도 축소 | 불필요한 가중치 제거 |
| 줄어드는 것 | 메모리 + 연산 속도 | 메모리 + 연산량 |
| 손실 원인 | 반올림 오차 | 정보 소실 |
| 하드웨어 가속 | 유리 (특화 연산) | 제한적 |

---

## 2. 양자화 (Quantization)

### 2.1 개념

양자화는 모델의 가중치와 활성화 값을 **FP32(32비트 부동소수점)** 에서
**INT8(8비트 정수)** 등 낮은 정밀도의 숫자로 변환하는 기법입니다.

```
FP32: -3.14159  →  INT8: -125  (반올림 + 스케일링)
```

### 2.2 동작 원리

양자화는 실수 값을 정수로 매핑하기 위해 **스케일(scale)** 과 **제로포인트(zero point)** 를 사용합니다.

```
real_value = (int_value - zero_point) × scale
```

- **스케일(scale)**: 실수 값을 정수 값으로 변환할 때 곱하는 비율
- **제로포인트(zero point)**: 실수 0에 해당하는 정수 값

예를 들어 실제 가중치 범위가 `[-1.0, 1.0]`인 FP32 텐서를 INT8로 양자화하면:

```
FP32 가중치: [0.375, -0.5, 1.0, -1.0]
INT8 가중치: [ 223,   63,  255,   1 ]  (scale ≈ 0.00784, zero_point = 128)
```

### 2.3 유형

| 유형 | 설명 | 장단점 |
|---|---|---|
| PTQ (Post-Training Quantization) | 학습 후 바로 양자화 | 간단, 약간의 정확도 손실 |
| QAT (Quantization-Aware Training) | 양자화를 고려하며 재학습 | 높은 정확도, 오래 걸림 |

### 2.4 장점 / 단점

**장점**
- 모델 크기가 약 **4배** 줄어듦 (FP32 → INT8)
- 연산 속도가 2~4배 빨라짐 (INT8 전용 하드웨어 가속 시)
- 전력 소모 감소 → 배터리 수명 연장
- 구현이 비교적 단순

**단점**
- 작은 값에서 상대 오차가 커짐 (왜곡/distortion)
- 일부 레이어(예: 특이값 분포)에서 손실이 큼
- INT8 가속 미지원 하드웨어에서는 효용이 떨어짐

---

## 3. 가지치기 (Pruning)

### 3.1 개념

가지치기는 신경망에서 **중요도가 낮은 가중치(또는 뉴런/채널)를 제거**하여
모델을 압축하는 기법입니다. 정확도에 거의 영향을 미치지 않는 연결을 잘라냅니다.

```
기존: [1.2, 0.0, -0.05, 3.4, 0.1, -2.0]   (6개 가중치)
가지치기 후: [1.2, 0, 0, 3.4, 0, -2.0]      (3개 가중치 0으로 제거)
```

### 3.2 유형

| 유형 | 단위 | 압축률 | 하드웨어 친화성 |
|---|---|---|---|
| Unstructured Pruning | 개별 가중치 | 높음 | 낮음 (희소 행렬 처리 어려움) |
| Structured Pruning | 채널/뉴런/레이어 | 낮음 | 높음 (실제 연산량 감소) |

### 3.3 중요도 판단 기준

- **절댓값이 큰 가중치** → 중요 (작은 가중치를 먼저 제거)
- **Gradient 기반 민감도** → 손실 변화가 큰 가중치를 유지
- **BatchNorm scale** → 채널별 중요도 판단

### 3.4 학습 순서

가장 흔한 방식은 **ITERATIVE PRUNE + FINE-TUNE** 입니다.

```
1. 사전 학습된 모델 → 2. 가지치기 → 3. 미세 조정(Fine-tuning) → 4. 반복
```

### 3.5 장점 / 단점

**장점**
- 불필요한 계산을 없애 실제 연산량 감소
- 양자화보다 정확도 유지율이 높은 경우가 많음
- 구조적 가지치기 시 특화 하드웨어 없이도 속도 향상

**단점**
- Unstructured 방식은 **희소 행렬 저장** 때문에 실질 압축 효과가 제한적
- 가지치기 후 미세조정이 필수 (손실 발생)
- 구조 설계에 따라 성능 차이가 큼

---

## 4. 두 기법의 비교

| 비교 항목 | 양자화 (INT8) | 가지치기 (Structured) |
|---|---|---|
| 목표 | 정밀도 ↓ | 밀도 ↓ |
| 감소 효과 | 메모리 4배, 속도 ↑ | 연산 FLOPs ↓, 메모리 ↓ |
| 정확도 영향 | 항상 약간 손실 | 미세조정 시 복구 가능 |
| 하드웨어 가속 | INT8 가속 (매우 유리) | 컨볼루션 구조 축소로 간접 가속 |
| 구현 난이도 | 쉽다 (PTQ 기준) | 중간 |
| 재학습 필요 | QAT만 필요 | 거의 항상 필요 |
| SIMD/벡터화 | 우수 | 제한적 |

### 핵심 포인트

> **양자화는 "메모리를 줄이고 계산을 빠르게"** 하는 데 강하고,
> **가지치기는 "실제 연산량(FLOPs)을 줄이는"** 데 강합니다.
> 실제 배포에서는 **두 기법을 함께 사용**하는 것이 가장 효과적입니다.

---

## 5. 시각화 예제

다음은 FP32 가중치를 INT8로 양자화하는 과정과 가지치기를 시각화하는 간단한 개념도입니다.

### 5.1 양자화 (연속 값 → 이산 값)

```
FP32 값:  -1.0    -0.5     0.0     0.5     1.0
               ↓ (스케일링)
INT8 값:    1      63     128     193     255

       [----정밀도 손실(반올림)----------]
```

### 5.2 가지치기 (밀집 → 희소)

```
가지치기 전 (밀집)                     가지치기 후 (희소)
┌───┬───┬───┐                        ┌───┬───┬───┐
│1.2│0.8│0.0│   threshold=0.1        │1.2│0.8│0.0│
├───┼───┼───┤     ↓                  ├───┼───┼───┤
│0.1│2.5│1.1│     ↓                  │0.0│2.5│1.1│
├───┼───┼───┤                        ├───┼───┼───┤
│0.3│0.0│1.5│                        │0.0│0.0│1.5│
└───┴───┴───┘                        └───┴───┴───┘
 9개 가중치(100%)                      6개 가중치 (66%)
```

---

## 6. PyTorch 실전 예제

아래 코드는 PyTorch에서 간단한 MLP를 대상으로 가지치기와 양자화를
실제로 적용하고 그 효과를 측정하는 예제입니다.

### 6.1 환경

```bash
pip install torch torchvision torchaudio
```

### 6.2 모델 정의

```python
import torch
import torch.nn as nn
import torch.nn.utils.prune as prune
import torch.nn.functional as F

class SimpleMLP(nn.Module):
    def __init__(self, input_size=28*28, hidden=128, num_classes=10):
        super().__init__()
        self.fc1 = nn.Linear(input_size, hidden)
        self.fc2 = nn.Linear(hidden, hidden)
        self.fc3 = nn.Linear(hidden, num_classes)

    def forward(self, x):
        x = x.view(x.size(0), -1)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        return self.fc3(x)
```

### 6.3 가지치기 (Pruning) 적용

```python
def apply_pruning(model, amount=0.3):
    """전체 가중치 중 30%를 L1 중요도 기준으로 가지치기"""
    for layer in [model.fc1, model.fc2, model.fc3]:
        prune.l1_unstructured(layer, name="weight", amount=amount)
    return model

# 가지치기된 가중치 비율 확인
def sparsity(model):
    total = 0
    zeros = 0
    for layer in [model.fc1, model.fc2, model.fc3]:
        w = layer.weight
        total += w.numel()
        zeros += (w == 0).sum().item()
    return zeros / total

model = SimpleMLP()
apply_pruning(model, amount=0.3)
print(f"가지치기 후 희소도(Sparsity): {sparsity(model):.2%}")
```

**출력 예시**

```
가지치기 후 희소도(Sparsity): 30.00%
```

### 6.4 양자화 (Quantization) 적용 (PTQ)

```python
from torch.ao.quantization.quantize_fx import prepare_fx, convert_fx

# 모델을 양자화 가능한 형태로 준비 후 8비트 연산으로 변환
def apply_quantization(model):
    model.eval()
    qconfig = torch.ao.quantization.QConfig(
        activation=torch.ao.quantization.default_observer,
        weight=torch.ao.quantization.default_weight_observer,
    )
    # 실제 배포에서는 calibration 데이터로 observer 실행 필요
    prepared = prepare_fx(model, {"": qconfig}, example_inputs=torch.randn(1, 28*28))
    converted = convert_fx(prepared)
    return converted
```

> **참고**: PTQ는 실제 배포 환경에서 **calibration set**을 이용해
> 활성화 값의 분포를 관찰한 뒤 양자화 파라미터를 확정합니다.

### 6.5 모델 크기 비교

```python
def get_model_size(param_generator):
    """모델 파라미터 크기(MB) 계산"""
    total = 0
    for p in param_generator:
        if p.dtype != torch.float32:
            continue
        total += p.numel() * 4  # FP32 = 4 bytes
    return total / (1024 ** 2)

# FP32 모델 크기
fp32_size = get_model_size(model.parameters())
print(f"FP32 모델 크기: {fp32_size:.2f} MB")

# 가지치기 후 (0인 값도 여전히 메모리 차지 → 크기 변화 없음)
print(f"가지치기 모델 크기: {fp32_size:.2f} MB (메모리는 동일!)")

# INT8 모델 크기 (가중치당 1 byte)
q_model = apply_quantization(model)
int8_size = get_model_size(q_model.parameters()) / 4
print(f"INT8 모델 크기: {int8_size:.2f} MB (약 1/4로 축소)")
```

**핵심 인사이트**: 가지치기만으로는 **파일 크기가 줄지 않습니다**.
메모리 절감 효과를 실제로 보려면 가지치기된 0을 제거하거나(희소 저장 형식),
양자화를 함께 적용해야 합니다.

### 6.6 가지치기 → 양자화 결합 파이프라인

```python
def full_compression_pipeline(model, prune_amount=0.3):
    # 1단계: 가지치기
    model = apply_pruning(model, amount=prune_amount)
    # 2단계: 미세조정 (실제 프로젝트에서는 여기서 재학습)
    # 3단계: 양자화
    model = apply_quantization(model)
    return model

compressed = full_compression_pipeline(model)
print("가치치기 + 양자화 결합 파이프라인 완료")
```

---

## 7. 실전 조합 전략

Edge AI 배포에서 흔히 사용하는 순서:

```
1. 사전 학습 (Pre-training)
        ↓
2. 가지치기 (Structured Pruning)  → FLOPs 감소
        ↓
3. 미세 조정 (Fine-tuning)        → 정확도 복구
        ↓
4. 양자화 (PTQ 또는 QAT)          → 메모리 4배 축소
        ↓
5. Edge 디바이스 배포 (TFLite, ONNX Runtime, MCU)
```

### 프레임워크별 지원

| 프레임워크 | 가지치기 | 양자화 | 배포 타깃 |
|---|---|---|---|
| PyTorch | `torch.prune`, `torch.nn.utils.prune` | `torch.ao.quantization` | Android, iOS |
| TensorFlow | `tensorflow_model_optimization` | `TFLite` | Mobile, MCU |
| ONNX Runtime | - | INT8 / FP16 | 모든 플랫폼 |
| TensorRT | 엔진 최적화 | INT8 / FP16 | NVIDIA Jetson |

---

## 8. 결론

| | 양자화 | 가지치기 |
|---|---|---|
| 메모리 절감 | ★★★★★ | ★★☆ (구조적 시 ★★★★) |
| 속도 향상 | ★★★★ | ★★★ |
| 정확도 유지 | ★★★ | ★★★★ (미세조정 시) |
| 구현 용이성 | ★★★★★ | ★★★ |

- **메모리가 병목**인 경우 → **양자화**가 정답 (4배 축소, 즉각 효과)
- **연산량(FLOPs)이 병목**인 경우 → **가지치기**가 정답
- **최상의 결과** → 가지치기로 연산량을 줄이고, 양자화로 메모리와 속도를 최적화

> **Best Practice**: 두 기법은 경쟁 관계가 아니라 **보완 관계**입니다.
> 실제 Edge AI 프로젝트에서는 정확도-속도-크기 사이의 트레이드오프를
> 평가 지표(ex: Top-1 accuracy, latency, model size)로 검증한 뒤
> 최적의 조합 비율을 찾아야 합니다.

---

### 추가 참고 자료

- [PyTorch Pruning Tutorial](https://pytorch.org/tutorials/intermediate/pruning_tutorial.html)
- [PyTorch Quantization Docs](https://pytorch.org/docs/stable/quantization.html)
- [TensorFlow Model Optimization](https://www.tensorflow.org/model_optimization)