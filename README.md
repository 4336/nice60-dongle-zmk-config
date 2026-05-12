# nice!60 Dongle ZMK Config

nice!60 키보드를 **nice!nano v2 동글**과 함께 사용하기 위한 ZMK 펌웨어 설정입니다.

## 구성 개요

[nice60-zmk-config](https://github.com/Nice-Keyboards/nice60-zmk-config) (공식 nice!60 ZMK 설정)를 기반으로,
ZMK의 [Keyboard Dongle](https://zmk.dev/docs/development/hardware-integration/dongle) 기능을 적용했습니다.

```
[nice!60] ── BLE ──▶ [nice!nano v2 동글] ── USB ──▶ [PC]
 peripheral                 central
 (모든 키 스위치)          (HID 출력 담당)
```

- **nice!60** — 키 스위치를 스캔해 BLE로 동글에 전송하는 peripheral 역할
- **nice!nano v2** — USB 또는 BLE로 PC에 연결되는 central 역할

## 이 구성의 장점

| 항목 | 효과 |
|---|---|
| 배터리 수명 | nice!60이 peripheral이 되어 전력 소모 대폭 감소 |
| 유선 안정성 | 동글을 USB로 연결하면 BIOS·부팅 화면에서도 인식 |
| 무선 사용 | 동글에 배터리 내장 시 완전 무선으로도 사용 가능 |
| 전환 | 키맵 매크로(`&out OUT_USB` / `&out OUT_BLE`)로 유·무선 전환 |

## 빌드 결과물

GitHub Actions로 빌드하면 `.uf2` 파일 4개가 생성됩니다.

| 파일 | 플래시 대상 | 용도 |
|---|---|---|
| `nice60_dongle-nice_nano_nrf52840_zmk-zmk.uf2` | nice!nano v2 | 동글 펌웨어 |
| `settings_reset-nice_nano_nrf52840_zmk-zmk.uf2` | nice!nano v2 | 최초 페어링 전 초기화 (1회) |
| `settings_reset-nice60_zmk-zmk.uf2` | nice!60 | 최초 페어링 전 초기화 (1회) |
| `nice60_zmk-zmk.uf2` | nice!60 | Peripheral 펌웨어 |

## 플래시 순서 (최초 1회)

> 기존 페어링 정보를 제거하고 새로 등록하는 과정입니다.

1. **동글(nice!nano v2)**에 `settings_reset` 플래시 → 자동 재부팅 대기
2. **nice!60**에 `settings_reset` 플래시 → 자동 재부팅 대기
3. **동글**에 `nice60_dongle` 펌웨어 플래시
4. **nice!60**에 `nice60` peripheral 펌웨어 플래시
5. 동글을 USB에 꽂으면 자동으로 nice!60과 페어링됨

## 키맵 레이어

기존 nice!60 키맵을 그대로 유지합니다.

| 레이어 | 진입 방법 | 주요 기능 |
|---|---|---|
| 0 `mac_default` | 기본 | Mac 배열 |
| 1 `win_default` | `bt_layer`의 `TOG 1` | Windows 배열 (Ctrl/Alt/GUI 순서 변경) |
| 2 `fn_layer` | `Caps Lock` 키 홀드 | 방향키, F1~F12, PgUp/Dn 등 |
| 3 `bt_layer` | 우하단 `FN` 키 홀드 | BT 프로필, RGB, 출력 전환 |

### bt_layer 주요 키

| 키 위치 | 기능 |
|---|---|
| `ESC` | USB ↔ BLE 출력 토글 (`OUT_TOG`) |
| `1`~`5` | BT 프로필 0~4 선택 |
| `BKSP` | 현재 BT 프로필 연결 해제 (`BT_CLR`) |
| `Tab` | Win 레이어 토글 |
| `Left Ctrl` | USB 출력 고정 (`OUT_USB`) |
| `Left Alt` | BLE 출력 고정 (`OUT_BLE`) |
| `Q`~`I` | RGB 제어 (토글, 색상, 밝기, 속도, 효과) |

## 파일 구조

```
├── build.yaml                          # 4개 펌웨어 빌드 정의
└── config/
    ├── west.yml                        # ZMK main 브랜치 참조
    ├── boards/shields/nice60_dongle/
    │   ├── Kconfig.shield              # 쉴드 선언
    │   ├── Kconfig.defconfig           # Split central, BT 연결 수 설정
    │   └── nice60_dongle.overlay       # Mock kscan + 매트릭스 트랜스폼
    ├── nice60_dongle.keymap            # 동글(central)이 실행하는 키맵
    ├── nice60_dongle.conf              # 동글 설정 (절전, ZMK Studio 등)
    ├── nice60.keymap                   # Peripheral 빌드용 (동작은 동글이 수행)
    └── nice60.conf                     # Peripheral 설정
```

> GitHub Actions는 `-DZMK_CONFIG=.../config`로 빌드하므로, 커스텀 쉴드는
> `config/boards/shields/nice60_dongle/` 아래에 둡니다.

## 참고

- [ZMK Keyboard Dongle 문서](https://zmk.dev/docs/development/hardware-integration/dongle)
- [ZMK Split Keyboards 문서](https://zmk.dev/docs/features/split-keyboards)
- [nice!60 공식 ZMK 설정](https://github.com/Nice-Keyboards/nice60-zmk-config)
- [nice!nano v2](https://nicekeyboards.com/nice-nano/)
