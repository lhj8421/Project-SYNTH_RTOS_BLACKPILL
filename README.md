# Project-SYNTH_RTOS_BLACKPILL

## 📋 프로젝트 개요

### 디지털 신디사이저 구현
  * 실시간 오디오 처리
  * 스위치와 로터리 입력으로 파라미터 제어
  * 다양한 음색 생성

### RTOS 기반 설계
  * 실시간 처리의 정확성과 안정성 보장
  * 다중 태스크 처리를 통한 효율적 자원 관리
  * 우선순위 기반 스케쥴링으로 오디오 처리 지연 최소화


## 🔧 하드웨어 구성
| 하드웨어 | 역할 |
|------|------|
| STM32 Black Pill (ARM Cortex-M) | 실시간 오디오 처리 및 사용자 입력 제어 |
| I2S 기반 외부 DAC (UDA1334A) | 디지털 오디오 신호를 아날로그 신호로 변환하여 출력 |
| TFT LCD 모듈 | ADSR 엔벨로프, Cutoff / Resonance 파라미터 및 상태 표시 |
| 4×4 버튼 매트릭스 | 노트 입력 및 피치 설정 |
| 로터리 엔코더 ×2 | ADSR, Cutoff Frequency, Resonance(Q), 출력 사운드 실시간 조절 |

<img width="1080" height="522" alt="image" src="https://github.com/user-attachments/assets/240107dd-b7ab-4add-9d5d-ce07734bb43c" />

## 🎵 소리의 3요소
1. 진폭 (Amplitude)
소리의 3요소 중 소리의 크기를 결정합니다.
2. 주파수 (Frequency)
소리의 높낮이를 결정하며, 음계와 직접적인 관계가 있습니다.
3. 파형 (Waveform)
소리의 색깔(음색)을 결정합니다.

* 사인파 (Sine Wave): 순수하고 부드러운 소리
* 사각파 (Square Wave): 로봇 같고 기계적인 소리
* 톱니파 (Sawtooth Wave): 밝고 강한 소리


## 🔊 파형 생성
<img width="1034" height="388" alt="image" src="https://github.com/user-attachments/assets/2e58aef5-6d38-4ae6-9560-fe3d554e6eec" />

룩업 테이블 사전 초기화
각 파형의 한 주기를 1024개의 점으로 균등 분할하여 int16_t 배열에 저장
* tuning_word = f_target * 2^32 / F_sample
* phase_acc += tuning_word
* index = phase_acc >> SHIFT 로 LUT 접근
* 샘플레이트: F_sample = 44.1kHz


## 🎚️ 신호 처리
### ADSR (Envelope)
* 소리가 시간에 따라 어떻게 변화하는지를 정의
![537391460-5211d62c-eb23-4a60-aaa6-2dedb4405495](https://github.com/user-attachments/assets/52440d38-ae94-492b-b960-e8bd7bb22f09)

 * Attack: 소리가 최대 음량에 도달하는 시간
 * Decay: 최대 음량에서 지속 음량으로 감소하는 시간
 * Sustain: 키를 누르고 있는 동안 유지되는 음량
 * Release: 키를 뗀 후 소리가 사라지는 시간

### IIR Low-Pass Filter
| Cutoff (LPF) | Resonance (Q) |
|:------------:|:-------------:|
| ![Cutoff](https://github.com/user-attachments/assets/a9b86e20-c2d3-4bc3-b6d2-0a4252169233) | ![Resonance](https://github.com/user-attachments/assets/175dc904-57f2-4de7-a56a-843191e75509) |

* Cutoff Frequency: 특정 주파수 이상의 고주파 성분 제거
* Resonance (Q Factor): Cutoff 주파수 근처의 신호를 강조


## 🔄 오디오 신호 처리 흐름도
<img width="1054" height="220" alt="image" src="https://github.com/user-attachments/assets/076195dd-55a6-4928-92c6-61c013771330" />


## 📡 DMA 전송
### DMA를 활용한 효율적인 오디오 데이터 전송
<img width="1592" height="1017" alt="image" src="https://github.com/user-attachments/assets/06b27eda-0f5d-4262-8d33-6d85c6bda12d" />

* Circular Buffer 방식: Half/Full 콜백으로 CPU 개입 최소화
* 44.1kHz 샘플링: I2S DMA를 통해 DAC로 전송
* CPU 부하 감소: DMA가 데이터 전송을 담당하여 CPU는 샘플 생성에 집중

## ⚙️ RTOS vs Bare-metal
<img width="1012" height="247" alt="image" src="https://github.com/user-attachments/assets/9cdc885a-402a-401d-9ce1-70d54df4983c" />

### Bare-metal (Super Loop)의 한계
* 다른 Task의 소요 시간이 길면 Task 1의 실시간성 보장 어려움
* 무거운 작업을 ISR에서 처리하기 부담스러움

### RTOS(FreeRTOS) 사용 이유
* 우선순위 기반 스케줄링
 * High Priority: Sound Engine (가장 높은 우선순위)
 * Medium Priority: Button + Encoder
 * Low Priority: UI

* Task 상태 관리
 * Running → Ready → Blocked → Suspended
 * Event 기반 Task 전환으로 효율적인 CPU 사용


## 🔧 트러블 슈팅 (Trouble-shooting)

| 문제 | 현상 | 원인 | 해결 |
|------|------|------|------|
| **스위치 문제** | 풀업 저항 작동 안함 | MCU의 풀업 저항 고장 | MCU 핀 위치 변경 |
| **버퍼 언더런** | 일정 주기로 짧게 소리가 끊김 | 샘플 생성 루프 내 과도한 연산으로 인해 I2S DMA 버퍼를 제때 채우지 못함 | 반복적으로 수행되던 계산을 루프 외부로 이동하여 샘플 생성 경로를 최적화함 |
| **LCD SPI 통신 오류** | LCD 라이브러리 파일에서 타임아웃 1ms로 설정, 타임아웃 부족으로 SPI 전송 실패하여 화면 깨짐 | 타임아웃 시간 부족 | HAL_MAX_DELAY 설정, 무한대기를 통해 해결 |


## 👥 팀원

| 이름 | 역할 |
|------|------|
| 원종완 | 팀장 |
| 서민솔 | 파형 및 필터 |
| 이동우 | 입력 장치 |
| 안창희 | 입력 장치 |
| 유종민 | 파형 및 필터 |
| 김영교 | UI |
| 이환중 | UI |



