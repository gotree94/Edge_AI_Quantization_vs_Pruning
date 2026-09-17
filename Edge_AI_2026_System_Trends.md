# Edge AI 2026년 하반기 동향: 시스템·하드웨어·배포 아키텍처

> 기존 문서 `Edge_AI_Recent_Trends.md`(모델 압축 기법 중심)와 달리,
> 2026년 8~9월 시점의 **시스템·하드웨어·배포 아키텍처 동향**을 다룹니다.

## 목차
1. [요약: 트렌드 축의 이동](#1-요약-트렌드-축의-이동)
2. [SLM: 온디바이스 언어모델의 골디락스 존](#2-slm-온디바이스-언어모델의-골디락스-존)
3. [NPU 하드웨어 생태계의 성숙](#3-npu-하드웨어-생태계의-성숙)
4. [MCU와 TinyML: 저전력 엣지의 기반](#4-mcu와-tinyml-저전력-엣지의-기반)
5. [병목의 이동: 컴퓨트 → 메모리 대역폭](#5-병목의-이동-컴퓨트--메모리-대역폭)
6. [Agentic AI와 Physical AI](#6-agentic-ai와-physical-ai)
7. [MoE가 모바일의 핵심 전략으로](#7-moe가-모바일의-핵심-전략으로)
8. [Hybrid Edge-Cloud가 기본 설계로](#8-hybrid-edge-cloud가-기본-설계로)
9. [런타임·소프트웨어 스택](#9-런타임소프트웨어-스택)
10. [평가·벤치마크의 변화](#10-평가벤치마크의-변화)
11. [결론](#11-결론)

---

## 1. 요약: 트렌드 축의 이동

2025년 이전의 Edge AI 논의는 **"모델을 어떻게 작게 만드는가"** (압축 기법)가
중심이었습니다. 그러나 2026년 하반기의 논의는 확실히 축이 바뀌었습니다.

> **"어떻게 압축할까" → "무엇을 어디에서, 어떤 하드웨어로 실행할까"**

- 모델: 소형화가 이미 산업 표준 → **SLM(작은 언어모델)** 생태계 정착
- 하드웨어: NPU가 모든 플래그십 SoC에 기본 내장 → **하드웨어가 소프트웨어를 유인**
- 워크로드: 단일 추론 → **지속적·상시 작동하는 에이전트(Agent)**
- 아키텍처: 엣지 vs 클라우드 대립 → **Hybrid(분업)가 기본 설계**

---

## 2. SLM: 온디바이스 언어모델의 골디락스 존

2026년 산업계가 합의한 온디바이스 언어 모델의 적정 크기는
**서브빌리언(sub-1B) ~ 수십억 파라미터(단수~10B)** 대역입니다.

| 모델/플랫폼 | 크기 | 비고 |
|---|---|---|
| Apple 온디바이스 모델 | ~3B (혼합 3.7bit/가중치) | 내부 도구 Talaria로 지연·전력 균형 |
| Google Gemini Nano (nano-v3) | 1.8B / 3.25B (4bit) | Pixel 10 Pro 탑재, AICore 관리 |
| Meta Llama 3.2 계열, MS Phi 계열 | ~1B~ | MLPerf/에코시스템 표준 축 |
| Gemma 3/4 (1B ~ 12B) | 1B~12B | 12B는 16GB 메모리 노트북에서 로컬 실행 |

**핵심 포인트**
- **SLM-first, LLM-backup**: 일상 요청의 80%는 로컬 SLM이 처리하고,
  복잡한 요청(~20%)만 클라우드 LLM으로 라우팅 → 전체 AI 비용 60~70% 절감 주장
- Gartner 전망: **2027년까지 조직이 특화 SLM을 범용 LLM보다 3배 더 많이 사용**
- 모델 설계 방향도 "클라우드 대형 모델의 압축 복사본"이 아니라
  **"디바이스·태스크에 맞게 처음부터 최적화된 작은 모델"**로 전환

---

## 3. NPU 하드웨어 생태계의 성숙

2026년 출시되는 주요 플래그십 SoC에는 전부 NPU가 내장되어 있으며,
에서 제조사들이 "에이전트 AI"를 하드웨어 설계의 최우선 조건으로 삼습니다.

| 플랫폼 | 주요 내용 |
|---|---|
| Qualcomm Snapdragon 8 Elite Gen 6 | Hexagon NPU(80~85 TOPS), 신설 **Element Accelerator**, 공유 메모리 +50%, MoE 라우팅 지원 |
| MediaTek Dimensity 9600 Pro | 최초 2nm 모바일 SoC, dual-NPU(NPU 1090 + Super Efficient NPU 2.0), 30B MoE/RWKV cache 압축 |
| Apple A18 Pro / M4·M5 | Neural Engine + GPU 내 **Neural Accelerator** 융합, M5 614GB/s 통합 메모리(70B 모델 구동 가능 주장) |
| Intel Panther Lake / AMD Ryzen AI 400 | Microsoft Copilot+ 인증 기준 40 TOPS 상회, 48~50 TOPS급 |
| NVIDIA Jetson Thor / T3000·T2000 | 최대 2070 FP4 TFLOPS, 2026.07 소형 모듈 발표(2027 Q1 상용 예정) |
| Arm CSS for Mobile 2 | C2 CPU(듀얼 SME2), Mali G2-Ultra NX(신경 가속기 통합 GPU), "에이전트 AI용 컴퓨트 플랫폼" 표방 |

**업계 전망**
- ABI Research: 글로벌 Edge AI 칩셋 시장 2026년 **344억 달러 → 2031년 960억 달러**,
  GPU가 2030년께 CPU를 추월 전망
- 전체 통합 메모리·분산 NPU 아키텍처 확산으로 "엣지 vs 클라우드"의 하드웨어
  경계 자체가 흐려지는 방향

---

## 4. MCU와 TinyML: 저전력 엣지의 기반

SLM·NPU 논의가 "고성능 온디바이스"를 다룬다면, TinyML은 그 반대편 —
**센서·배터리로 돌아가는 MCU(마이크로컨트롤러)**에서 모델을 실행하는 극한 저전력
구간입니다. 출하량 기준으로는 이 구간이 엣지 AI의 최대 볼륨입니다.

### 4.1 MCU의 비중: 출하량이 말해주는 현실

> ABI Research (2026.06): TinyML AI 칩셋 출하량(개인·업무용 기기 제외)은
> **2031년까지 연평균 37%(CAGR)로 성장해 41억 개를 돌파**, 관련 매출은
> **78억 달러 이상**에 이를 것으로 전망

- 구조적으로 **MCU가 이번 10년 전체에 걸쳐 TinyML 시장을 주도**
- NPU는 최고 성장률(90% CAGR)이지만, 절대 볼륨은 여전히 MCU가 압도
- 지역별 엣지 AI 칩셋 출하 CAGR: **유럽 17% · 북미 16% · 아태 18%**
  (아태는 2030년까지 7억 2,100만 개 돌파 전망)
- 배경: 임베디드 AI가 "실험"에서 "실전 확대"로 전환 — **산업 IoT와 원격(far-edge)
  환경**이 성장을 견인
- 의미: **클라우드 왕복 없는 연속 실행 추론**(센서 분석 · 이상 감지 · 음향 인식)이
  수요의 중심 → "연산·메모리·전력을 아끼고 성능을 유지"라는 화두의 최저전력 버전

### 4.2 NPU를 품은 MCU: STM32N6 / Neural-ART 실측

MCU도 이제 "CPU + NPU" 구조로 진화하며 성능 벽이 크게 낮아졌습니다.
ST의 STM32N6x7(Cortex-M55 + Neural-ART Accelerator)이 대표 사례입니다.

| 항목 | 실측값 (ST wiki, STM32Cube.AI 10.0.1 기준) |
|---|---|
| 코어 / NPU | Cortex-M55 (기본 600MHz · 퍼포먼스 800MHz) + **Neural-ART NPU 600 GOPS, 최대 1GHz** |
| 효율 | 약 3 TOPS/W |
| 내장 메모리 | 4.2MB 연속 SRAM (모델 가중치는 외부 플래시에 저장) |
| MobileNet v2 1.0 (224×224) | 가중치 4.13MB · 활성화 2.01MB · **23.2ms · 43 inf/s** · 6mJ |
| YOLOv8n (256×256) | 가중치 2.91MB · 활성화 1.59MB · **35.6ms · 28 inf/s** · 8mJ |
| YAMNet 1024 (64×96) | 가중치 3.41MB · 활성화 0.14MB · **9.9ms · 101 inf/s** · 2.72mJ |
| TinyYOLOv2 (224×224) | 가중치 10.55MB · 활성화 0.38MB · **31.4ms · 32 inf/s** · 10.8mJ |

- 요점: 수십~수백 밀리와트 예산이던 MCU에서도 **YOLOv8이 ~28fps로 실시간 추론** →
  "MCU에서는 안 되는 것"이라는 인식의 전환점
- ST 외에도 Infineon·NXP 등이 NPU 내장 MCU를 포트폴리오에 추가 —
  **"MCU에 NPU 심기"**가 Far-edge AI의 표준 흐름으로 정착
- Neural-ART는 **on-the-fly(실시간) 가중치 압축 해제와 데이터 암호화/복호화**를
  하드웨어로 지원 → 저장 용량은 줄고 실행 성능은 유지되는 하드웨어 레벨 압축 사례

### 4.3 크기를 줄이면서 성능을 유지·향상: TinyML 최신 기법

MCU 구간은 "작게 + 빠르게 + 오래(배터리)"가 생존 조건이라, 앞 문서(압축 기법)가
**가장 실전적으로 적용되는 무대**입니다.

- **INT8 양자화**: STM32 모델줄 int8 예시(MNIST)에서 가중치 20.1KiB로
  float 대비 **약 74.7% 축소**, 활성화 메모리 4KiB 수준 → 플래시·RAM 동시 절감
- **Width(폭) 슬리밍 / 채널 감소**: MobileNet 계열의 폭 승수(0.25/0.35/0.5)처럼
  채널을 줄여 연산량을 최대 75% 이상 감소 — STM32 모델줄에서 사실상 표준
- **KD(지식 증류)·NAS**: 강한 선생 모델·검색된 아키텍처에서 작은 학생 모델로
  지식을 이관해 **"작아도 비슷하거나 더 높은 정확도"**를 달성
- **구조적 가지치기(채널 단위)**: 제거 후에도 NPU·DSP의 연산 포맷과 호환되도록
  구조를 유지하는 가지치기 → MCU에서 실용적인 가지치기의 이유
- **QAT·캘리브레이션 개선**: 양자화로 인한 정확도 손실을 훈련 단계에서 상쇄
- **하드웨어 가중치 압축(NPU 지원)**: Neural-ART처럼 NPU가 가중치를 실시간 압축
  해제 → 저장 공간 감소 + 실행 대역폭 그대로 (크기↓ ≒ 성능 유지)

**핵심 인사이트**: MCU 구간에서 "압축"은 단순 파일 크기 문제가 아니라
**전력 · 배터리 수명 · 실시간성 · 메모리 예산**을 함께 결정하는 통합 설계 문제입니다.
같은 원리는 모바일 SLM의 KV cache 압축(5절)이나 MoE 라우팅(7절)에서 그대로 재등장합니다.

### 4.4 개발 생태계

- **STM32Cube.AI / ST Edge AI Core**: Keras·TensorFlow·ONNX 모델을 자동 분석 ·
  양자화 · NPU 매핑 → C 코드 생성
- **ST Edge AI Developer Cloud**: 온라인 보드팜에서 모델별 추론 시간·메모리 실측
- **STM32 모델줄(stm32ai-modelzoo)**: 최적화된 사전 학습 모델 + 훈련·양자화·벤치마크
  스크립트 자동화
- **TFLite Micro / ONNX Runtime Edge**: 경량 런타임이 MCU 배포의 표준 스택으로 정착
  (MQTT · IEEE 802.15.4 메시 토폴로지에서 저전력 배터리 노드로 운용)

---

## 5. 병목의 이동: 컴퓨트 → 메모리 대역폭

토큰 생성은 본질적으로 **메모리 대역폭 바운드(memory-bound)** 입니다.
전체 가중치가 토큰 하나마다 메모리에서 다시 스트리밍되어야 하기 때문입니다.

```
모바일 디바이스 메모리 대역폭:  약 50~90 GB/s
데이터센터 GPU 대역폭:          약 2~3 TB/s   (30~50배 차이)
```

**함의**
- NPU의 TOPS 수치보다 **실질 대역폭·캐시·KV cache 처리**가 실사용 성능을 결정
- Qualcomm이 "Element Accelerator + 대형 공유 NPU 메모리(+50%)"를 강조하는 이유:
  트랜스포머 연산 장치와 상수 이동 경로를 최적화해 **메모리 왕복을 줄이는 것**
- OSDI 2026 연구(Sereno): 동시 모바일 LLM 추론이 일반 앱의 **버벅임(stutter)을 153% 증가**시킬 수 있음
  → NPU 속도가 아니라 **공유 메모리 대역폭 경쟁**이 실제 체감 성능을 좌우
- KV cache 압축·페이지 관리(Paging)·Low-bit 정밀도가 메모리 부담 해소의 핵심 병기

---

## 6. Agentic AI와 Physical AI

**"Edge AI 4.0 = 행동"** — 로컬 추론이 단순 인식·분류를 넘어
**이해하고 판단하고 실제 세계를 제어**하는 단계로 진화하고 있습니다.

- Qualcomm + Arduino 일러스트(2026.07): 스마트폰 음성 명령 → 로컬 파운데이션
  모델이 로봇 카메라로 객체 인식(지연 ~20ms) → 피킹(pick-and-place) 수열 계산 →
  로봇팔 제어까지 **클라우드 왕복 없이** 수행
- NVIDIA: Amazon Robotics, Boston Dynamics, FANUC, 1X 등이 Jetson Thor 기반
  물리 AI 구축, **Cosmos 3 Edge(4B)** 로 임베디드 기계에 시·공간 이해 제공
- Google: **LiteRT-LM**이 Android·iOS·웹에서 Gemma 4 로컬 실행, Multi-Token
  Prediction으로 디코딩 최대 2.2배 가속
- Arm "에이전트가 유지하는 컨텍스트"를 하드웨어 설계 원칙으로 명문화

**의미**: 로컬 추론을 위한 메모리·지연·전력 망라 최적화는 더 이상 선택이 아니라
에이전트 실행을 위한 **전제 조건**이 되었습니다.

---

## 7. MoE가 모바일의 핵심 전략으로

Mixture-of-Experts(MoE)가 모바일 NPU의 공식 지원 대상으로 부상했습니다.

- Qualcomm Hexagon 신세대: MoE 라우팅을 하드웨어 레벨 지원, **30B 모델 중
  토큰당 ~3B만 활성화**
- MediaTek NPU 1090: MoE LLM 지원, INT4 기준 prefill +51%, 토큰 생성 +55%/와트
- 원리: 전체 파라미터를 메모리에 두되 **라우팅으로 필요한 전문가만 연산·로딩**
  → "모델 크기가 커도 실제 연산·메모리 트래픽은 작게" — 이 문서의 화두인
  **연산/메모리 절약 + 성능 유지**와 정확히 일치

---

## 8. Hybrid Edge-Cloud가 기본 설계로

2026년 결론은 "엣지가 클라우드를 대체한다"가 아니라 **"분업"** 입니다.

```
일상·저지연·민감 작업  → 로컬 (무료·개인정보 보호·오프라인)
복잡 추론·긴 문맥·신규 정보 → 클라우드 (필요할 때만)
라우팅/오케스트레이션 → 태스크 복잡도 판정(복잡도 평가 + 동적 스위칭)
```

- **SLM-first 의식**: 로컬이 처리하는 요청 비율을 높일수록
  서버 비용·대역폭·왕복 지연이 줄어 수익 구조 개선
- **OpenPhone**(ACL 2026): 로컬 GUI 에이전트(3B)가 먼저 작업하고, 실시간
  복잡도 판정으로 어려운 소부분만 클라우드 MLLM으로 승격 → 1단계 실패 OL을 줄여
  클라우드 호출과 비용 대폭 절감
- **규격화**: MCP(Model Context Protocol)·A2A(Agent-to-Agent)가 로컬/클라우드
  워크로드 분산의 표준 인터페이스로 확산

---

## 9. 런타임·소프트웨어 스택

2026년에는 **추론 런타임의 경량화·표준화**가 하드웨어 활성화를 좌우하고 있습니다.

| 런타임/도구 | 특징 |
|---|---|
| ExecuTorch (Meta) | ~50KB 배포 크기, 온디바이스 Python→배포 파이프라인 |
| llama.cpp / GGUF | CPU·GPU 하이브리드 추론, 저비트 포맷 표준 |
| MLX (Apple) | Apple Silicon 최적, <14B 모델에서 llama.cpp보다 20~87% 빠름 |
| LiteRT-LM (Google) | Gemma 4를 Android·iOS·웹에서 로컬 실행 |
| Qualcomm AI Hub | 온디바이스 모델 딜리버리·양자화 자동화 |
| AICore (Google) | Android 로컬 실행·앱별 추론 할당량·백그라운드 제한 |

- 성장 모델: **"NPU가 있어도 런타임이 못 따라오면 무용지물"** — SoC 제조사들이
  소프트웨어 스택까지 묶어 제공하는 이유

---

## 10. 평가·벤치마크의 변화

- TOPS 수치의 불신: 제조사별로 INT8/INT4/희소 연산 기준이 달라 실측 성능과 괴리
- **MLPerf Client v2.0 (2026.08.18)**: 에이전트 AI·이미지 생성이 공식 벤치마크
  카테고리에 처음 포함 (랩톱 로컬 AI 실측)
- MLCompess 서버 측에도 에이전트 추론 트랙 신설(900+ 멀티턴 에이전트 궤적)
- 현장 기준: **"실측 지연시간·메모리·에너지" (ALEM)** 중심 평가가 정착

---

## 11. 결론

- Edge AI는 "모델 압축의 문제"에서 **"온디바이스 시스템 설계의 문제"** 로 진화
- 핵심 축: **SLM 생태계 + NPU/메모리 아키텍처 + MCU·TinyML(저전력 대량 출하) +
  에이전트/물리 AI 워크로드 + Hybrid 분업 + 런타임 표준화**
- 여전히 유효한 결론: 메모리 대역폭·KV cache·MoE 등은 결국
  **"연산/메모리를 아끼고 성능을 유지"** 하는 우리의 원래 화두로 귀결됨
  → 압축 기법(앞 문서)과 시스템 최적화(이 문서)는 별개가 아니라 서로 맞물리는 관계

> **한 줄 요약**: 압축 기법 세대는 뒷심이며, 2026년 하반기는
> **SLM + NPU + 에이전트 + Hybrid 엣지-클라우드**가 만나는
> "엣지 AI 시스템 세대"로 접어들었습니다.

---

### 주요 출처 (2026.06~09 기준)

- ABI Research Press: TinyML AI Chipset Shipments to Top 4.1 Billion by 2031 (2026.06.18)
- ST wiki: AI:STM32Cube.AI model performances (STM32N657 Discovery 측정, STM32Cube.AI 10.0.1)
- ST: STM32N6 시리즈/STM32N657X0 제품 페이지 (Neural-ART 600 GOPS, 3 TOPS/W)
- Wevolver 2026 Edge AI Technology Report (2026.08)
- Dell "The Power of Small: Edge AI Predictions", ABI Research (2026.01~06)
- Android Authority: Qualcomm Snapdragon 8 Elite Gen 6 NPU 소식 (2026.09.10)
- All About Circuits / PR Newswire: MediaTek Dimensity 9600 Pro (2026.09.15~16)
- Arm CSS for Mobile 2 발표 (2026.09.08)
- NeuralWired "On-Device AI in 2026: The Stack Replacing Cloud APIs" (2026.09.08)
- The Brief Script "Edge AI News 2026: Chips, Robots and On-Device Intelligence" (2026.08)
- arXiv: OpenPhone (ACL 2026), Diffusion Language Models for Mobile Edge Agentic AI (2026.09)