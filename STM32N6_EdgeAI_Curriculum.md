# STM32N6 Edge AI 개발자 교육 커리큘럼 — NUCLEO-N657X0-Q 기반

> 대상: **C/STM32 사용 경험이 있는 임베디드 개발자** (AI 이론은 기초부터 시작)
> 형태: 단일 마스터 로드맵 + 모듈별 커리큘럼 (2~4시간/모듈, 총 약 40~60시간 기준)
> 보드: `NUCLEO-N657X0-Q` (STM32N657X0H3Q, Cortex-M55 + Neural-ART NPU)
> 사전 참고: 바탕화면의 `README.md`(양자화 vs 가지치기), `Edge_AI_Recent_Trends.md`(압축 기법 동향),
> `Edge_AI_2026_System_Trends.md` 4절(MCU/TinyML, STM32N6 실측표)

---

## 1. 문서 개요

### 1.1 교육 목적

STM32N6의 **Neural-ART NPU**를 활용해, "ST가 제공하는 모델을 그대로 쓰는 단계"에서
"**자체 모델을 만들어 훈련·양자화·배포(Fitting)하는 단계**"까지 전 과정을 손으로 익힌다.

### 1.2 학습 성과 (이 교육을 마치면)

1. NUCLEO-N657X0-Q 보드를 부팅 모드(dev/flash)에 맞게 구동하고 외부 플래시에 펌웨어를 올릴 수 있다.
2. `stedgeai`(ST Edge AI Core)로 모델을 NPU용 C 코드로 생성하고, 벤치마크(지연·메모리·에너지)를 실측할 수 있다.
3. STM32 Model Zoo의 사전학습 모델(분류·객체검출)을 카메라 없이/카메라로 실시간 배포할 수 있다.
4. PyTorch/Keras로 **자체 모델**을 훈련하고, ONNX QDQ INT8 양자화(+QAT, 가지치기)를 거쳐
   NPU에 배포(fitting)할 수 있다.
5. 카메라·디스플레이·마이크·시계열(IMU/진동) 각 센서 도메인의 실습 프로젝트를 완성할 수 있다.
6. 지연·RAM·플래시·전력 예산을 읽고, 모델/구성이 임계값을 넘으면 최적화 여지를 판단할 수 있다.

### 1.3 전제 조건

| 항목 | 전제 |
|---|---|
| 프로그래밍 | C 중급 이상, STM32CubeIDE 사용 경험 |
| 임베디드 | UART·I2C·SPI·GPIO·인터럽트 이해, 보드 부팅/디버깅 경험 |
| AI | 수학(선형대수 기초 감각) 수준. 신경망·양자화 개념은 본 교육에서 커버 |
| Python | 파이썬 기본 문법, 가상환경(venv) 사용 가능 |
| 하드웨어 | NUCLEO-N657X0-Q + (선택) CSI 카메라, X-NUCLEO-GFX01M2, PDM 마이크, MPU6050/피에조 |

> **주의**: STM32N6는 **내부 플래시가 없다.** 모든 펌웨어는 외부
> xSPI(NOR) 플래시에 프로그래밍하거나 SRAM(dev mode)으로 로드해야 한다.
> "복사-붙여넣기로 끝나는 강좌"가 아니라, 메모리 맵과 부트 모드를 이해해야 하는 과정이다.

---

## 2. 교육 환경 구성

### 2.1 하드웨어 구성

| 구성 | 품목 | 용도 | 난이도 |
|---|---|---|---|
| 필수 | NUCLEO-N657X0-Q + **USB‑C ↔ USB‑C** 케이블 | 전원/ST‑LINK/디버그 (USB‑A→C는 전원 부족 이슈) | - |
| 선택 A | CSI 카메라 모듈: IMX335(MB1854B), STEVAL‑55G1MBI, STEVAL‑66GYMAI1, STEVAL‑1943‑MC1 | 실시간 분류·객체검출 입력 | 중 |
| 선택 B | X‑NUCLEO‑GFX01M2 (확장 디스플레이) | SPI 출력 UI (보드에 LCD 없음) | 중 |
| 선택 C | PDM MEMS 마이크(3‑wire/SAI) | 키워드 검출·음성 분류 | 중상 |
| 선택 D | MPU6050(I2C)·피에조(ADC) | 시계열·진동 분류/이상 감지 | 중상 |

보드 핵심 사양 (NUCLEO‑N657X0‑Q / MB1940)
- 코어: Cortex‑M55(최대 800MHz, TrustZone·FPU) / NPU: **Neural‑ART 600 GOPS, 1GHz, 288 MAC/cycle, ~3 TOPS/W**
- 메모리: **4.2MB 연속 SRAM** (내부 플래시 없음), 외부 xSPI(1×16/1×8 Bbit)로 NOR/PSRAM 확장
- 영상: 2‑lane MIPI CSI‑2(FPC 22‑pin CN6, RPI0 규격), 통합 ISP(3 파이프: bad pixel·demosaic·crop·gamma…), H.264 인코더, JPEG 코덱, Neo‑Chrom 2.5D GPU, LCD‑TFT(최대 XGA)
- 인터페이스: 4×I2C, 6×SPI, 3×I2S, 2×I3C, 5×USART, 2×USB OTG HS, USB‑C PD, 3×FDCAN, 2×SDMMC, Ethernet, 2×ADC(12bit), MDF/ADF 필터
- 보드: Arduino Uno V3/ST Zio, morpho, M.2 Key A, ST‑LINK V3EC(VCOM·대용량 USB·디버그)

> **Nucleo 주의점**: 보드에 외장 RAM이 없어 **대형 입력(예: 320×320 검출)의 일부
> 검증 앱은 제한**된다. 커리큘럼 초반에는 224×224 이하 입력부터 시작한다.
> (ST 공식 벤치마크인 STM32N657‑DK의 실측값은 `Edge_AI_2026_System_Trends.md` 4.2절 참고)

### 2.2 소프트웨어 스택

| 도구 | 역할 | 버전 참고 |
|---|---|---|
| STM32CubeIDE | 임베디드 개발/디버깅 | 1.17.0 이상 |
| STM32CubeProgrammer | 외부 플래시 프로그래밍 | 2.18 이상 |
| ST Edge AI Core (구 X‑CUBE‑AI/STM32Cube.AI) | 모델→NPU C 코드 생성(`stedgeai` CLI), 양자화 | **버전 혼재(2.x~10.x) — 모델줄과 반드시 호환** |
| STM32 Model Zoo (stm32ai‑modelzoo) | 사전학습/최적화 모델 저장소 | - |
| STM32 Model Zoo Services (stm32ai‑modelzoo‑services) | `stm32ai_main.py` + YAML로 훈련·양자화·배포 자동화 | - |
| STM32N6 Getting Started (분류/검출 C 프로젝트) | 배포용 응용 C 코드 (모델줄 서비스의 `c_project_path`) | - |
| Python 3.10+ | PyTorch/Keras/TF, ONNX Runtime, stgui | venv 구성 |
| (선택) ST Edge AI Developer Cloud | 온라인 보드팜 벤치마크 (N6는 `on_cloud` 미지원 → 로컬 설치 권장) | - |

설치 순서 체크리스트
1. STM32CubeIDE → 2. STM32CubeProgrammer → 3. Python(venv: `torch`, `onnxruntime`, `tensorflow` 필요 시)
4. ST Edge AI Core(`stedgeai` 경로 확인: `stedgeai --version`) → 5. Model Zoo + Zoo Services 클론(같은 상위 폴더)
6. Getting Started 패키지 복사 → 7. 보드 전원/드라이버 확인(장치 관리자: 가상 COM port)

### 2.3 부트 모드와 최초 실행

STM32N6는 내부 플래시가 없어 **두 가지 부트 방식**을 쓴다.

```
dev mode(개발):       BOOT0(JP1) 1번 / BOOT1(JP2) 2번  → SRAM에 로드 (전원 끄면 소실)
boot from flash(실전): BOOT0(JP1) 1번 / BOOT1(JP2) 1번  → 외부 xSPI 플래시에서 부팅(영구)
```

M1에서 할 일
- USB‑C to USB‑C로 ST‑LINK 포트 연결 → 드라이버 정상 확인
- STM32CubeIDE에서 빈 프로젝트로 **LED 토글 + UART(VCOM) print** 동작 확인
- 부트 모드를 "flash mode"로 바꾸고 외부 플래시에 펌웨어를 올려 재부팅 후 유지되는지 확인
- (개념) `0x70000000`(XSPI2) 외부 플래시 주소와 STM32CubeProgrammer **External Loader** 개념 이해

---

## 3. 전체 커리큘럼 지도

| 트랙 | 모듈 | 제목 | 난이도 | 소요 | 전제 |
|---|---|---|---|---|---|
| A. 기초 | M1 | 보드 부팅·주변기기·디버깅 | 초 | 3h | - |
| A. 기초 | M2 | NPU 첫 추론 (모델 개념 + 시리얼 벤치) | 초 | 3h | M1 |
| A. 기초 | M3 | stedgeai 분석·생성·보고서 읽기 (메모리/속도 해석) | 초 | 2h | M2 |
| B. Zoo | M4 | 사전학습 이미지 분류 배포 (카메라 없이) | 초중 | 3h | M3 |
| B. Zoo | M5 | 객체 검출 실시간 배포 (카메라) | 중 | 4h | M4, 선택A |
| B. Zoo | M6(선택) | 얼굴/분할/추가 모델 멀티 모델 | 중 | 3h | M5 |
| C. 자체 모델 | M7 | Python 훈련 파이프라인과 TinyML 설계 사고 | 중 | 4h | M4 |
| C. 자체 모델 | M8 | ONNX QDQ 양자화·QAT·가지치기 실습 | 중상 | 4h | M7 |
| C. 자체 모델 | M9 | chain_qd(양자화+배포) 자동화 | 중상 | 2h | M8 |
| C. 자체 모델 | M10 | 커스텀 모델 standalone 배포 (직접 C 통합) | 상 | 4h | M9 |
| D. 응용 | D1 | 카메라(ISP→NPU) 파이프라인 심화 | 중 | 3h | M5 |
| D. 응용 | D2 | 디스플레이(UVCL/SPI+GFX01M2) GUI | 중 | 3h | M5 |
| D. 응용 | D3 | 오디오: 키워드 검출(PDM mic) | 중상 | 4h | M10 |
| D. 응용 | D4 | 시계열: MPU6050/피에조 진동 분류·이상 감지 | 중상 | 4h | M10 |
| D. 응용 | D5 | 통합 프로젝트 (전 센서 수렴) | 상 | 6h | D1~D4 |
| X. 전문 | X1 | 최적화·정밀 벤치마크(지연/RAM/에너지) | 상 | 3h | M10 |

> 제안 일정: 집중 과정 5일(하루 8h) → M1~M8, M9~M10+D3/D4 선택, D5/ X1은 숙제·심화.
> 자기 주도: 주 2모듈씩 8주.

---

## Track A. 펌웨어·NPU 기초

### M1. 보드 부팅 · 주변기기 · 디버깅

**학습 목표**
- 보드 리비전(MB1940)과 MCU(ST32N657X0H3Q)의 리소스 맵 이해
- dev mode vs flash mode 차이를 직접 확인
- GPIO(LED/버튼), UART(VCOM), SysTick/절전을 CubeIDE에서 구성

**실습 단계**
1. USB‑C to USB‑C 연결 → ST‑LINK V3EC 인식, 가상 COM 포트 확인
2. CubeIDE에서 빈 프로젝트 생성(STM32N657XX, TrustZone 비활성으로 시작)
3. 보드 LED(3개) 토글 + 버튼 인터럽트 + UART printf
4. 화면에 각 부트 모드 셋팅을 바꿔가며 동작 확인 (SRAM 로드 → 재부팅 시 소실)

**산출물/검증**
- UART로 1초마다 시각 출력, 버튼으로 LED 토글
- "flash mode에서 재부팅 후에도 동작 유지" 확인

**참고 자료**
- `UM3417` STM32N6 Nucleo‑144 board (MB1940) user manual
- STM32CubeN6 MCU 패키지 예제

---

### M2. NPU 첫 추론 (모델 개념 + 시리얼 벤치)

**학습 목표**
- "신경망 = 레이어별 연산 + 파라미터" 최소 개념 (FC → Conv → ReLU → Softmax 순서)
- 사전학습 INT8 모델 하나를 NPU에서 실행해 지연을 재는 최소 경로를 안다

**실습 단계**
1. Model Zoo에서 `mobilenetv2_a035_128_fft_int8.tflite`(flowers 5클래스) 내려받기
2. `stedgeai`로 NPU C 코드 생성:
   ```
   stedgeai generate -m mobilenetv2_a035_128_fft_int8.tflite \
                     --target stm32n6 --st-neural-art
   ```
3. `st_ai_output/` 산출물 확인: `network.c`, `network.h`, `network_ecblobs.h`,
   `network_atonbuf.xSPI2.raw`(가중치 바이너리), `network_generate_report.txt`
4. Getting Started 분류 프로젝트에 network.c 통합 → flash mode로 부팅
5. AiRunner로 추론 시간 실측:
   ```
   python $STEDGEAI_CORE_DIR/scripts/ai_runner/examples/ai_checker.py -d serial:921600
   ```

**산출물/검증**
- 분류 앱이 사전 정의 클래스(한글 라벨)로 추론 수행
- 추론 시간·RAM ativations을 기록: (참고값, DK 기준) MobileNetV2 1.0@224 = 23.2ms/43fps
- `n6_loader.py`로 로더/검증 앱 빌드 과정 소화

**핵심 개념 카드**
- INT8 정수 추론: 가중치·활성화가 int8이라 NPU 288 MAC/cycle 활용
- **whether**를 "양자화된 사전 4KB~ 수십 ms" 단위로 이해 (양자화 개념은 M8에서 심화)

---

### M3. stedgeai 분석·생성·보고서 읽기

**학습 목표**
- `analyze`/`validate`/`generate`/`quantize` 네 명령 구분
- 보고서에서 **플래시(weight) / RAM(activations) / latency / 에너지**를 읽고 모델링

**실습 단계**
1. `stedgeai analyze -m <모델>` 로 메모리·속도 리포트 읽기
2. 여러 모델(128 vs 224, width 0.35 vs 1.0)을 비교해 "입력 크기의 영향" 정리
3. `network_c_info.json`과 `network_generate_report.txt` 항목 대조
4. (개념) `--inputs-ch-position chlast --input-data-type uint8` 등 I/O 포맷 옵션 확인

**산출물/검증**
- 모델 A/B 비교표 작성 (Flash, RAM, ms, 정확도 희생 여부)
- "이 모델을 이 보드에 넣으면 RAM이 부족한가?"를 스스로 판정

---

## Track B. ST 제공(Zoo) 모델 활용

### M4. 사전학습 이미지 분류 배포 (카메라 없이)

**학습 목표**
- Model Zoo Services의 YAML 기반 자동 배포 흐름(훈련 없이 ``deployment``만) 이해
- UVCL(USB UVC) 또는 시리얼 로 출력해 "PC 화면으로 분류 결과 보기" 구현

**실습 단계**
1. `stm32ai-modelzoo` + `stm32ai-modelzoo-services` 동기화 폴더 구조 이해
2. `README_DEPLOYMENT_STM32N6.md`의 분류 YAML을 따라
   `deployment_n6_config.yaml` 작성 (아래 형식 참고)
3. **NUCLEO 전용 설정**: `board: NUCLEO-N657X0-Q`, `output: "UVCL"` (또는 `"SPI"`)
4. 실행:
   ```
   python stm32ai_main.py --config-path ./config_file_examples/ \
          --config-name deployment_n6_config.yaml
   ```
5. 카메라 없이도 **정적 이미지/시리얼 입력**으로 분류 동작 검증

**산출물/검증**
- flowers(데이지·민들레·장미·해바라기·튤립) 5클래스 라벨 출력 정합
- 모델 바꿔보기(128→224, efficientnet)로 "단순 교체" 경험

**YAML 참고 템플릿 (분류 예)**
```yaml
general:
  project_name: ic_mobilenet
model:
  model_path: ../../stm32ai-modelzoo/image_classification/mobilenetv2/ST_pretrainedmodel_public_dataset/tf_flowers/mobilenetv2_a035_128_fft/mobilenetv2_a035_128_fft_int8.tflite
operation_mode: deployment
dataset:
  dataset_name: tf_flowers
  class_names: [daisy, dandelion, roses, sunflowers, tulips]
preprocessing:
  resizing: { interpolation: bilinear, aspect_ratio: crop }
  color_mode: rgb
tools:
  stedgeai:
    optimization: balanced
    on_cloud: False            # N6는 클라우드 미지원 → 로컬
    path_to_stedgeai: C:/ST/STEdgeAI/<ver>/Utilities/windows/stedgeai.exe
  path_to_cubeIDE: C:/ST/STM32CubeIDE_<ver>/STM32CubeIDE/stm32cubeide.exe
deployment:
  c_project_path: ../application_code/image_classification/STM32N6/
  IDE: GCC
  verbosity: 1
  hardware_setup:
    serie: STM32N6
    board: NUCLEO-N657X0-Q
    output: "UVCL"             # "UVCL"(USB) 또는 "SPI"(GFX01M2)
mlflow: { uri: ./experiments_outputs/mlruns }
hydra:
  run:
    dir: ./experiments_outputs/${now:%Y_%m_%d_%H_%M_%S}
```

---

### M5. 객체 검출 실시간 배포 (카메라)

**학습 목표**
- CSI 카메라(IMX335류)→ISP→NPU→NMS 박스 표시 전체 파이프라인 이해
- YOLOv8n / ST‑YOLO‑Xn 중 선택 기준(속도 vs 정확도) 실측 비교

**실습 단계**
1. 카메라 모듈을 CN6(22‑pin FPC)에 장착, `x-cube-n6-camera-capture`로 UVC 스트림 확인
2. Model Zoo: `st_yoloxn_d033_w025_416_int8.tflite`(person) 또는 `yolov8n` 선택
3. 검출 YAML 작성: `postprocessing` 섹션(confidence/NMS `max_detection_boxes`)
4. 배포 후 실물(사람/사물) 앞에서 박스·confidence 확인
5. `output: "SPI"`로 바꿔 GFX01M2 화면에 표시(디스플레이 보유 시)

**산출물/검증**
- 사람 검출 박스 + 추론 시간 표시 동영상/스크린샷
- 모델 A(작은 입력) vs B(큰 입력) fps·메모리 비교표

**참고 자료**
- `README_DEPLOYMENT_STM32N6.md` (object_detection), Getting Started ObjectDetection
- 참고 실측(DK, nominal): YOLOv8n 256×256 = 35.6ms·28fps, MobileNetV2@224 = 43fps

---

### M6(선택). 얼굴 검출·분할 등 멀티 모델

- `yunetn_320_qdq_int8.onnx`(WIDERFACE 얼굴 검출) 배포
- `deeplabv3_mnv2_a050_s16_asppv2_320_qdq_int8.onnx`(Pascal VOC 의미론적 분할) 배포
- 같은 board에서 모델 교체를 위해 YAML의 `model/model_type`·`postprocessing` 차이 이해

**산출물/검증**: 얼굴 ROI 표시, 분할 마스크 출력, 각각 지연 기록

---

## Track C. 자체(커스텀) 모델 파이프라인

### M7. Python 훈련 파이프라인과 TinyML 설계 사고

**학습 목표**
- "정확도만 높은 모델"이 아니라 "데이터·모델·하드웨어 예산"을 같이 설계하는 습관
- 모델줄의 `training` 모드(YAML 기반 훈련)와 직접 PyTorch 훈련의 차이 이해

**실습 단계**
1. 가상환경 구성(venv): `torch` / `tensorflow` + `onnxruntime` + `onnxslim`
2. 작은 데이터셋(예: 개인 사물 3~5클래스, 또는 wakesenor) 준비·분할(train/val/test)
3. MobileNetV2‑유사 소규모 CNN을 PyTorch로 구성 (width 0.25~0.5 수준으로 시작)
4. 훈련 → val 정확도 목표(예: ≥ 90%) 확인 → ONNX export
5. 모델줄 서비스 `training` 모드(YAML `operation_mode: training`)를 병행 비교

**산출물/검증**
- val 정확도·손실 커브 플롯, exports된 `.onnx` 생성
- "내 모델의 파라미터/연산량(FLOPS)이 MCU 4.2MB SRAM + NPU 예산에 들어가는가"
  를 1차 계산 (예산 계산 워크시트 작성)

**핵심 카드**: TinyML 4대 제약 = RAM(활성화), Flash(가중치), 지연(실시간성), 에너지(배터리)

---

### M8. ONNX QDQ 양자화 · QAT · 가지치기

**학습 목표**
- **PTQ(Post‑Training Quantization)**: ONNX Runtime 정적 양자화, QDQ 포맷, per‑channel INT8
- **QAT(Quantization Aware Training)**: 훈련 중 양자화 손실 상쇄
- **가지치기/폭 슬리밍**: 채널 단위 축소와 정확도 트레이드오프 (README.md 7절 복습)

**실습 단계**
1. ONNX 모델을 `onnxruntime.quantization`으로 QDQ INT8 변환:
   ```python
   from onnxruntime.quantization import (QuantFormat, QuantType,
       StaticQuantConfig, quantize, CalibrationMethod)
   conf = StaticQuantConfig(
       calibration_data_reader=dr,
       quant_format=QuantFormat.QDQ,
       calibrate_method=CalibrationMethod.MinMax,
       optimize_model=True,
       activation_type=QuantType.QInt8,
       weight_type=QuantType.QInt8,
       per_channel=True)
   quantize(infer_model, output_model_path, conf)
   ```
2. 보정(calibration) 데이터셋의 중요성 실험: 대표 데이터 vs 랜덤 데이터 정확도 비교
3. `opset13` 이상 확인, ONNX Simplifier로 전처리
4. QAT(예: PyTorch `torch.quantization`/TensorFlow QDQ simulate)로 PTQ 대비 정확도 회복 확인
5. 구조적(채널) 가지치기로 모델 크기 50% 감소 실험 → 정확도 영향 측정

**산출물/검증**
- float vs INT8 크기비(통상 ~4배), 정확도 델타(보통 <2%), NPU 연산 호환 여부 표
- "손실이 큰 연산(예: 일부 라운드가 민감한 레이어)만 물과 함께 제외" 노하우
  — `nodes_to_exclude` 실험

**참고 자료**
- ST 문서: "Quantization" (stedgeai‑dc), "Deep Quantized Neural Network(DQNN)" (<8bit)
- 모델줄 노트북 `stm32ai_quantize_onnx_benchmark.ipynb`

> **경고**: dynamic(런타임) 양자화는 미지원, per‑tensor는 NPU에서 비효율.
> always per‑channel + QInt8 권장. 정적 양자화만 사용.

---

### M9. chain_qd (양자화 + 배포 자동화)

**학습 목표**
- float(또는 Keras) 모델을 준비만 하면, **양자화→C코드→빌드→플래시**까지 한 번에
  처리하는 `chain_qd` 워크플로를 익힌다

**실습 단계**
1. `chain_qd_n6_config.yaml`에서 model(onnx/tflite), dataset(class_names·calibration) 설정
2. 실행:
   ```
   python stm32ai_main.py --config-path ./config_file_examples/ \
          --config-name chain_qd_n6_config.yaml
   ```
3. 생성 중간 산출물(양자화된 `*_Q.onnx`, network.c 등)을 확인

**산출물/검증**
- "float 모델 1개 → 현장 동작 펌웨어" 1명령 성공
- chain 실패 지점(퀀트 실패·호환 오류) 식별 능력

---

### M10. 커스텀 모델 standalone 배포 (직접 C 통합)

**학습 목표**
- Model Zoo 서비스의 use‑case 가드(pre/post 처리 포함)를 벗어나
  **내 모델을 직접 C 어플리케이션에 통합**하는 수동 경로 이해
- "표준 검출/분류와 다른 I/O"인 모델도 배포 가능해지는 단계

**실습 단계**
1. 내 ONNX QDQ 모델 → `stedgeai generate` 로 `network.c`/`network.h`/`network_ecblobs.h` 생성
2. 큐커 샘플 준비: `network_c_info.json`에서 I/O shape·데이터 타입 확인
3. Getting Started C 프로젝트(또는 audio 계열의 manual 배포 예제)에 통합:
   - `@AI@` 헤더 include, `ai_network_run` 호출, 전위 처리(리사이즈/정규화) 작성
4. `n6_loader.py`로 로더 응용 빌드 → 검증(`validate --mode target`)
5. (선택) 커스텀 모델이므로 데이터셋을 전제로 `validate` 대표 캘리브레이션 넣어 정확도 확인

**산출물/검증**
- "내 데이터 → 내 모델 → 내 보드 추론 결과" end‑to‑end 성립
- 수동 배포 시 필요한 항목(입력 포맷, 레이아웃, 배치, 시리얼 프로토콜) 체크리스트 작성

**참고 자료**
- ST "Getting started – Evaluate a model on STM32N6" (stneuralart_getting_started.html)
- 커뮤니티: "How to deploy a custom PyTorch model on STM32N6570"

---

## Track D. 센서별 응용 프로젝트

### D1. 카메라(ISP→NPU) 파이프라인 심화

**학습 목표**
- BISP 3‑파이프 구성(bad pixel·decimation·demosaic·crop·downsize·gamma·ROI) 이해
- "ISP 결과를 DMA로 NPU에 직접 공급"하는 흐름을 다이어그램으로 설명

**실습 단계**
1. `x-cube-n6-camera-capture`로 RAW→YU b** 라인 추적(JUVC 스트림)
2. `crop/downsize/ROI` 설정을 바꾸며 NPU 입력 크기에 맞는 관심영역만 내리기
3. 입력 해상도를 192→256→320으로 바꾸며 추론 지연과 정확도 트레이드오프 측정

**검증**: 30fps 근접 파이프라인, ROI 기반 고속 추론 데모

---

### D2. 디스플레이(UVCL/SPI + GFX01M2) GUI

**학습 목표**
- Nucleo(보드 내 LCD 없음)의 두 출력 경로 구분: **UVCL(USB UVC) vs SPI(GFX01M2)**
- JPEG 코덱 + Neo‑Chrom GPU를 이용해 빠른 화면 갱신

**실습 단계**
1. UVCL: 카메라 프레임 + 분류/검출 오버레이를 PC 웹캠으로 표시
2. SPI: `output: "SPI"` + X‑NUCLEO‑GFX01M2로 온보드 디스플레이 출력
3. 분류 라벨·박스·fps를 GUI 요소로 렌더링

**검증**: 실시간 오버레이 GUI 동작, 갱신률 기록

---

### D3. 오디오: 키워드 검출 (PDM 마이크)

**학습 목표**
- PDM → PDM‑to‑PCM 변환(DFSDM/SAI), 1~2초 프레임화, MFCC/멜 스펙트럼 입력 전처리 이해
- 키워드 검출(KWS) 모델 구조(소형 CNN, 순환 계층은 선택)와 NPU 배포 제약 확인

**실습 단계**
1. PDM 마이크(STEVAL‑MIC1류)를 SAI/I2S에 연결, 오디오 캡처 확인(MDM 벤치)
2. Model Zoo audio(KWS) 모델 또는 직접 훈련한 1D‑CNN을 M10 경로로 배포
3. 마이크에 "키워드"를 불러 활성화 확률·시계열 표시
4. (확장) 음성 명령 2~5개 분류

**검증**: 실시간 키워드 트리거 데모 (참고 실측: YAMNet 64×96 = 9.9ms/101 inf/s, DK)

**참고 자료**
- STM32 Model Zoo `audio/` use case, Getting Started `audio/STM32N6` manual deployment

---

### D4. 시계열: MPU6050 / 피에조 진동 분류·이상 감지

**학습 목표**
- 시계열 신호 → 윈도우 프레이밍(청크) → 특징(FFT/통계 또는 raw) → 1D‑CNN/오토인코더 설계
- **이상 감지(anomaly detection)** 를 지도 분류 대신 비지도(오토인코더 재구성 오차)로 구현

**실습 단계**
1. 센서 연결: MPU6050(I2C, 주의: 카메라가 I2C2 사용 → I2C1/3/4 사용), 피에조는 ADC 입력
2. 샘플링 루프 작성(예: 1kHz, 256~512 샘플 윈도우), UART/serial로 데이터셋 수집
   - 정상 상태(대기), 고장 상태(진동 패턴)로 클래스 수집
3. PC에서 1D‑CNN 분류모델 훈련 → INT8 QDQ → M10 경로로 배포
4. (비지도 확장) 시계열 오토인코더를 훈련해 "정상만 학습 → 이탈 탐지"
5. 실시간 예측을 LED/UART로 표시, 피에조 타격 시 "검출" 데모

**산출물/검증**
- 실물 데모: 진동 패턴(정상/불량) 구분 정확도 측정
- 재구성 오차 임계값 튜닝으로 false alarm/miss tradeoff 리포트

**참고 자료**
- ST `vibration monitoring`/(범위 확대) FFT‑기반 특징, N6 MDF/ADF 필터 활용
- TinyML 시계열: 1D‑CNN · TCN · (경량) RNN 비교 리포트

---

### D5. 통합 프로젝트 (선택형 종합)

- 주제 예:
  - "스마트 사물 분류기": 카메라로 물건 분류 + LCD 라벨 + UART 로그
  - "설비 감시 센서 노드": 피에조/IMU 이상 감지 발생 시 사진 스냅 + 경고 (카메라·시계열 융합)
  - "음성+동작 컨트롤러": 키워드 + IMU 제스처로 LED 3색 제어
- 요구사항: 자체 데이터셋 훈련(C) + 최소 2개 센서(D) + 벤치마크 보고서(X1 양식)

**평가 기준**: 기능 완성도 / 지연·메모리 예산 리포트 / 발표(5분) / 코드 리뷰

---

## X. 전문: 최적화·정밀 벤치마크

### X1. 지연 · RAM · 에너지 측정

- `stedgeai`/AiRunner로 **지연(ms)** 실측, `network_c_info.json`으로 **RAM** 해석
- external flash read(가중치 스트리밍)가 병목인지 NPU 연산 병목인지 진단
- `optimization: balanced|time|ram` 3모드 비교 (M10 이후)
- (고급) epoch controller(`--O3-ec`), DQNN(<8bit), on‑the‑fly 가중치 압축(Neural‑ART) 이해
- ISR 과 부팅 런타임 최소화(CPU/NPU 비동기, `LL_ATON_RT_ASYNC`)

**산출물**: 보고서 양식(모델/입력/Flash/RAM/ms/fps/정확도/에너지) 완성

---

## 9. 평가·진도 체크리스트

| 모듈 | 통과 기준(pass criteria) |
|---|---|
| M1 | flash mode 재부팅 후 펌웨어 유지, UART 로그 정상 |
| M2 | NPU 추론 1회 성공, 지연 실측 기록 |
| M3 | 비교표 1건, "RAM 부족 판정" 워크시트 |
| M4 | flowers 라벨 출력, UVCL/SPI 중 1개 |
| M5 | 실물 박스+confidence, fps 기록 |
| M7 | val≥90%, ONNX export, 예산 계산 |
| M8 | INT8 크기 3~4배 절감, 정확도 델타<2% |
| M9 | 1명령 chain 성공 |
| M10 | 커스텀 모델 end‑to‑end 성립 |
| D1~D4 | 각 데모 + 요약 보고서 |
| D5(심화) | 통합 데모 + 벤치 보고서 + 발표 |

---

## 10. 트러블슈팅

| 증상 | 원인/해결 |
|---|---|
| 보드 인식 안 됨 | USB‑A→C는 전력 부족 가능 → **USB‑C to USB‑C** 사용 |
| 재부팅 후 프로그램 소실 | dev mode로 SRAM 로드 상태 → flash mode로 바꾸고 외부 플래시 프로그래밍 |
| CubeProgrammer 외부 플래시 실패 | **External Loader** 체크, 시작 주소(0x7000_0000 계열) 확인 |
| 모델 생성 오류 | stedgeai ↔ Model Zoo **버전 불일치**(v3.0 breaking change 유의). README "What's new"로 매칭 |
| link/RAM 부족 | Nucleo는 외장 RAM 없음 → 입력 축소/width 축소, `optimization: ram` |
| UVC 표시 안 됨 | USB1(CN8) 별도 케이블, OTP xSPI 200MHz fuse 설정 확인 |
| 양자화 정확도 급락 | calibration 대표 데이터 누락, per‑tensor 사용, QAT 미적용 |
| cloud 배포 불가 | N6는 `on_cloud: False`(로컬 설치 필수) |
| 시리얼 통신 안 됨 | AiRunner/validate는 921600 baud, `-d serial:<포트>` 지정 |

---

## 11. 부록

### A. stedgeai CLI 레퍼런스 (요약)

```
stedgeai analyze  -m <model>            # 메모리·속도 리포트
stedgeai generate -m <model> --target stm32n6 --st-neural-art [<profile>]
                   [--input-data-type uint8] [--inputs-ch-position chlast]
                   [--enable-epoch-controller]   # → network_ecblobs.h
stedgeai validate -m <model> --target stm32n6 --mode target -d serial:921600
stedgeai quantize ...                    # <8bit DQNN 등 (ST Edge AI Core)
```

### B. Model Zoo Services operation_mode 목록

| mode | 용도 |
|---|---|
| `training` | 데이터셋→모델 훈련 (YAML 구성) |
| `evaluation` | float/INT8 모델 정확도 측정 |
| `quantization` | ONNX/TF→INT8 변환 + 캘리브레이션 |
| `benchmarking` | 플래시/RAM/ms 시뮬레이션 |
| `deployment` | C코드 생성+빌드+플래시 |
| `chain_qd` | quantize→deploy 일괄 (M9) |

### C. 폴더 구조 (권장 참조)

```
stm32ai-modelzoo/                 # 사전학습/최적화 모델
  image_classification/ object_detection/ semantic_segmentation/
  face_detection/ audio/
stm32ai-modelzoo-services/        # 자동화 + C 응용
  src/stm32ai_main.py
  config_file_examples/*_n6_config.yaml
  application_code/{image_classification,object_detection,audio}/STM32N6/
```

### D. 주요 자료 링크

- 보드: NUCLEO‑N657X0‑Q 제품 페이지, UM3417(사용자 매뉴얼)
- 툴: ST Edge AI Core, STM32CubeIDE, STM32CubeProgrammer, STM32CubeN6 패키지
- 저장소: `stm32ai-modelzoo`, `stm32ai-modelzoo-services`
- 문서: ST wiki "AI:STM32Cube.AI model performances"(벤치 실측), stedgeai‑cs 문서(quantization, getting started)
- 예제: Getting Started ImageClassification/ObjectDetection, `x-cube-n6-camera-capture`
- 동향 보충: 바탕화면 `README.md`, `Edge_AI_Recent_Trends.md`, `Edge_AI_2026_System_Trends.md` 4절

### E. 용어 초기 맵

INT8 QDQ · PTQ/QAT · NPU 288 MAC/cycle · Neural‑ART · XSPI · UVCL · dev/flash mode ·
chain_qd · epoch controller · MDF/ADF · DQNN · per‑channel

---

> 이력: v1.0 초안 (2026‑09‑17). 모듈별 소요시간·순서는 교육 여건에 따라 조정 가능.
> 각 모듈의 실측 수치는 최신 도구 버전에서 다시 검증할 것을 권장.