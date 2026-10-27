# ⚙️ STM32 DC Motor Controller

> STM32F411RE에서 **L293 H-Bridge + PWM**으로 DC 모터를 제어하는 상태 기반(State-based) 컨트롤러 팀 프로젝트입니다.
> UART 명령과 보드 버튼(숏/롱클릭)으로 정회전·역회전·정지와 9단계 속도를 제어합니다.

## ✨ 주요 기능

| 기능 | 설명 |
|---|---|
| 방향 제어 | UART `F`(정회전) / `R`(역회전) / `S`(정지) |
| 속도 제어 | UART `1`~`9` → PWM Duty 50% ~ 90% (5% 단계) |
| 버튼 제어 | 숏클릭: 정지↔마지막 방향 / 정·역 토글, 롱클릭(3초): 정지 |
| 안전한 방향 전환 | 방향 전환 시 `Motor_Stop()`을 거쳐 정-정지-역 순서 보장 (탈조 방지) |
| 로그 | UART(printf)로 상태·Duty 출력 |

## 🧠 설계 포인트

- **상태 기반 구조**: `MotorController{state, last_dir, duty}` 구조체 하나로 상태를 관리하고, `Motor_Update()`가 상태를 하드웨어에 반영합니다.
- **PWM**: TIM5 20kHz, PA0/PA1을 정/역 방향 채널로 사용. 한쪽 핀은 PWM(AF), 반대쪽은 0V 출력으로 전환하는 방식으로 H-Bridge를 구동합니다.
- **버튼 이벤트**: 20ms 디바운싱 + TIM2 원샷 타이머로 3초 롱클릭 판정, 이벤트 함수 테이블(`key_event_funcs[]`)로 처리 분기.
- **타이머 분리**: TIM4(1ms 시스템 틱) / TIM2(롱클릭) / TIM5(모터 PWM).
- **UART RX 인터럽트**로 수신 명령을 받아 메인 루프에서 처리.

## 🔌 하드웨어

| 항목 | 내용 |
|---|---|
| MCU | NUCLEO-F411RE (STM32F411RE, Cortex-M4F, 96MHz) |
| 모터 드라이버 | L293 (H-Bridge) |
| 모터 입력 | PA0 → 1A, PA1 → 2A |
| 버튼 | PC13 (온보드 User 버튼) |
| 통신 | USART2, 115200 bps |

> 모터 전원은 MCU 전원과 분리하는 것을 권장합니다 (USB 전류 제한으로 보드가 리셋될 수 있음).

## 📁 파일 구성

| 파일 | 역할 |
|---|---|
| `main.c` | 메인 루프, UART 명령/버튼 이벤트 처리 |
| `motor.c/.h` | 모터 상태 정의 및 방향·Duty 제어 |
| `timer.c/.h` | TIM2/TIM4/TIM5 초기화, PWM Duty 설정 |
| `key.c/.h` | 버튼 디바운싱, 숏/롱클릭 판정 |
| `uart.c`, `clock.c`, `exception.c` | UART, 클럭, 인터럽트 핸들러 |
| `device_driver.h`, `macro.h`, `option.h` | 공용 선언/비트 매크로/클럭 상수 |
| `crt0.s`, `rom_0x08000000.lds` | 시작 코드, 링커 스크립트 |

## 🚀 빌드 및 실행

```bash
make        # 빌드 (arm-none-eabi-gcc)
make run    # STM32CubeProgrammer로 다운로드
make clean
```

Tera Term 등으로 115200 접속 후 `F` → `5` → `R` → `S` 순으로 입력해 동작을 확인합니다.

## 📚 배운 점

- H-Bridge 동작 원리와 정·역 전환 시 정지 구간의 필요성
- 타이머 기반 PWM 및 버튼 이벤트(디바운싱, 롱클릭) 처리
- 레지스터 직접 제어 방식의 펌웨어 모듈화
