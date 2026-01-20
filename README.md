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


