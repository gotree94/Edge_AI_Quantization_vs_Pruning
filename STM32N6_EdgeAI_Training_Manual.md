# STM32N6 Edge AI 교육 진행 매뉴얼 (NUCLEO-N657X0-Q)

> 상위 문서: `STM32N6_EdgeAI_Curriculum.md` (로드맵/목차 참조)
> 대상: 임베디드 경험 개발자 / 형태: 강사·학습자 공용 실습 진행서
> 본 매뉴얼은 **설치 → 상세 설정 → 모듈별 교육 진행**을 순서대로 세세히 기술한다.
> 모든 경로·버전은 설치 시점에 다시 확인하되, 표기된 값은 2026‑09 기준 공식 문서 기반.

---

## 1. 매뉴얼 개요

### 1.1 구성과 사용법

```
Part 1 (2장)  프로그램 설치        →  설치 전 진단부터 스모크 테스트까지
Part 2 (3장)  상세 설정            →  부트/외부플래시/stedgeai/YAML/NUCLEO 특화
Part 3 (4~8장) 모듈별 교육 진행     →  M1~M10, D1~D5, X1 각각의 진행 대본
Part 4 (9~11장) 운영·트러블슈팅·부록
```

- 각 모듈은 동일한 6단계 틀: **목표 → 준비물 → 상세 절차 → 예상 결과 → 합격 기준 → 강사 팁**
- 강사는 Part 1/2를 사전에 1회 완주하고, 본과정에서는 모듈별 "상세 절차"만 인쇄해 사용한다.
- 로드맵 전체 지도·시간 배분은 상위 문서 3장을 따른다.

### 1.2 일정안 (본 매뉴얼 기준 추천)

| 운영 | 일정 | 모듈 배분 |
|---|---|---|
| 집중 5일 | 08h × 5 | 1일 M1~M2, 2일 M3~M4, 3일 M5+M8, 4일 M7/M9/M10, 5일 D3~D5+X1 |
| 주말형 8주 | 주 1회 4~6h | 주차별: M1→M2→M3→M4→M5→M7/M8→M9/M10→D 트랙+발표 |

### 1.3 교육 시작 전 진단 (전제 확인)

- [ ] STMicroelectronics(www.st.com) 무료 계정 — 모든 툴 다운로드에 필요
- [ ] PC: Windows 10 64bit 이상, 디스크 60GB+ 여유, RAM 16GB+ 권장
- [ ] 네트워크: my.st.com(다운로드), github.com(모델줄) 접근 가능
- [ ] 하드웨어: NUCLEO‑N657X0‑Q + USB‑C to USB‑C 케이블 (전원·ST‑LINK 겸용)
- [ ] 서명/<버전> 개념 안내된 이 문서의 Part 2 사전 숙지

---

## 2. 프로그램 설치 (Part 1)

### 2.0 설치 순서 총괄표

| 순서 | 항목 | 용도 | 필수? |
|---|---|---|---|
| S1 | STM32CubeIDE 1.17+ | 펌웨어 개발·디버깅 | 필수 |
| S2 | STM32CubeProgrammer 2.18+ | 외부 플래시 프로그래밍/서명 | 필수 |
| S3 | STM32Cube AI Studio + ST Edge AI Core | 모델→NPU C코드(그래픽 + `stedgeai` CLI) | 필수 |
| S4 | (선택) STM32CubeMX | 주변기기 구성·코드 생성 | 권장 |
| S5 | Python 3.10+ + venv | 훈련·양자화 스크립트 | 필수(Track C) |
| S6 | Git 클론(Model Zoo/Services/GettingStarted/카메라) | 사전학습 모델·배포 코드 | 필수(Track B~) |
| S7 | 보드 첫 연결·드라이버 확인 | 모든 실습의 전제 | 필수 |
| S8 | 스모크 테스트(아래 2.8) | 설치 검증 | 필수 |

### 2.1 S1. STM32CubeIDE 설치

1. my.st.com → STM32CubeIDE 다운로드(`STM32CubeIDE‑<ver>-win64.zip` 형태)
2. 압축 해제 → `SetupSTM32CubeIDE-<ver>.exe` 실행 → 기본 경로 권장: `C:\ST\STM32CubeIDE_<ver>\`
3. 구성 요소: 기본 선택 유지(STM32CubeMX 통합 포함 여부 체크 → 이후 권장)
4. 설치 후 첫 실행: **workspace** 경로 지정(예: `D:\n6edu\ws`) → "STM32N657X0H3Q" 지원 패키지 설치 확인
   (`Window > Preferences > … > STM32CubeIDE` 또는 프로젝트 생성 시 자동 다운로드)
5. ST‑LINK 드라이버: ST‑LINK V3EC는 일반적으로 자동 인식. 인식 안 되면
   `STM32CubeProgrammer` 설치 시 포함되는 드라이버 재설치.

### 2.2 S2. STM32CubeProgrammer 설치

1. my.st.com → STM32CubeProgrammer 다운로드(2.18+)
2. 기본 경로: `C:\Program Files\STMicroelectronics\STM32Cube\STM32CubeProgrammer\`
3. 핵심 구성 요소 확인:
   - GUI: `STM32CubeProgrammer.exe`
   - CLI: `bin\STM32_Programmer_CLI.exe`
   - 외부 로더: `bin\ExternalLoader\` 아래 **`MX25UM51245G_STM32N6570-NUCLEO.stldr`** 존재 확인
   - 서명 도구: `bin\STM32_SigningTool_CLI.exe` (2.21+는 `-align` 인자 필요)
4. PATH에 CLI 추가(선택): `…\STM32CubeProgrammer\bin` 을 시스템 PATH에 등록

### 2.3 S3. STM32Cube AI Studio + ST Edge AI Core

> 최신 구성 기준: 그래픽 프런트엔드는 **STM32Cube AI Studio**(1.2.0, 2026‑02), 핵심 엔진은
> **ST Edge AI Core**(CLI `stedgeai`). `X‑CUBE‑AI` 플러그인은 신규 개발 비권장(NRND)이며
> ST Edge AI Core 3.0.0부터 미지원 — 본 교육은 AI Studio + Edge AI Core를 기준으로 한다.

1. my.st.com → "STM32Cube AI Studio" 다운로드(Windows)
   - **v1.2.0부터 ST Edge AI Core 설치기가 AI Studio 설치기에 포함** → 별도 다운로드 불필요
2. 설치 경로(기본 예): `C:\ST\STEdgeAI\<ver>\`
   ```
   C:\ST\STEdgeAI\<ver>\Utilities\windows\stedgeai.exe   ← CLI 위치
   C:\ST\STEdgeAI\<ver>\scripts\N6_scripts\n6_loader.py  ← N6 로더/검증 스크립트
   ```
3. `stedgeai`를 PATH 또는 doskey로 등록:
   ```
   :: 관리자 CMD
   setx PATH "%PATH%;C:\ST\STEdgeAI\<ver>\Utilities\windows"
   ```
   (비관리 사용자는 `doskey stedgeai="C:\ST\STEdgeAI\<ver>\Utilities\windows\stedgeai.exe" $*`)
4. 버전 확인:
   ```
   stedgeai --version
   ST Edge AI Core v2.0.0-A1
       STM32CubeAI 10.0.0
       ...
   ```
   > **버전 규칙**: 모델줄(Model Zoo) 생성물은 특정 툴 버전과 호환된다.
   > ST Edge AI Core 3.0/STM32Cube.AI 11 등 **버전 불일치 시 C코드 생성 오류** 발생.
   > 각 실습 전에 Model Zoo README의 "What's new in release"에서 요구 버전 확인.
5. AI Studio 첫 실행 → 메뉴에서 다음 실행파일 경로를 등록(설치되지 않은 것은 S4도 진행 후 등록):
   - STM32CubeMX: `stm32cubemx.exe`
   - STM32CubeProgrammer: `STM32_Programmer_CLI.exe`
   - STM32CubeIDE: `stm32cubeidec.exe`
   - ST Edge AI Core: `stedgeai.exe`

### 2.4 S4(선택). STM32CubeMX 설치

- 기본 경로: `C:\ST\STM32CubeMX\` (설치기 `STM32CubeMX-win-<ver>.exe`)
- 본 교육에서 모델줄 자동 배포를 쓰면 필수는 아니지만, D2(디스플레이)·D4(시계열)의
  센서/주변기기 커스텀과 FSBL 수동 구축에서 사용할 수 있어 권장.

### 2.5 S5. Python 가상환경 구성

```
py -3.10 -m venv C:\ST\n6env
C:\ST\n6env\Scripts\activate
python -m pip install --upgrade pip
pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu   :: CPU로 충분
pip install onnx onnxruntime onnxslim tensorflow 2>&1                            :: TF는 선택
pip install numpy matplotlib scikit-learn pillow
python -c "import torch, onnx, onnxruntime; print(torch.__version__, onnx.__version__, onnxruntime.__version__)"
```

> 강의실 대비: 모듈 M7~M8 소요 시간을 줄이려면 **사전에 이 환경을 스냅샷/배포**한다.

### 2.6 S6. Git 클론 (모델줄·배포 코드)

같은 상위 폴더(예: `C:\ST\`)에 클론:

```
cd C:\ST
git clone https://github.com/STMicroelectronics/stm32ai-modelzoo.git --depth 1
git clone https://github.com/STMicroelectronics/stm32ai-modelzoo-services.git --depth 1
git clone https://github.com/STMicroelectronics/STM32N6-GettingStarted-ImageClassification.git
git clone https://github.com/STMicroelectronics/STM32N6-GettingStarted-ObjectDetection.git
git clone https://github.com/STMicroelectronics/x-cube-n6-camera-capture.git   :: D1용
```

- 모델줄 하위 구조(분류 예):
  ```
  stm32ai-modelzoo/image_classification/mobilenetv2/ST_pretrainedmodel_public_dataset/
      tf_flowers/mobilenetv2_a035_128_fft/mobilenetv2_a035_128_fft_int8.tflite
  ```
- Model Zoo Services 하위 구조:
  ```
  stm32ai-modelzoo-services/src/stm32ai_main.py
  stm32ai-modelzoo-services/src/config_file_examples/*_n6_config.yaml
  stm32ai-modelzoo-services/application_code/{image_classification,object_detection,audio}/STM32N6/
  ```
- **N6 배포 앱 = Getting Started 독립 프로젝트**. 모델줄 서비스는 이를 `c_project_path`로 참조한다.
  (일부 환경에서는 Getting Started 폴더를 모델줄 서비스 `application_code/` 아래로 복사하기도 한다)

### 2.7 S7. 보드 첫 연결 (드라이버/부트 확인)

1. **점퍼 초기 상태 확인**(공장 기본 = flash boot):
   - BOOT0(`JP1`) = 1‑2 / BOOT1(`JP2`) = 1‑2  (둘 다 실크 인쇄된 1번)
2. USB‑C to USB‑C로 **ST‑LINK 포트(CN10)** 연결 → 보드 전원 LED, COM 포트 인식
   (가상 COM: 장치 관리자에서 `ST‑Link VCP`)
3. STM32CubeProgrammer 연결 테스트(dev 모드로 우선):
   - 부트 점퍼를 **dev 모드**(아래 3.2)로 바꿔 놓은 뒤:
   ```
   "%ProgramFiles%\STMicroelectronics\STM32Cube\STM32CubeProgrammer\bin\STM32_Programmer_CLI.exe" -c port=swd mode=HOTPLUG
   ```
4. 확인 후 점퍼는 다시 flash boot(1‑2/1‑2)로 원복.

### 2.8 S8. 설치 스모크 테스트 (합격 기준)

| # | 테스트 | 명령/동작 | 기준 |
|---|---|---|---|
| 1 | IDE 실행 | STM32CubeIDE 부팅 | 정상 |
| 2 | 프로그래머 연결 | 위 S7‑3 명령 | `Device ID` 출력 |
| 3 | stedgeai | `stedgeai --version` | 버전 표시 |
| 4 | 외부 로더 | `bin\ExternalLoader` 목록 | `MX25UM51245G_STM32N6570-NUCLEO.stldr` 존재 |
| 5 | Python | import 3종 | 오류 없음 |
| 6 | 모델줄 | 분류 모델 tflite 존재 확인 | 파일 존재 |
| 7 | 사인 도구 | `STM32_SigningTool_CLI.exe --help` | 도움말 표시 |

**전체 통과 후에만 Part 2로.** 실패 항목은 10장 트러블슈팅을 참고.

---

## 3. 보드·툴 상세 설정 (Part 2)

### 3.1 보드 리소스 맵 (NUCLEO‑N657X0‑Q / MB1940)

| 구분 | 항목 |
|---|---|
| MCU | STM32N657X0H3Q: Cortex‑M55 800MHz, Neural‑ART NPU 600GOPS/1GHz, Neo‑Chrom GPU, JPEG/H.264, **4.2MB SRAM, 내부 플래시 없음** |
| 점퍼 | JP1(BOOT0), JP2(BOOT1), JP3(STLK_RST, 통상 OFF) |
| USB | CN10(ST‑LINK V3EC: 전원+VCOM+디버그), CN8(USB‑C, UVC/직렬 부트 겸용) |
| 카메라 | CN6 22‑pin FPC(ZIF), CSI‑2 2‑lane, I2C2 제어 버스(IMX335류) |
| 확장 | Arduino Uno V3/ST Zio, morpho, M.2 Key A, Ethernet RJ45, FDCAN 해더 |
| 표시 | 보드 자체 LCD 없음 → UVCL(USB 가상 디스플레이) 또는 SPI(GFX01M2) · LCD‑TFT 최대 XGA |

### 3.2 부트 모드 상세

RM0486(STM32N657/647) + UM3417 기준. **BOOT1 우선순위**: BOOT1=1이면 dev boot.

| 모드 | BOOT0 `JP1` | BOOT1 `JP2` | 동작 | 사용 시점 |
|---|---|---|---|---|
| **Flash boot**(초기값) | 1‑2 (0) | 1‑2 (0) | BootROM이 외부 NOR FSBL을 SRAM으로 복사해 부팅 | 실전·펌웨어 유지 |
| **Dev(개발) boot** | 1‑2 (0) | **2‑3 (1)** | ST‑LINK 디버그/CubeProgrammer 외부 메모리 접근 허용 | 프로그래밍·디버깅 |
| **Serial boot** | **2‑3 (1)** | 1‑2 (0) | USB/UART ROM 부트로더(펌웨어 없을 때 리커버리) | 위기 상황 |

> PCB 표기: JP1/JP2의 **실크 인쇄된 1번(1‑2) = 값 0**, 인쇄 안 된 2번(2‑3) = 값 1.
> "BOOT1 스위치 position 2‑3" = dev 모드. 전압 변경 시 **전원 제거 후 점퍼 조작**.

**전환 절차(교과 절차)**
1. USB 케이블 제거
2. 점퍼 위치 변경(위 표)
3. USB 재연결 또는 보드 Power key/리셋
4. (검증) 동작 확인 후 실전 모드로 되돌리고 리셋

### 3.3 외부 플래시 프로그래밍 (FSBL + Appli + Network)

STM32N6 부팅 3요소: **FSBL(제1 부트로더) → Appli(응용, 0x7010_0000) → Network 가중치(0x7038_0000)**.
BootROM은 외부 NOR(0x7000_0000)에서 FSBL을 SRAM2로 복사 → FSBL이 xSPI를 메모리 맵으로 열어 Appli 실행.

#### 주소표(NUCLEO‑N657X0‑Q, NoR `MX25UM51245G` 외장)

| 내용 | 주소 | 비고 |
|---|---|---|
| FSBL (signed) | `0x7000_0000` | 서명 필수, `ai_fsbl.hex` |
| Appli (signed) | `0x7010_0000` | `-t ssbl` 서명 |
| Network weights | `0x7038_0000` | `network_atonbuf.xSPI2.bin` |

#### 외부 로더로 프로그래밍 (GUI)

1. BOOT를 **dev 모드**(JP1 1‑2, JP2 2‑3)로 설정 → USB 케이블 연결
2. STM32CubeProgrammer 실행 → 좌하단 **External Loader** 아이콘 →
   `MX25UM51245G_STM32N6570-NUCLEO` 선택
3. `Erasing & Programming` 탭 → 파일·주소 입력(위 표) → Verify 체크 → Start
4. 완료 후 전원 제거 → **flash boot**(1‑2/1‑2)로 → 재연결/리셋

#### CLI(Getting Started `stmaic_*.conf`의 명령과 동일 패턴)

```
"…\STM32_Programmer_CLI.exe" -c port=swd mode=HOTPLUG -hardRst -w <FSBL>.hex
"…\STM32_Programmer_CLI.exe" -c port=swd mode=HOTPLUG -hardRst -w <Appli>_signed.bin 0x70100000
"…\STM32_Programmer_CLI.exe" -c port=swd mode=HOTPLUG -hardRst -w network_atonbuf.xSPI2.bin 0x70380000
```

#### 서명(Signing) — Appli/FSBL 모두 필수

```
:: CubeProgrammer 2.21+는 -align 필수
"…\bin\STM32_SigningTool_CLI.exe" -bin <Appli>.bin -nk -of 0x80000000 -t ssbl -hv 2.3 -o <Appli>_signed.bin -align
"…\bin\STM32_SigningTool_CLI.exe" -bin <FSBL>.bin   -nk -of 0x80000000 -t fsbl -hv 2.3 -o <FSBL>_signed.bin -align
```

> **Model Zoo Services는 이 3단계(FSBL→Appli→weights)를 YAML로 자동 수행**한다(M4 이후).
> M1~M3에서는 Getting Started 프로젝트 빌드 설정의 `post‑build`/`.conf` 결과물로 흐름을 본다.

### 3.4 stedgeai CLI 핵심

| 명령 | 역할 | 예 |
|---|---|---|
| `analyze` | 비실행 리포트(Flash/RAM/Latency) | `stedgeai analyze -m <m>.tflite --target stm32n6` |
| `generate` | NPU C코드 생성 | `stedgeai generate -m <m>.tflite --target stm32n6 --st-neural-art` |
| `validate` | 타깃 정확도/지연 검증(시리얼) | `stedgeai validate -m <m>.tflite --target stm32n6 --mode target -d serial:921600` |
| `quantize` | 저비트/DQNN 등 고급 양자화 | (M8 심화) |

주요 옵션(N6)
- `--st-neural-art [<profile>@<user.json>]` : Neural‑ART 컴파일 프로파일
  - Mode z zoo가 쓰는 파일: `user_neuralart_NUCLEO-N657X0-Q.json` + `stm32n6-app2_NUCLEO-N657X0-Q.mpool`
- `--input-data-type uint8 --inputs-ch-position chlast` : 카메라(ISP) 파이프라인의 NHWC/uint8 맞추기
- `--output-data-type float32` : 후처리 편의용 float 출력
- `--enable-epoch-controller` : epoch 컨트롤 모드 → `network_ecblobs.h` 생성(논 npm 컨트롤)

`generate` 산출물(`st_ai_output/`)
```
network.c / network.h / network_ecblobs.h
network_atonbuf.xSPI2.raw   ← 가중치 원본(xSPI2 위치용)
network_generate_report.txt / network_c_info.json ← 보고·기계 판독용
```

### 3.5 Model Zoo Services YAML 상세

`stm32ai_main.py --config-path <dir> --config-name <file>` 실행 시 사용. 필수 5대 섹션:

```yaml
general:
  project_name: <문자>            # 실험 폴더 이름
model:
  model_type: <uc별>              # 분류:mobilenetv2 / 검출:st_yoloxn|yolov8n … 
  model_path: <상대 경로 모델>      # .tflite or .onnx(QDQ)
operation_mode: <mode>           # training|evaluation|quantization|deployment|chain_qd
dataset:
  dataset_name: <coco|tf_flowers …>
  class_names: [<라벨 리스트>]
preprocessing:
  resizing:  { interpolation: bilinear, aspect_ratio: crop|full_screen|fit }
  color_mode: rgb|bgr
postprocessing:                  # 검출 계열만
  confidence_thresh: 0.5
  NMS_thresh: 0.5
  max_detection_boxes: 10
tools:
  stedgeai:
    optimization: balanced   # balanced|time|ram
    on_cloud: False          # N6는 클라우드 불가 → 반드시 False
    path_to_stedgeai: C:/ST/STEdgeAI/<ver>/Utilities/windows/stedgeai.exe
  path_to_cubeIDE: C:/ST/STM32CubeIDE_<ver>/STM32CubeIDE/stm32cubeide.exe
deployment:
  c_project_path: ../application_code/<uc>/STM32N6/
  IDE: GCC                    # N6 배포는 GCC만 지원
  verbosity: 1
  hardware_setup:
    serie: STM32N6
    board: NUCLEO-N657X0-Q     # 또는 STM32N6570-DK
    output: "UVCL"             # UVCL(USB 가상 디스플레이) | SPI(GFX01M2)
mlflow: { uri: ./experiments_outputs/mlruns }
hydra:
  run: { dir: ./experiments_outputs/${now:%Y_%m_%d_%H_%M_%S} }
```

- 실행 위치: `stm32ai-modelzoo-services/src/` 에서
  `python stm32ai_main.py --config-path ../config_file_examples/ --config-name <파일명>`
- 로그: `[INFO] : Deployment complete.` 최종 출력되면 성공
- **주의**: `on_cloud: True`로 두면 N6에서는 실패. `efficientnetv2s_384_...` 등 일부 모델은 미지원.

### 3.6 NUCLEO 특화 설정

1. **출력 인터페이스**
   - `UVCL`: PC를 모니터로 사용(웹캠으로 인식), 추가 케이블 1개(CN8→PC) 필요
   - `SPI`: `X-NUCLEO-GFX01M2` 실드 필요, 보드에 LCD가 없으므로 실물 화면용
2. **외부 RAM 없음**: 4.2MB SRAM만 사용. 입력 224×224 이하를 기본으로 하고,
   RAM 부족 시 `optimization: ram` 또는 입력/width 축소.
3. **`.conf` 파일**(Getting Started): `stmaic_NUCLEO-N657X0-Q.conf` 에 UVCL/SPI 두 빌드 설정과
   외부 로더·주소·서명 명령이 정의되어 있다. 배포 프록시로 참조.
4. **OTP fuse**: xSPI 200MHz 고속용 `VDDIO3_HSLV=1` 등은 일부 고속 카메라/플래시에서 필요.
   OTP는 1회성 → 실습 과정에서는 주의 문구만 안내하고 기본 설정으로 진행.
5. **SW 버전 민감성**: N6는 "매우 특수한 타깃". 모델줄 생성물은 CubeIDE 1.17.x + 대응 ST Edge AI
   버전으로 검증되어 있다(일부 2.0.x 조합에서 문제 보고). 모듈별로 README 버전 고정 안내.

---

## 4. 교육 진행 — Track A: 펌웨어·NPU 기초

### M1. 보드 부팅 · 주변기기 · 디버깅 (3h)

**목표**: 부트 모드 전환/외부 플래시 개념, GPIO·UART·버튼 인터럽트, CubeProgrammer 연결.

**준비물**: 보드+케이블, CubeIDE, CubeProgrammer, (교안) 3.2/3.3.

**상세 절차**
1. (15분) 강사 발표: 내부 플래시 없음 → "Flash boot/Dev boot" 다이어그램 설명
2. (20분) 보드 전원: USB‑C 교체 케이블로 CN10 연결 → 장치 관리자에서 COM 인식 확인
3. (60분) CubeIDE 프로젝트 생성:
   - File → New → STM32 Project → `STM32N657X0H3Q` 선택
   - `Empty project` → Device Configuration Tool에서: LD3(LED, PG0) 출력, USART2 VCOM, B1 버튼(PIN 이름 확인)
   - 코드: LED 250ms 토글 + USART printf + 버튼 인터럽트
4. (40분) 부트 실험:
   - flash boot(1‑2/1‑2)로 재부팅 → 동작 유지 확인
   - CubeProgrammer → dev 모드로 전환 → `0x70000000`에 서명된 이미지 없으면 BootROM 대기 상태임을 확인
5. (30분) Q&A/퀴즈: "SRAM vs 플래시 유지성", "외부 로더란?"

**예상 결과**: UART 1초 타임스탬프, 버튼→LED 토글, "dev 모드엔 프로그램이 살아남지 않는다" 확인

**합격 기준**
- [ ] Flash boot 재부팅 후 펌웨어 유지
- [ ] CubeProgrammer로 외부 메모리(로더) 확인 가능
- [ ] UART 로그 정상

**강사 팁**: N6는 내부 플래시가 없어 "재부팅 = 다시 사인+프로그래밍" 이라는 불편함이 정상.
M2~M3 동안 dev 모드로 자주 스왑하게 되므로, "프로그래밍=dev, 실행=flash" 골격을 강조.

---

### M2. NPU 첫 추론 (3h)

**목표**: stedgeai `generate` → Getting Started 배포 → AiRunner로 지연 실측.

**준비물**: S3·S6 완료, `mobilenetv2_a035_128_fft_int8.tflite`, GettingStarted 분류 프로젝트.

**상세 절차**
1. (30분) 강사 발표: 신경망 최소 모형 + "INT8 정수 추론이 왜 288 MAC/cycle을 쓰는가"
2. (30분) 모델 확인/생성:
   ```
   cd <stm32ai-modelzoo>/image_classification/mobilenetv2/ST_pretrainedmodel_public_dataset/tf_flowers/mobilenetv2_a035_128_fft
   stedgeai generate -m mobilenetv2_a035_128_fft_int8.tflite --target stm32n6 --st-neural-art
   ```
   → `st_ai_output/`의 `network.*`, `network_c_info.json` 확인
3. (60분) Getting Started 포팅:
   - `STM32N6-GettingStarted-ImageClassification` 열기 → `Application/NUCLEO-N657X0-Q/STM32CubeIDE`
   - 생성된 파일(`network.c`, `network_ecblobs.h`, `network_atonbuf.xSPI2.raw`)을 `Model/NUCLEO-N657X0-Q/`에 복사
   - `stmaic_NUCLEO-N657X0-Q.conf` 기반 빌드 설정 `UVCL`로 빌드/플래시 (3.3 CLI 패턴)
4. (40분) 실행/벤치:
   - 보드를 flash boot로 → 실행
   - AiRunner(시리얼 921600)로 추론 시간/출력 확인:
   ```
   python <STEDGEAI_CORE_DIR>/scripts/ai_runner/examples/ai_checker.py -d serial:COM## 2>&1
   ```

**예상 결과**: 화면/시리얼에 flowers 클래스 결과 + ms 단위 추론 시간 표시

**합격 기준**
- [ ] NPU에서 추론 1회 성공
- [ ] 지연 시간 기록(M2 워크시트)
- [ ] `network_atonbuf.xSPI2.raw`가 외부 플래시에 0x70380000으로 기록됨을 아는 상태

**강사 팁**: "weights는 플래시, activations는 SRAM" 이 두 가지만 계속 반복.
첫 실습 프레임부터 "이왕이면 224와 128 둘 다 해보고 표로" 라고 유도(다음 M3 이어짐).

---

### M3. stedgeai 분석·보고서 읽기 (2h)

**목표**: `analyze`/`validate` 구분, `network_c_info.json` 해석, "RAM 부족 판정" 능력

**상세 절차**
1. (20분) analyze/validate/generate 개념 도식
2. (40분) 비교 실습:
   ```
   stedgeai analyze -m mobilenetv2_a035_128_fft_int8.tflite --target stm32n6
   stedgeai analyze -m mobilenetv2_a035_256_fft_int8.tflite --target stm32n6   :: (있으면)
   ```
   → Flash/weights, RAM/activations, latency 항목을 표로 정리
3. (40분) c_info 읽기: `network_c_info.json`에서 I/O shape·데이터 타입·메모리 섹션 확인
4. (20분) 판정 문제: "이 보드에 320×320 검출 모델이 들어갈까?" → 3.6의 예산 계산으로 답

**예상 결과**: "입력 크기↑ → activations↑(SRAM), weights↑(flash)" 표와 1줄 결론

**합격 기준**
- [ ] 모델 A/B 비교표 완성
- [ ] "이 모델은 이 보드에서 RAM 부족/충분" 판정 근거 제시

**강사 팁**: 이 모듈이 "왜 이 코스가 그냥 복붙이 아닌지"를 만드는 지점. 숫자를 추측하지 말고 보고서에서 근거를 찾게 한다.

---

## 5. 교육 진행 — Track B: Zoo 모델 활용

### M4. 사전학습 이미지 분류 배포 (3h)

**목표**: Model Zoo Services YAML 자동 배포 완주(훈련 없이), UVCL 출력.

**준비물**: S6 완료(모델줄+서비스), 3.5 YAML 지식.

**상세 절차**
1. (15분) YAML 섹션 소개(3.5)
2. (30분) 기존 예제 복사:
   ```
   cd C:\ST\stm32ai-modelzoo-services\src
   cp ..\config_file_examples\deployment_n6_config.yaml ..\config_file_examples\deployment_nucleo_edu.yaml
   ```
3. (30분) 편집: `board: NUCLEO-N657X0-Q`, `output: "UVCL"`, `path_to_stedgeai`/`path_to_cubeIDE` 실제 경로
4. (60분) 실행(서명·FSBL 자동 처리 확인):
   ```
   python stm32ai_main.py --config-path ../config_file_examples/ --config-name deployment_nucleo_edu.yaml
   ```
   - 로그에서 "sign_cmd / flash_fsbl_cmd / flash_cmd / flash_network_data_cmd" 흐름 관찰(3.3과 대조)
5. (30분) 모델 교체 실습: 128→224, class_names 변경, 재실행
6. (15분) 학습 포인트: "자동화가 무엇을 했는지 수동 명령으로 매핑"

**예상 결과**: `Deployment complete.` 후 flash boot에서 flowers 분류 라벨 표시

**합격 기준**
- [ ] 1명령 배포 성공 (5클래스 라벨)
- [ ] 자동 배포가 실제 한 "3단계(FSBL/Appli/weights)"를 설명할 수 있음

**강사 팁**: 이쯤에서 "자동화 = 훌륭, 이해 = 필수" 밸런스를 강조. 오류 로그를 무서워하지 않게 "성공+실패 둘 다 보여주기" 데모 추천.

---

### M5. 객체 검출 실시간 배포 (4h) [선택: 카메라]

**목표**: CSI→ISP→NPU→검출박스 전체 파이프라인, UVCL/SPI 출력.

**준비물**: 카메라 모듈(IMX335 등), `st_yoloxn_d033_w025_416_int8.tflite`(또는 yolov8n), GFX01M2(선택).

**상세 절차**
1. (30분) 카메라 장착: IMX335 모듈 → CN6 FPC(ZIF) 고정, I2C2 제어 확인(3.1 주의)
2. (30분) 카메라 미리보기 확인:
   - `x-cube-n6-camera-capture` → 빌드된 hex(to dev mode로 플래시) →
     flash boot → CN8 별도 USB → PC 웹캠 어플리케이션에서 UVC 스트림 확인
3. (60분) 검출 YAML: `model_type: st_yoloxn`, `postprocessing`(confidence/NMS/max_detection_boxes) 설정 → 배포
4. (60분) 실물 테스트: 사람/사물 앞에서 박스+점수 확인, `aspect_ratio: crop/full_screen` 차이 관찰
5. (30분) YOLOv8n(작은입력)과 ST‑YOLO‑Xn(큰입력) fps/wiki 측정 비교표
6. (30분, 선택) `output: "SPI"` + GFX01M2로 온보드 표시

**예상 결과**: 실시간 검출 박스 + ms 표기 (참고 실측: YOLOv8n@256 ≈ 28 inf/s, DK)

**합격 기준**
- [ ] 실물 검출 데모 1건
- [ ] fps·RAM 기록, "이 파이프라인의 병목" 1줄 정리

**강사 팁**: "카메라 안 잡힘"이 교실 최다 이슈. 전원(USB‑C to C), 점퍼(dev/flash), I2C2 충돌을 먼저 진단하게 표준 절차를 칠판에 고정.

---

### M6(선택). 얼굴 검출·분할 등 멀티 모델 (3h)

- `yunetn_320_qdq_int8.onnx`(WIDERFACE) : bucket post‑processing 차이
- `deeplabv3_mnv2_a050_s16_asppv2_320_qdq_int8.onnx`(Pascal VOC): 분할 출력 GUI
- 실습 포인트: `model/model_type` 변경만으로 같은 `deployment` 흐름 재사용
- 합격: 2종 각각 데모+지연 기록

---

## 6. 교육 진행 — Track C: 자체 모델 파이프라인

### M7. Python 훈련 파이프라인과 TinyML 설계 사고 (4h)

**목표**: 임베디드 개발자가 "학습 루프"를 직접 돌리고, 예산(4.2MB SRAM/외장 NOR)과 정확도를 같이 설계.

**준비물**: S5(venv) 완료, 소규모 이미지 데이터(예: 개인 사물 3~5클래스 60~100장/클래스).

**상세 절차**
1. (30분) 강사: TinyML 설계 우선순위(정확도 > 지연 > 메모리 > 에너지) 실사례
2. (40분) 데이터 준비(폴더 구조 `train/val/test`), `torchvision.datasets.ImageFolder` 로드
3. (50분) MobileNetV2 계열 소형 CNN 훈련 스크립트 작성(`width_mult=0.25/0.35/0.5`)
   - 간단 학습 루프(AdamW, CrossEntropy, 5~10 epoch, 셔플·증강 최소)
4. (40분) 훈련 → val 정확도 목표(≥ 90% 작은 것부터), 손실/정확도 커브 저장
5. (30분) ONNX export:
   ```
   torch.onnx.export(model, dummy, "my.onnx", opset_version=13, input_names=["input"], output_names=["output"])
   python -m onnxruntime.quantization.preprocess --input my.onnx --output my-infer.onnx
   ```
6. (30분) **예산 워크시트**: export된 모델 파라미터 수×2/4Byte 계상 — M8에서 INT8로 줄이는 예고
7. (20분) Model Zoo `training` 모드(YAML `operation_mode: training`) 병행 시연

**예상 결과**: 손실커브·export 파일·“이 모델은 SRAM/플래시 예산 ⭐” 워크시트

**합격 기준**
- [ ] val ≥ 90% (또는 합의된 목표)
- [ ] 예산 계산이 “왜 작은 모델이 기본인지” 를 숫자로 설명

**강사 팁**: 임베디드 개발자는 데이터 셔플/증강 경험이 적어, "훈련 정확도 100% & 검증 60%" 함정을 한 번 겪게 유도하는 것이 좋다(과적합 교육 효과).

---

### M8. ONNX QDQ 양자화 · QAT · 가지치기 (4h)

**목표**: PTQ/QAT/채널 가지치기를 손으로 구현, 정확도↔크기 트레이드오프 측정.

**준비물**: M7 산출물(`my-infer.onnx`), 캘리브레이션(대표 100~500장).

**상세 절차**
1. (30분) 이론: QDQ 그래프(DequantLinear — 연산 — QuantLinear)와 per‑channel INT8,
   static 정적 양자화만 지원(동적 불가)
2. (40분) PTQ 실습:
   ```python
   from onnxruntime.quantization import (QuantFormat, QuantType,
       StaticQuantConfig, quantize, CalibrationMethod)
   conf = StaticQuantConfig(
       calibration_data_reader=dr,                 # 대표 이미지 npz/이터레이터
       quant_format=QuantFormat.QDQ,
       calibrate_method=CalibrationMethod.MinMax,
       optimize_model=True,
       activation_type=QuantType.QInt8,
       weight_type=QuantType.QInt8,
       per_channel=True)
   quantize("my-infer.onnx", "my-qdq.onnx", conf)
   ```
3. (30분) 캘리브레이션 실험: 대표 데이터 vs 랜덤 데이터 정확도 차이 기록
4. (30분) `onnxslim` → opset≥13 확인 → `stedgeai analyze`로 INT8 크기/지연 확인(≈1/4 flash)
5. (40분) QAT: PyTorch `torch.ao.quantization`(또는 TF 자동)으로 2~3 epoch fine‑tune 후 재비교
6. (40분) 채널 가지치기: 합의 지표(BN 감마 L1) 기반 임계값으로 채널 제거 → 128/224로
   재훈련 → "크기 ↓ vs 정확도 ↓" 커브 작성 (README.md 7절 복습)
7. (30분) `nodes_to_exclude` 실험: 민감 레이어만 float 유지(혼합)가 정확도에 미치는 영향

**예상 결과**: float↔INT8 크기 ~4배, 정확도 델타 <2%(캘리브레이션·QAT로 회복) 표

**합격 기준**
- [ ] my‑qdq.onnx 생성, analyze리포트 제출
- [ ] "손실 지점 1곳 지목 + 대응" 서술

**강사 팁**: "quant 하는 순간 정확도가 조금 떨어진다"를 dbp당하지 말고,
"이 보드는 INT8만 가속한다" 사실을 먼저 각인시켜야 한다.

---

### M9. chain_qd (양자화 + 배포 자동화) (2h)

**목표**: "float 모델 → 현장 펌웨어" 1명령.

**상세 절차**
1. `chain_qd_n6_config.yaml` 복사/수정: `model: my-onnx`, `dataset.class_names`, calibration/캄비네이션
2. 실행:
   ```
   python stm32ai_main.py --config-path ../config_file_examples/ --config-name chain_qd_nucleo_edu.yaml
   ```
3. 중간 산출물: `quantized_models/<name>_Q.onnx`(또는 tflite) 등 체크
4. 실패 지점 로그 읽기 훈련(퀀트 상태/호환)

**예상 결과**: `Deployment complete.`, 양자화된 중간 모델 존재

**합격 기준**
- [ ] float 모델 1개→동작 펌웨어 성공
- [ ] 실패 시 로그에서 원인 줄을 지목

**강사 팁**: chain_qd가 "블랙박스"화 되는 순간이 많다. 중간 산출물 폴더를 열어
"퀀트는 어디서 끝났는가"를 보게 한다.

---

### M10. 커스텀 모델 standalone 배포 (4h)

**목표**: 자체 모델을 Getting Started/로더에 통합하는 수동 경로. “표준 use‑case 외 모델”도 배포 가능.

**준비물**: `my-qdq.onnx`(M8), GettingStarted(분류 또는 오디오 manual 배포 예).

**상세 절차**
1. (20분) 수동 배포 vs 모델줄 자동 배포 차이 설명(전처리/후처리 커스터마이즈 여지)
2. (30분) 생성:
   ```
   stedgeai generate -m my-qdq.onnx --target stm32n6 --st-neural-art \
       --inputs-ch-position chlast --input-data-type uint8 --output-data-type float32
   ```
   → `network_c_info.json`에서 I/O shape/type 확인
3. (90분) C 통합:
   - `network.c`/`network_ecblobs.h`/`network_atonbuf.xSPI2.raw` → `Model/NUCLEO-N657X0-Q/`
   - `ai_network_run(...)` 호출부, 전처리(리사이즈/uint8 변환), 후처리(내 라벨 매핑) 작성
   - `app_config.h`의 I/O 길이 상수 갱신
4. (60분) 시리얼 검증/프로파일:
   ```
   python <STEDGEAI_CORE_DIR>/scripts/N6_scripts/n6_loader.py    # 로더 응용 빌드·플래시
   stedgeai validate -m my-qdq.onnx --target stm32n6 --mode target \
        -d serial:COM## --val-json network_c_info.json
   ```
5. (30분) 현장 데모(내 데이터 촬영→분류)

**예상 결과**: 내 모델·내 데이터가 보드에서 실시간 추론, 정확도 리포트(타깃) 출력

**합격 기준**
- [ ] end‑to‑end 성립(위 절차 4‑5)
- [ ] 수동 배포 체크리스트(입력 포맷·레이아웃·시리얼 프로토콜) 작성

**강사 팁**: N6 호환의 첫 관문은 "QDQ per‑channel + input uint8/chlast". 이 "규격의 한 끗"에
대부분의 실패가 모이고, 그만큼 교육적으로 가치가 높다.

---

## 7. 교육 진행 — Track D: 센서별 응용

### D1. 카메라(ISP→NPU) 파이프라인 심화 (3h)

**목표**: ISP 3‑파이프(bad pixel·demosaic·crop·downsize·gamma·ROI)와 NPU 직접 공급 이해.

**준비물**: 카메라, x‑cube‑n6‑camera‑capture, M5 재사용.

**상세 절차**
1. (30분) ISP 다이어그램 강의(입력 2672×1940 → NPU 입력 224 등)
2. (40분) `x-cube-n6-camera-capture` 재실행, `crop/downsize/ROI` 설정 변경 → UVC 화면 비교
3. (60분) NPU 입력 크기 스윕 실험(192→256→320): 지연·정확도·fps 표
4. (30분) 고속 데모: ROI만 NPU로 보내 30fps 근접 재현

**예상 결과**: ROI 기반 고속 추론 데모, ISP 설정 영향 비교표

**합격 기준**
- [ ] 30fps 근접(또는 명시된 목표 fps)
- [ ] ISP 설정 항목 3개 이상 동작 설명

**강사 팁**: 이 모듈은 "카메라 하드웨어 비용 없이 소프트웨어로 ROI만 줄여도 빠른" 것을 직관적으로
만드는 지점. 데모 시 실측 fps를 화면 모서리에 항상 표시.

---

### D2. 디스플레이(UVCL/SPI + GFX01M2) (3h)

**목표**: UVCL/SPI 출력 경로와 JPEG·Neo‑Chrom 활용, GUI 오버레이.

**준비물**: GFX01M2(SPI 쪽), M5 출력 프로젝트.

**상세 절차**
1. (20분) UVCL vs SPI 구분(GUI는 PC 웹캠 / 실물 패널)
2. (60분) UVCL 오버레이: 분류 라벨·박스·fps를 JPEG 코덱 가속으로 렌더링 → 웹캠 노출
3. (60분) SPI: `output: "SPI"` + GFX01M2 → 온보드 샘플 GUI(Neo‑Chrom 2.5D 확인)
4. (30분) 갱신률 측정·"GPU 가속 없으면 누가 물" 비교

**합격 기준**
- [ ] 오버레이 GUI 동작(2경로 중 1)
- [ ] 갱신률(ms) 기록

**강사 팁**: 보드에 LCD가 없다는 점에서 UVCL이라는 "PC 화면 디버깅 방식"이 실무적으로 유용함을 강조.

---

### D3. 오디오: 키워드 검출 (4h) [선택: PDM 마이크]

**목표**: PDM→PCM, 프레임화, 스펙트럼 입력, KWS 모델 배포.

**준비물**: PDM 마이크(STEVAL‑MIC1류, SAI/I2S), 모델줄 audio use case(M10 경로 재사용).

**상세 절차**
1. (30분) 오디오 파이프라인 강의: PDM 캡처 → 16k PCM → 스펙트로그램(예: 64×256)
2. (40분) 마이크 연결·캡처 확인(ADC/필터·레벨 미터)
3. (90분) KWS 모델(Model Zoo audio 또는 직접 훈련)을 **M10 수동 경로**로 배포
4. (60분) 현장: 키워드→트리거 확률 표시, 윈도우 길이·오버랩 튜닝

**예상 결과**: 실시간 키워드 트리거 (참고: YAMNet 64×96 ≈ 101 inf/s, DK)

**합격 기준**
- [ ] 키워드 트리거 데모
- [ ] false‑trigger 관찰→튜닝 1회

**강사 팁**: M10에서 배운 수동 배포가 그대로 재사용되는 모듈. "표준 use‑case에 안 들어가면
직접 통합"의 근력이 드러나는 대목.

---

### D4. 시계열: MPU6050/피에조 진동 분류·이상 감지 (4h)

**목표**: 시계열 프레이밍·특징(원시/FFT) 선택, 1D‑CNN 분류 + 오토인코더 이상 감지.

**준비물**: MPU6050(I2C — 카메라와 **I2C2 충돌 주의**→I2C1/3/4 사용), 피에조(ADC 12bit), M10 경로.

**상세 절차**
1. (30분) 시계열 파이프라인 강의: 1kHz 샘플 → 윈도우(256~512)→ 특징 → 모델
2. (40분) 센서 연결:
   - MPU6050: `PB10/PB11`(I2C2는 카메라 전용이므로 I2C1/3/4) — 보드 핀 확인
   - 피에조: 아날로그 ADC 입력(신호 대역 확인), 12bit
3. (50분) 데이터 수집 프로그램: 정상(대기)/불량(진동/타격)을 시리얼로 라벨링 수집
   → PC에서 `npz` 저장
4. (60분) 모델 훈련: **1D‑CNN 분류**(정상/불량) + (비지도) **오토인코더**(재구성 오차, 정상만 학습)
5. (40분) INT8 QDQ → M10 배포 → 실시간 예측(LED/UART)
6. (30분) 임계값 튜닝: false‑alarm(오탐) vs miss(유실) tradeoff 리포트

**예상 결과**: 실물 데모(타격 시 ‘불량’ 검출), 오차 분포 히스토그램

**합격 기준**
- [ ] 분류 정확도 측정(또는 오탐/유실 목표)
- [ ] 재구성 오차 임계값 튜닝 기록

**강사 팁**: "이상 감지 = 클래스 리스트가 없어도 되는" 실전형 임베디드 AI. 시계열을 직접 잡는
(손 타이밍) 실습을 추가하면 임베디드 개발자 몰입도가 크게 오른다.

---

### D5. 통합 프로젝트 (6h)

- 팀 구성 2~3명, M7~M10 + D1~D4 최소 2도메인 결합
- 예: "설비 이상 스냅샷": 피에조 이상 감지(D4) → 카메라 스냅(D1) → UI 표시(D2)
- 산출물: 데모 + 벤치 보고서(X1 양식) + 5분 발표 + 코드 리뷰
- 합격 기준: 기능 완성 / 예산 리포트 / 발표·리뷰 통과

**강사 팁**: 팀별로 "다른 보급형 실패(name)" 경험을 1개씩 발표하도록 하여(10장 트러블슈팅과 연결)
지식 공유를 유도.

---

## 8. 교육 진행 — X1: 최적화·정밀 벤치마크 (3h)

**목표**: 지연/RAM/플래시/에너지를 보고서 양식으로.

**상세 절차**
1. (30분) 지표 정의: latency(ms)·fps·weights(플래시)·activations(SRAM)·에너지(mJ, DK 측정값 토대로)
2. (40분) `optimization: balanced|time|ram` 3회 비교 실습(모델 1개 고정)
3. (30분) 병목 진단: external flash read(가중치 스트리밍) vs NPU 연산 — `network_generate_report` 해석
4. (40분) epoch controller(`--enable-epoch-controller`/`--O3-ec`) 체험 → `network_ecblobs.h` 비교
5. (40분) 참고(강의): DQNN(<8bit)·on‑the‑fly 가중치 압축(Neural‑ART) 원리, `LL_ATON_RT_ASYNC`
6. (30분) 통합 보고서 양식 완성(D5에 재활용)

**합격 기준**
- [ ] 3모드 비교 데이터
- [ ] 병목 1줄 결론 + 근거

**강사 팁**: X1의 결론은 "TOPS 숫자가 아니라 메모리·대역폭 예산" (상위 문서
`Edge_AI_2026_System_Trends.md` 4~5절과 연결).

---

## 9. 운영 체크리스트 (교육 전/중/후)

### 교육 전 (강사, D‑7→D‑1)
- [ ] S1~S8 스모크 테스트(2.8) 자동 통과 / VDI·강의실 PC 복제 이미지 준비
- [ ] 모델줄·서비스 클론 최신 상태 + 버전 매칭(2.3 규칙) 확인
- [ ] `deployment_nucleo_edu.yaml`(M4), `chain_qd_nucleo_edu.yaml`(M9) 사전 검증
- [ ] 카메라/마이크/센서 세트 실습용 하드웨어 점검 (케이블 USB‑C to C)
- [ ] 데이터셋(개인사물/진동) 강의실 공유 폴더 배포

### 교육 중
- [ ] 매 모듈 시작에 "설치/버전 점검 2분 확인"(stedgeai --version, 보드 COM)
- [ ] 각 모듈 종료 시 1인 1결과물(기록표) 저장
- [ ] D5 보드 수 부족 시 2인 1보드 운영

### 교육 후
- [ ] 산출물 폴더 회수, 벤치 보고서 접수
- [ ] 설문: "어느 모듈이 흔들렸는가" → 다음 회차 M 시간 재배분

---

## 10. 상세 트러블슈팅

| 증상 | 원인/확인 | 해결 |
|---|---|---|
| 보드 전원 인식 안 됨 | USB‑A→C 케이블(전력 부족) | USB‑C to USB‑C로 교체 |
| CubeProg 연결 불가 | 부트 모드/로더 | dev 모드(JP1 1‑2/JP2 2‑3) → 외부 로더 `MX25UM51245G_STM32N6570-NUCLEO` |
| 재부팅 후 프로그램 소실 | SRAM 로드 상태 | flash boot(1‑2/1‑2) + FSBL/Appli 사인 플래시 |
| 0x0800_0000 읽으려다 오류 | 내부 플래시 없음(SRAM2가 BOOTROM 영역) | 0x0800_0000을 내부 플래시로 취급 금지, 외부 주소(0x70…)만 사용 |
| 서명한 앱이 안 뜸 | 서명 미적용/타입 오류 | `-t ssbl`(앱)·`-t fsbl`(FSBL), CubeProg ≥2.21은 `-align` 추가 |
| stedgeai 오류 | 모델줄↔툴 버전 불일치 | 모델줄 README "What's new"로 요구 버전 매칭(N6는 매우 예민) |
| link/RAM 부족 | 외장 RAM 없음 | 입력 축소(224 이하)·width 축소·`optimization: ram` |
| NPU 코드만 뜨고 실행 없음 | weights가 0x70380000에 없음 | `network_atonbuf.xSPI2.bin` 프로그래밍 확인(Verify 체크) |
| UVC 스트림 없음 | CN8 케이블 누락 / 부트 모드 | CN8 별도 USB, 플래시(hex) 후 flash boot |
| 카메라 인식 안 됨 | I2C2 제어 불응 / 점퍼 | 카메라가 I2C2 사용 확인, DS‑카메라 재장착, 전원 풀사이클 |
| 검출 정확도 저조 | 캘리브레이션 데이터 누락 | 대표 데이터 재캘리브레이션, `CalibrationMethod` 변경, QAT |
| on_cloud 만 C클라우드 오류 | N6 미지원 | `on_cloud: False`, 로컬 stedgeai 경로 지정 |
| AiRunner 시리얼 무응답 | baud/포트 | 921600, `-d serial:<COM>` 명시 |
| 양자화 정확도 급락 | per‑tensor 사용, 동적 퀀트 | per‑channel INT8, static(QDQ), 민감 노드만 float |
| 훈련 오버피팅(100% vs 60%) | 데이터 증강 부족 | 증강·정규화·epoch 줄임, 데이터 재분할 |

---

## 11. 부록

### A. 자주 쓰는 명령 모음

> `«CubeProg»` = `…\STM32CubeProgrammer\bin\STM32_Programmer_CLI.exe`, `«SigningTool»` = 동일 `bin\STM32_SigningTool_CLI.exe` (2.2 경로로 치환).

```
:: 1) 버전/환경 확인
stedgeai --version
set PATH=%STEDGEAI_CORE_DIR%\Utilities\windows;%PATH%

:: 2) 모델 → NPU C코드
stedgeai generate -m <model> --target stm32n6 --st-neural-art \
    [--inputs-ch-position chlast --input-data-type uint8 --output-data-type float32]

:: 3) 타깃에서 검증/프로파일
stedgeai validate -m <model> --target stm32n6 --mode target -d serial:<COM>

:: 4) 로더/벤치 앱 빌드+플래시
python <STEDGEAI_CORE_DIR>\scripts\N6_scripts\n6_loader.py

:: 5) 모델줄 자동 배포/체인
python stm32ai_main.py --config-path ../config_file_examples/ --config-name <name>.yaml

:: 6) 외부 플래시(CLI, dev 모드)
«CubeProg» -c port=swd mode=HOTPLUG -hardRst -w <FSBL>.hex
«CubeProg» -c port=swd mode=HOTPLUG -hardRst -w <Appli>_signed.bin 0x70100000
«CubeProg» -c port=swd mode=HOTPLUG -hardRst -w network_atonbuf.xSPI2.bin 0x70380000

:: 7) 서명
«SigningTool» -bin <f>.bin -nk -of 0x80000000 -t ssbl -hv 2.3 -o <f>_signed.bin -align
```

### B. 주소·로더 요약

| 항목 | 값 |
|---|---|
| 외부 NOR | `MX25UM51245G` (Octo‑SPI, xSPI2), 외부 로더 `MX25UM51245G_STM32N6570-NUCLEO.stldr` |
| FSBL | `0x7000_0000` (signed, `-t fsbl`) |
| Appli | `0x7010_0000` (signed, `-t ssbl`) |
| Network weights | `0x7038_0000` (`network_atonbuf.xSPI2.bin`) |

### C. 필수 소프트웨어·링크

- STM32CubeIDE / STM32CubeProgrammer / STM32Cube AI Studio (+ST Edge AI Core) — my.st.com
- STM32CubeN6 MCU 패키지(수동 FSBL 템플릿)
- GitHub: `STMicroelectronics/stm32ai-modelzoo`, `…/stm32ai-modelzoo-services`,
  `…/STM32N6-GettingStarted-{ImageClassification,ObjectDetection,PoseEstimation}`,
  `…/x-cube-n6-camera-capture`
- 문서: ST wiki `AI:STM32Cube.AI model performances`(실측 벤치), stedgeai‑cs 문서(quantization, getting started),
  RM0486(부트 모드), UM3417(보드), UM3234(BootROM), AN5967(부트 설정)
- 동향 보충: 바탕화면 `README.md`(양자화/가지치기), `Edge_AI_Recent_Trends.md`,
  `Edge_AI_2026_System_Trends.md` 4절(STM32N6 실측표)

### D. 버전 매칭 체크 시트 (실습 전 반드시)

| 항목 | 값 | 확인 날짜 |
|---|---|---|
| STM32CubeIDE | <버전> | |
| STM32CubeProgrammer | <버전> (2.21+ : `-align`) | |
| ST Edge AI Core / STM32Cube.AI | <버전> | |
| Model Zoo / Model Zoo Services | <커밋/버전> | |
| Getting Started | <패키지 버전> | |
| 보드 | NUCLEO‑N657X0‑Q / MB1940 | |

---

> 이력: v1.0 (2026‑09‑17). 상위 문서 `STM32N6_EdgeAI_Curriculum.md`와 쌍으로 사용.
> 각 모듈의 실측 수치는 최신 도구 버전에서 재검증 후 갱신 권장.