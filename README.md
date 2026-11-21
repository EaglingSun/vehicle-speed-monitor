# vehicle-speed-monitor — 차속 감시(VSM) 예제 코드

차속 센서 값을 걸러 경고 등급을 판정하고 진단 이력에 기록하는 C 예제 코드입니다.
요구사항은 `test_ref/요구사항.md` 에 있습니다. 팀 규칙과 팀이 승인한 도구 버전은 팀 하네스 저장소(`team-harness`)에 있습니다.

## 빌드 환경 — 이 프로젝트가 정한 것

| 항목 | 값 |
|---|---|
| 운영체제 | Windows 10 1809 이상 · Windows 11 |
| 컴파일러 | WinLibs GCC 15.2.0 + MinGW-w64 14.0.0 · 32비트(i686) · MSVCRT · release 7 |
| 컴파일러 배포 파일 | `winlibs-i686-posix-dwarf-gcc-15.2.0-mingw-w64msvcrt-14.0.0-r7.zip` — github.com/brechtsanders/winlibs_mingw 릴리스 `15.2.0posix-14.0.0-msvcrt-r7` |
| 배포 파일 SHA256 | `ed7fb2db387834849af00216a609d36b3d65730ba28ca73c7d759360edaeb913` |
| make | 같은 배포 파일의 `mingw32-make.exe` |
| 빌드 | `make` → `vsm_demo.exe` |
| 시험 | `make test` → `test-results/test_<모듈>.xml` · 실패한 시험이 있으면 0 이 아닌 값으로 끝난다 |

- `src/` · `include/` · `tests/` · `Makefile` 은 이 프로젝트의 기준이다. 빌드 환경을 맞추려고 고치지 않는다.
- 이 저장소에는 아직 빌드 환경을 맞추는 스크립트와 안내 문서가 없다.

## 구성

```
vehicle-speed-monitor/
  Makefile           빌드 · 시험 — CC 는 PATH 에서 찾은 gcc
  test_ref/
    요구사항.md      요구사항 명세
  include/
    vsm_types.h      경고 등급 enum(vsm_warn_t) · 차속 한계값 · 샘플 수
    speed_monitor.h  차속 판정
    sensor_filter.h  센서 필터
    diag_log.h       진단 이력
  src/
    main.c           데모 진입점 — 차속 0 ~ 220 을 20 씩 올리며 판정 결과를 출력
    speed_monitor.c  sm_evaluate() · sm_reset() · sm_over_count()
    sensor_filter.c  sf_push() · sf_latest() · sf_average() — 8칸 링 버퍼
    diag_log.c       dl_record() · dl_parse_threshold() · dl_read_status_reg()
  tests/
    vsm_test.h       외부 프레임워크 없는 C 단위 시험 틀 — 검사 매크로 + JUnit XML 쓰기
    test_speed_monitor.c  기존 시험 1건
```

## 함수

| 함수 | 하는 일 |
|---|---|
| `sm_evaluate(speed_kph, brake_on)` | 차속으로 경고 등급(NONE · LOW · MID · HIGH)을 판정하고 과속 횟수를 센다 |
| `sm_reset(ecu_id)` | 과속 카운터 초기화 |
| `sm_over_count()` | 과속 횟수 |
| `sf_push(value)` · `sf_latest()` | 링 버퍼에 센서 값을 넣고 · 최근 값을 꺼낸다 |
| `sf_average(samples, count)` | 샘플 평균 |
| `dl_record(warn)` | 경고 등급을 진단 이력에 기록 |
| `dl_parse_threshold(text)` | 임계값 문자열을 숫자로 — 해석 못 하면 -1 |
| `dl_read_status_reg()` | 하드웨어 상태 레지스터(`0x40001000`) 읽기 — PC 에서는 호출하지 않는다 |
