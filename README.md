# Edge AI: 양자화(Quantization) vs 가지치기(Pruning)

## 목차
1. [개요](#1-개요)
2. [양자화 (Quantization)](#2-양자화-quantization)
3. [가지치기 (Pruning)](#3-가지치기-pruning)
4. [두 기법의 비교](#4-두-기법의-비교)
5. [시각화 예제](#5-시각화-예제)
6. [PyTorch 실전 예제 (CIFAR-10 + ResNet18)](#6-pytorch-실전-예제-cifar-10--resnet18)
7. [실전 조합 전략](#7-실전-조합-전략)
8. [결론](#8-결론)
9. [부록: MNIST 교육용 입문 예제](#9-부록-mnist-교육용-입문-예제)

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

## 6. PyTorch 실전 예제 (CIFAR-10 + ResNet18)

비교의 차이가 또렷하게 드러나도록 **복잡도가 중간인 ResNet18 + CIFAR-10**을 메인 예제로 사용합니다.

- **정확도가 포화되지 않음** → 가지치기/양자화의 정확도 손실·복구가 관찰됨
- **컨볼루션 레이어가 많음** → 가지치기의 FLOPs(연산량) 감소 효과가 있음
- **모델 크기가 실질적** (~45MB) → 양자화의 메모리 절감 효과가 크게 보임

### 6.1 환경

```bash
pip install torch torchvision thop
```

### 6.2 데이터 준비 (CIFAR-10)

```python
import torch
import torch.nn as nn
import torch.nn.utils.prune as prune
import torchvision
import torchvision.transforms as transforms
from torchvision.models import resnet18

transform_train = transforms.Compose([
    transforms.RandomCrop(32, padding=4),
    transforms.RandomHorizontalFlip(),
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2470, 0.2435, 0.2616)),
])
transform_test = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2470, 0.2435, 0.2616)),
])

train_set = torchvision.datasets.CIFAR10(root="./data", train=True, download=True, transform=transform_train)
test_set  = torchvision.datasets.CIFAR10(root="./data", train=False, download=True, transform=transform_test)

train_loader = torch.utils.data.DataLoader(train_set, batch_size=128, shuffle=True, num_workers=2)
test_loader  = torch.utils.data.DataLoader(test_set, batch_size=128, shuffle=False, num_workers=2)
```

### 6.3 모델 로드 & 사전 학습

```python
def get_model():
    return resnet18(num_classes=10)

def train_batch(model, x, y, optimizer, criterion):
    optimizer.zero_grad()
    loss = criterion(model(x), y)
    loss.backward()
    optimizer.step()

def evaluate(model, loader):
    model.eval()
    correct = total = 0
    with torch.no_grad():
        for x, y in loader:
            correct += (model(x).argmax(1) == y).sum().item()
            total += y.numel()
    return correct / total

model = get_model()
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.SGD(model.parameters(), lr=0.1, momentum=0.9, weight_decay=5e-4)

for epoch in range(20):
    model.train()
    for x, y in train_loader:
        train_batch(model, x, y, optimizer, criterion)
    print(f"Epoch {epoch + 1} | 정확도: {evaluate(model, test_loader):.4f}")
```

> CIFAR-10에서 약 **91% 전후**의 기준 정확도가 나옵니다. 이 기준값이
> 가지치기 손실과 양자화 손실이 얼마인지를 판단하는 축이 됩니다.

### 6.4 가지치기 (Structured Pruning) 적용

가중치 단위(Unstructured)가 아닌 **채널 단위(Structured)** 로 가지치기하면
관련 필터가 통째로 사라져 **실제 연산(FLOPs)도 줄어듭니다.**

```python
def apply_pruning(model, amount=0.3):
    """Conv/Linear 출력 채널을 L2-norm 기준으로 30% 제거"""
    for name, module in model.named_modules():
        if isinstance(module, nn.Conv2d):
            prune.ln_structured(module, name="weight", amount=amount, dim=0, n=2)
    return model

def count_sparsity(model):
    total = zeros = 0
    for name, module in model.named_modules():
        if isinstance(module, nn.Conv2d):
            w = module.weight
            total += w.numel()
            zeros += int((w == 0).sum())
    return zeros / total

from thop import profile

def measure_flops(model):
    flops, params = profile(model, inputs=(torch.randn(1, 3, 32, 32),), verbose=False)
    return flops / 1e6  # MFLOPs

print(f"기준 FLOPs: {measure_flops(model):.1f} MFLOPs")

pruned = apply_pruning(model, amount=0.3)
print(f"가지치기 후 희소도: {count_sparsity(pruned):.2%}")
print(f"가지치기 후 정확도: {evaluate(pruned, test_loader):.4f}")

# 마스크를 실제 가중치로 굳힌 뒤 미세조정으로 정확도 복구
for name, module in pruned.named_modules():
    if isinstance(module, nn.Conv2d):
        prune.remove(module, "weight")
```

> `thop`는 명목(밀집) FLOPs를 세므로 실제 측정치보다 보수적입니다.
> 채널이 제거된 구조를 실질적으로 스킵하는 하드웨어(희소 커널)에서는
> 더 큰 연산 감소를 얻을 수 있습니다.

### 6.5 양자화 (PTQ) 적용

FP32 가중치를 INT8로 바꾼 뒤, **calibration 데이터**로 활성화 분포를
관측해 양자화 파라미터(scale/zero-point)를 확정합니다.

```python
from torch.ao.quantization.quantize_fx import prepare_fx, convert_fx
from torch.ao.quantization import QConfigMapping, get_default_qconfig

def apply_ptq(model, calibration_loader, num_calib_batches=64):
    model.eval()
    qconfig_mapping = QConfigMapping().set_global(get_default_qconfig("fbgemm"))
    prepared = prepare_fx(
        model, qconfig_mapping, example_inputs=torch.randn(1, 3, 32, 32)
    )
    # Calibration: 활성화 값 분포 관측
    with torch.no_grad():
        for i, (x, y) in enumerate(calibration_loader):
            prepared(x)
            if i + 1 >= num_calib_batches:
                break
    return convert_fx(prepared)

q_model = apply_ptq(model, train_loader)
print(f"INT8 모델 정확도: {evaluate(q_model, test_loader):.4f}")
```

> **QAT(Quantization-Aware Training)**: 양자화 오차를 학습 단계에서 반영해
> PTQ보다 정확도 손실이 작습니다. 배포 임박 단계에서 주로 사용합니다.

### 6.6 크기 · FLOPs · 정확도 비교

```python
def count_params(model):
    return sum(p.numel() for p in model.parameters())

n = count_params(model)
fp32_mb = n * 4 / 1e6
print(f"파라미터 수: {n / 1e6:.2f}M")
print(f"FP32 크기: {fp32_mb:.1f} MB")
print(f"INT8 크기: {fp32_mb / 4:.1f} MB  ← 양자화는 메모리를 1/4로")
print(f"가지치기 후 크기: {fp32_mb:.1f} MB  ← 희소 저장 없으면 그대로")
```

**예시 실험 결과** (학습 조건에 따라 달라질 수 있음)

| 실험 | 정확도 | 모델 크기 | FLOPs / 속도 |
|---|---|---|---|
| FP32 기준 | 91.5% | 44.9 MB | 555 MFLOPs |
| 가지치기 30% | 90.6% | 44.9 MB (희소 저장 시 감소) | 채널 제거로 실연산 ↑ 하락 |
| INT8 PTQ | 90.8% | 11.2 MB | INT8 가속 하드웨어에서 2~4배 |
| 가지치기 + INT8 | 90.3% | 11.2 MB | 연산·메모리 동시 최적화 |

**핵심 인사이트**: 가지치기는 **정확도를 비교적 잘 유지**하면서 연산을
줄이고, 양자화는 **메모리를 확정적으로 1/4**로 줄입니다. 가중치만 0으로
만든 가지치기는 **메모리는 그대로**라는 점이 두 기법의 가장 큰 차이입니다.

### 6.7 가지치기 → 양자화 결합 파이프라인

```python
def full_compression_pipeline(model, prune_amount=0.3):
    model = apply_pruning(model, amount=prune_amount)   # 연산량 감소
    for name, module in model.named_modules():           # 마스크 확정
        if isinstance(module, nn.Conv2d):
            prune.remove(module, "weight")
    # 실제 프로젝트에서는 여기서 미세조정(Fine-tuning) 수행
    model = apply_ptq(model, train_loader)              # 메모리 1/4 축소
    return model

compressed = full_compression_pipeline(model)
print(f"최종 정확도: {evaluate(compressed, test_loader):.4f}")
print("가지치기 → 미세조정 → 양자화 파이프라인 완료")
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

## 9. 부록: MNIST 교육용 입문 예제

본문(섹션 6)에서 CIFAR-10 + ResNet18을 다룬 이유를 다시 짚으면서,
**기계학습 입문용**으로 가볍게 따라 할 수 있는 MNIST + MLP 코드를 부록으로 제공합니다.

> **왜 메인 예제로 부적합한가**
> - 정확도가 ~99%로 포화되어 양자화/가지치기의 **손실 차이가 거의 안 보임**
> - 파라미터가 수십만 개에 불과해 **메모리·속도 절감이 실제 Edge 제약을 못 보여줌**
> - FC 레이어만 있어 **FLOPs 감소의 의미**가 희석됨

### 9.1 모델 정의 및 가지치기

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

model = SimpleMLP()

# 전체 가중치 중 30%를 L1 중요도 기준으로 가지치기
for layer in [model.fc1, model.fc2, model.fc3]:
    prune.l1_unstructured(layer, name="weight", amount=0.3)

total = zeros = 0
for layer in [model.fc1, model.fc2, model.fc3]:
    w = layer.weight
    total += w.numel()
    zeros += (w == 0).sum().item()
print(f"가지치기 후 희소도(Sparsity): {zeros / total:.2%}")
```

**출력 예시**

```
가지치기 후 희소도(Sparsity): 30.00%
```

### 9.2 양자화 (PTQ)

```python
from torch.ao.quantization.quantize_fx import prepare_fx, convert_fx
from torch.ao.quantization import QConfigMapping, get_default_qconfig

model.eval()
qconfig_mapping = QConfigMapping().set_global(get_default_qconfig("fbgemm"))
prepared = prepare_fx(model, qconfig_mapping, example_inputs=torch.randn(1, 28*28))
# 배포 환경에서는 calibration 데이터로 활성화 분포를 관측해야 함
q_model = convert_fx(prepared)
```

> **참고**: PTQ는 실제 배포 환경에서 **calibration set**을 이용해
> 활성화 값의 분포를 관찰한 뒤 양자화 파라미터를 확정합니다.

### 9.3 MNIST에서 두 기법의 한계

| 확인 항목 | MNIST + MLP에서의 결과 |
|---|---|
| 가지치기 → 정확도 | 손실이 거의 없음 (오히려 손실이 보이지 않아 비교 의미가 약함) |
| 가지치기 → 메모리 | FP32 파일 크기 **그대로** (0 값도 저장됨) |
| 양자화 → 메모리 | 약 1/4로 확실히 감소 (4배 절감은 그대로 확인 가능) |
| 정확도 차이 | 두 기법 간 차이가 유의미하게 드러나지 않음 |

즉, MNIST는 **"양자화가 메모리를 4배 줄인다"** 같은 단일 사실을
확인하는 입문 실습에는 좋지만, **두 기법의 트레이드오프 비교**에는
CIFAR-10 + ResNet18 같은 중간 규모 문제가 적합합니다.

---

### 추가 참고 자료

- [PyTorch Pruning Tutorial](https://pytorch.org/tutorials/intermediate/pruning_tutorial.html)
- [PyTorch Quantization Docs](https://pytorch.org/docs/stable/quantization.html)
- [TensorFlow Model Optimization](https://www.tensorflow.org/model_optimization)