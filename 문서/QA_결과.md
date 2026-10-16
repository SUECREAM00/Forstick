# 포스틱 QA 결과

> **최신(2026-10-02, 커밋 `c0e1374`): 화면설계서 기능 구현 후 Playwright 55개 중 53 통과 · 실패 0 · 미실행 2**(로봇 동작 검사 — 팀원 허락 대기, D5).
> 세 번 연속 같은 결과. 아래 1~4장은 구현 전 기록이고, 6장이 구현 뒤 남은 것이다.

- 실행일: 2026-10-02 · 기준 커밋 `03dbcb6` · 실행 PC: Windows(이 PC), 작업 셀 서버 `<내부 주소>` 켜짐
- 무엇을 검사했는지는 `QA_요구사항_추적표.md`. 판정은 **통과 / 실패 / 미실행**뿐이다(미실행을 통과로 세지 않는다).
- 다시 돌리는 법(`Fourstick/dashboard2`): `npm run qa`(화면+서버) → `npm run qa:report`(표), 백엔드 대응: 저장소 루트에서 `python dashboard2/e2e/backend_map.py`

## 한눈에

| 영역 | 통과 | 실패 | 미실행 | 비고 |
|---|---|---|---|---|
| 백엔드 기능(SFR·TER, 기존 테스트 실행) | 16 / 18 항목 | 2 항목 | 0 | 실패 2개는 같은 원인: Windows에서 SQLite 잠김(테스트 코드 문제, 백엔드라 수정 안 함) |
| 서버 API(블랙박스) | 8 | 0 | 2 | API-07은 D7(필터는 정확히 일치하는 6개 키) 기준으로 통과. 로봇 동작 검사 2개는 팀원 허락 전 미실행(D5) |
| 화면(dashboard2) | 18 | 27 | 0 | "즉시 정지"(D4) 반영해 UI-ESTOP-02 통과. UI-SIM-01은 D3로 제외(기획서 갱신 대상). 나머지 실패는 화면설계서 기능이 아직 없음 |
| 측정 지표(NFR-001~005·007~009, TER-003·005) | — | — | 전부 | 로봇 동작·평가셋 실행 필요(D5) |

### 결론
- **백엔드 핵심 기능(STT·슬롯·계획·형식·안전·기하·실행 허가·정지·저장·어댑터·조건 해소)은 테스트가 있고 통과**한다(SFR 대응 테스트 약 1,030개, 이 PC 기준).
- **서버 동작**: 시연 명령은 바로 실행하지 않고 확인 카드(만료 60초)를 만들며, 취소하면 작업이 생기지 않는다. 3D 관측 스트림 29.7회/초.
- **화면은 요구사항 대비 크게 부족**하다. 명령 패널 흐름(질문·차단·최종점검·만료·취소·오류)과 개발 중 결정(라이트 테마, 표지 없음 등)은 통과했지만,
  화면설계서의 상태 위젯·긴급 배너·안전 3종·로봇 관리 목록/탭·기록 필터/검색/타임라인·진단·설정 3구역·연결 상태·제어권·반응형은 **구현되지 않았다**.
- 화면 데이터 대부분이 목업이라 설계원칙 2("화면은 서버 값을 그대로")를 지키지 못한다(D2).

## 1. 백엔드 기능 — 기존 테스트 대응(`backend_map.py`)
| 요구사항 | 모듈 수 | 테스트 수 | 통과 | 실패 | 오류 | 건너뜀 | 판정 | 실패한 테스트(최대 3개) |
|---|---|---|---|---|---|---|---|---|
| SFR-001 | 5 | 104 | 104 | 0 | 0 | 0 | 통과 |  |
| SFR-002 | 2 | 56 | 56 | 0 | 0 | 0 | 통과 |  |
| SFR-003 | 5 | 83 | 83 | 0 | 0 | 0 | 통과 |  |
| SFR-004 | 4 | 146 | 146 | 0 | 0 | 0 | 통과 |  |
| SFR-005 | 3 | 33 | 33 | 0 | 0 | 0 | 통과 |  |
| SFR-006 | 3 | 59 | 59 | 0 | 0 | 0 | 통과 |  |
| SFR-007 | 3 | 54 | 51 | 0 | 0 | 3 | 통과 |  |
| SFR-008 | 4 | 103 | 103 | 0 | 0 | 0 | 통과 |  |
| SFR-009 | 5 | 80 | 80 | 0 | 0 | 0 | 통과 |  |
| SFR-010 | 2 | 61 | 61 | 0 | 0 | 0 | 통과 |  |
| SFR-011 | 2 | 111 | 109 | 0 | 2 | 0 | 실패 | integration.test_storage_roundtrip.TestPlanRoundTrip.test_persists_across_connections<br>integration.test_storage_roundtrip.TestSchemaAndMigration.test_migration_is_idempotent_across_reopen |
| SFR-013 | 4 | 108 | 108 | 0 | 0 | 0 | 통과 |  |
| SFR-014 | 1 | 29 | 29 | 0 | 0 | 0 | 통과 |  |
| NFR-006 | 2 | 112 | 110 | 0 | 2 | 0 | 실패 | integration.test_storage_roundtrip.TestPlanRoundTrip.test_persists_across_connections<br>integration.test_storage_roundtrip.TestSchemaAndMigration.test_migration_is_idempotent_across_reopen |
| TER-001 | 1 | 15 | 15 | 0 | 0 | 0 | 통과 |  |
| TER-002 | 2 | 24 | 24 | 0 | 0 | 0 | 통과 |  |
| TER-004 | 1 | 47 | 47 | 0 | 0 | 0 | 통과 |  |
| TER-006 | 1 | 29 | 29 | 0 | 0 | 0 | 통과 |  |

- SFR-007의 건너뜀 3건은 MoveIt/ROS가 필요한 테스트(이 PC에 없음)다.
- SFR-011·NFR-006 실패 2건: `test_storage_roundtrip`이 SQLite 연결을 닫기 전에 임시 폴더를 지운다 → Windows에서만 PermissionError. 기능 결함이 아니라 **테스트 코드 결함**이고, 리눅스(작업 셀 PC)에서는 통과할 것으로 본다(미확인).
- SFR-012(터치 스킬 버튼)는 백엔드 테스트 대상이 아니라 화면(UI-CMD-10)에서 실패.

## 2. 화면·서버 자동 테스트(Playwright) — D1~D8 반영 후 재실행

| 통과 | 실패 | 미실행 | 합계 |
|---|---|---|---|
| 26 | 27 | 2 | 55 |

| 프로젝트 | 테스트 ID | 요구사항 | 내용 | 결과 | 실패 원인·측정값 |
|---|---|---|---|---|---|
| 서버 | API-01 | SFR-011·DEV-10 | /health: DB 사용 가능, 로봇은 시뮬레이션 | 통과 |  |
| 서버 | API-02 | DEV-01 | 3D 관측: 모델을 주고 스트림이 초당 20회 이상 | 통과 | 측정: 3D 관측 27.5회/초 / 측정: 3D 관측 27.5회/초 |
| 서버 | API-03 | DEV-01 | Gazebo 장면 카메라 사용 가능(대체 경로) | 통과 |  |
| 서버 | API-04 | SFR-010 | 시연 명령이 아닌 문장: 판정 + 사람이 읽는 사유, 작업 없음 | 통과 |  |
| 서버 | API-05 | SFR-003·⑥-2 | 대상이 모호한 문장은 되묻는다(작업 없음) | 통과 |  |
| 서버 | API-06 | DEV-11·⑥-4 | 이송 명령은 바로 실행하지 않고 확인 카드(만료 60초)를 만든다 — 취소하면 작업 0건 | 통과 | 판정: CONFIRM · A자재를 컨베이어 2번 위치로 옮기겠습니다. / 판정: CONFIRM · A자재를 컨베이어 2번 위치로 옮기겠습니다. |
| 서버 | API-07 | DEV-10 | 공개 응답에 실기 상태 키(공개 필터의 6개 키)가 없다 | 통과 |  |
| 서버 | API-08 | SFR-009 | 정지 래치 진단이 읽히고 시뮬레이션으로 표시된다 | 통과 |  |
| 서버 | API-ROBOT-01 | TER-003·NFR-009 | 이송 → 원래 자리 복귀 E2E, 실행 성공률 | 미실행 | skip: QA_ROBOT=1이 아님 — 로봇을 움직이는 검사는 팀원 허락 뒤에만 돈다(미실행) / skip: QA_ROBOT=1이 아님 — 로봇을 움직이는 검사는 팀원 허락 뒤에만 돈다(미실행) |
| 서버 | API-ROBOT-02 | NFR-002·TER-004 | 실행 중 즉시 정지: 요청→응답 시간, 정지 확인 | 미실행 | skip: QA_ROBOT=1이 아님 — 로봇을 움직이는 검사는 팀원 허락 뒤에만 돈다(미실행) / skip: QA_ROBOT=1이 아님 — 로봇을 움직이는 검사는 팀원 허락 뒤에만 돈다(미실행) |
| 화면 | UI-CMD-01 | SFR-001·DEV-04 | 대기: 텍스트 입력 + 입력칸 안 마이크 + 보내기, STEP 제목·Enter 안내 없음 | 통과 |  |
| 화면 | UI-CMD-02 | ⑥-2·§4 ASK | 질문은 정보입력 요청 — 서버 질문 문장 + 답 입력칸, 승인 버튼 없음 | 통과 |  |
| 화면 | UI-CMD-03 | ⑥-3 BLOCK | 차단은 실행 불가 통보 — 사유 문장, 우회 버튼 없음, "명령 수정"으로 입력 복귀 | 통과 |  |
| 화면 | UI-CMD-04 | ⑥-4·SFR-010 | 실행 전 최종점검: 서버 요약 + 실행 승인/취소 + 남은 시간, 승인 시 확인 요청 1회 | 통과 |  |
| 화면 | UI-CMD-05 | §4 결과 만료 | 확인 시간이 지나면 실행 승인 버튼이 잠긴다 | 통과 |  |
| 화면 | UI-CMD-05b | 취소 | 취소하면 확인 요청은 cancel로 가고 작업이 시작되지 않는다 | 통과 |  |
| 화면 | UI-CMD-06 | ⑥-1·§11 | 중간과정 3단계 체크리스트(동작 준비 → 안전 규칙 확인 → 가상 동작 확인) | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-CMD-07 | ⑦·§10 | 대상 로봇 선택 Chip, 명령 직전 로봇명 재명시 | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-CMD-08 | 오류 | 서버에 닿지 못하면 오류 카드 + 새 명령 입력 | 통과 |  |
| 화면 | UI-CMD-10 | SFR-012 | 터치 기반 스킬 버튼(No-Code fallback) | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-COMMON-01 | ㉔·§7 | 연결 상태·데이터 수신 시각 전역 표시 | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-COMMON-02 | ㉕·§9 | 제어권·운전 모드 표시 | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-DEV-01 | DEV-02 | 라이트 테마: 페이지 바탕 #e4e9f0, 카드 #e8edf3 | 통과 |  |
| 화면 | UI-DEV-02 | DEV-03 | "실제 로봇 아님" 상시 표지가 없다 | 통과 |  |
| 화면 | UI-DEV-03 | DEV-05 | 위험 판정 창 영상 칸에 문구 없음 · 설정 정책 문장에 "(안)" 없음 | 통과 |  |
| 화면 | UI-DEV-04 | DEV-09 D2 | 화면 값은 서버에서 온다 — 목업 데이터 모듈을 쓰지 않는다 | 실패 | Error: expect(received).not.toMatch(expected) |
| 화면 | UI-DEV-05 | DEV-12 | 개발 서버는 localhost에만 열린다(인증 없음) | 통과 |  |
| 화면 | UI-DIAG-01 | ⑰·⑲-2·§15 | 서비스 상태 목록(정상/경고/장애/수신없음) | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-DIAG-02 | ⑲-5 | 진단자료 내보내기 | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-ESTOP-01 | SFR-009·DEV-06 | 정지 버튼은 시뮬레이션 창이 열려 있어도 보이고 눌린다 | 통과 |  |
| 화면 | UI-ESTOP-02 | ㉓·용어표 D4 | UI STOP 버튼 문구는 "즉시 정지"(비상정지는 물리 E-stop 상태 표시 전용) | 통과 |  |
| 화면 | UI-HOME-01 | ①·§1 | 로봇 상태는 색+모양 이중 구분(●◆▲■?) | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-HOME-02 | ①·용어표 | 로봇 카드 1차 표시는 셀 위치·별칭(모델명 UR5e/FR3 아님) | 통과 |  |
| 화면 | UI-HOME-03 | ②·§2 | 안전 3종(비상정지·보호정지·가드) 개별 표시 영역이 있다 | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-HOME-04 | ③·§3 | 정상 범위 연속값은 원시 수치 없이 "정상" 배지만 | 실패 | Error: expect(locator).not.toContainText(expected) failed |
| 화면 | UI-HOME-05 | ④⑤·§5·6 | 긴급 배너(긴급 N건 집계, 정지 버튼 비가림) | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-HOME-06 | ⑥·§11 | 사이드바 로봇 관리 배지 "가동 N/전체" | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-LAYOUT-01 | 가-1 | 좌측 메뉴 5개: 현황·로봇 관리·기록·진단·설정 | 통과 |  |
| 화면 | UI-LAYOUT-02 | 가-2 | 모든 화면이 우측 명령 패널을 공유한다 | 통과 |  |
| 화면 | UI-LAYOUT-03 | 가-3 | 명령 패널 폭을 드래그로 조절할 수 있다 | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-LAYOUT-04 | 가-4·§15 | 사이드바 System status 위젯 4행(ROS 2·PLANNER·SAFETY PLC·LATENCY) + "N/4 정상" | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-RECORD-01 | ⑮·§14 | 필터 두 축 분리: 검증 결과 / 실행 결과 | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-RECORD-02 | ⑯-4 | 명령 원문·ID 텍스트 검색 | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-RECORD-03 | ⑭·⑯-2 | 명령 1건 상세 타임라인(계획→시뮬레이션→허가→실행) | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-RESP-01 | DEV-07 | 폭 1024px에서 가로 스크롤 없음, 비상 정지 보임 | 실패 | Error: expect(received).toBeLessThanOrEqual(expected) |
| 화면 | UI-RESP-01 | DEV-07 | 폭 820px에서 가로 스크롤 없음, 비상 정지 보임 | 실패 | Error: expect(received).toBeLessThanOrEqual(expected) |
| 화면 | UI-RESP-02 | DEV-08 | 태블릿(1024px): 사이드바는 아이콘 레일(≤80px) | 실패 | Error: expect(received).toBeLessThanOrEqual(expected) |
| 화면 | UI-ROBOTS-01 | ⑬-1 | 목록 + 선택 시 우측 상세 패널 구조 | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-ROBOTS-02 | ⑫·§12 | 운용 가능 여부는 결과값으로 표시, 켜고 끄는 토글 없음 | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-ROBOTS-03 | ⑬-2 | 상세 탭 3개: 개요·도구 장착 이력·프로파일 버전 | 실패 | Error: expect(locator).toBeVisible() failed |
| 화면 | UI-SET-01 | ⑳·§16 | 설정 3구역: 개인 / 운영 / 안전·권한 | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-SET-02 | ㉑·§17 | 안전 임계값 변경은 상태머신(작성 중→검사 중→승인 필요→적용 예약→사용 중) | 실패 | Error: expect(locator).toContainText(expected) failed |
| 화면 | UI-SIM-02 | ⑧·§4 | 영상(3D·카메라)을 못 받으면 빈 그림 대신 이유를 적는다 | 통과 |  |
| 화면 | UI-SIM-03 | DEV-01 | 3D 모델을 못 받으면 Gazebo 영상 경로로 대신한다 | 통과 |  |
| 화면 | UI-TERM-01 | 용어표 | 화면 1차 문구에 내부값 PASS/BLOCK/ASK를 쓰지 않는다 | 실패 | Error: home |


## 3. 실행하지 않은 것(측정 필요)
| 항목 | 이유 |
|---|---|
| NFR-001 응답 p95 5초, NFR-004·005·008 안전·오차단·되묻기 지표, TER-005 봉인 평가 | 평가셋(`fixtures/workcell_eval`)을 실제 LLM으로 돌려야 함 — 작업 셀 PC에서 |
| NFR-002 정지 반응시간, NFR-009 시뮬레이션 성공률, TER-003 E2E | 로봇을 실제로 움직여야 함(D5) |
| NFR-003 GPU 메모리 | 작업 셀 PC에서 측정 |

## 4. 결정 필요
| 번호 | 내용 |
|---|---|
| D1 | 요구사항 기준: 기획서(UR5e+FR3 2종·루시아·PWA) vs 현재 구현(FR3 1종·Qwen3·React) |
| D2 | 대시보드 목업 → 실제 서버 값 전환 범위(목록 API가 없어 백엔드 수정 필요) |
| D3 | 시뮬레이션 창 "SIMULATION — 실시간 아님"(⑧) vs "실제 로봇 아님 표지 없음"(10-01 결정) 충돌 — UI-SIM-01 |
| D4 | 헤더 버튼 "비상 정지" vs 용어표 "즉시 정지" — UI-ESTOP-02 |
| D5 | 로봇을 움직이는 검사(E2E·정지 반응시간·성공률) 실행 여부·시점 |
| D6 | 화면설계서 기능(권한·설정 3구역·진단 내보내기·기록 타임라인 등)의 이번 구현 범위 |
| D7 | 공개 응답 필터가 `AGENTS.md` 설명(`real_hardware*`)과 달리 6개 키만 지움 → `/v1/robots`에 `real_hardware_claim:false` 노출. 문서를 고칠지 필터를 고칠지(백엔드) |
| D8 | `test_storage_roundtrip`의 Windows 실패(테스트 코드 결함)를 팀원에게 알리고 고칠지 |


## 5. 결정 항목 답 (프로젝트 자료로 판단, 2026-10-02)
| 번호 | 답 | 근거 |
|---|---|---|
| D1 | **현재 구현이 기준**(FR3-WMS+2F-85 1대 · Qwen3-8B · React `dashboard2`, G1은 별도 실험). 기획서의 UR5e·루시아·PWA 항목은 실패가 아니라 **기획서 갱신 대상** | 9-21 사용자 "FR3·UR5e 두 개지만 확정은 아님", 저장소에 UR5e 어댑터·프로필 없음, LLM 설정 `qwen3-8b-awq`, README "Gazebo 시뮬레이션 전용" |
| D2 | **기존 API로 받을 수 있는 값만 실제로 연결**(`/health` DB·실행 통계, `/v1/robots`·`/v1/config` 로봇 정보, `/v1/sim-demo` 자재 현황), 서버에 없는 칸은 "데이터 없음". 목록 API 추가(백엔드)는 하지 않음 | 2026-10-02 "백엔드는 수정하지 않는 쪽으로", 설계원칙 2(화면은 서버 값) |
| D3 | **표지 넣지 않음 유지** → UI-SIM-01은 실패가 아니라 기획서 ⑧ 갱신 대상. 지금 시뮬레이션 창은 실시간 관측이라 "실시간 아님" 문구 자체가 사실과 다름 | 2026-10-01 결정(메모리·반응형 문서), 재검토 조건(실기 연결 전·외부 공유·신규 운영자)일 때만 헤더 `SIM` 표시 재검토 |
| D4 | **"즉시 정지"**(사용자 결정 2026-10-02) — 헤더 버튼·시뮬레이션 창 문구 변경. 피그마 원본은 아직 "비상 정지" | 기획서 용어표 |
| D5 | **팀원 허락 뒤, 그 PC를 안 쓸 때 실행**(사용자 결정 2026-10-02). `e2e/api/robot.spec.js`는 기본 꺼짐 — `QA_ROBOT=1 npm run qa:api`로만 돌고 끝나면 자재를 되돌림 | — |
| D6 | **화면설계서 기능 전부 구현 대상**(권한·설정 3구역·진단 내보내기·기록 타임라인 포함). 데이터는 D2 범위(기존 API)로 채우고 없는 값은 "데이터 없음" | 기획서 ⑲·㉒ "전부 구현 대상이다", 메모리 "MVP 축소 대신 전체 구현" |
| D7 | **필터(백엔드)는 의도대로 동작, `AGENTS.md` 문구가 부정확** → 문서를 "정해진 6개 키"로 고치고 QA 테스트도 그 6개 키 기준으로 바꾼다. `real_hardware_claim:false`는 준비도가 아니라 "실기를 주장하지 않음" 선언 | `common.py` 주석·코드(정확히 일치하는 6개 키), 이 키는 GitHub에 없음 → **팀원 서버가 push 안 된 설정으로 실행 중**(팀원에게 알릴 것) |
| D8 | **팀원에게 알린다**(테스트 코드 결함: 연결을 닫기 전에 임시 폴더 삭제 → Windows 실패). 고치는 것은 백엔드라 팀원 몫 | 결함 자체는 확인됨, 백엔드 수정 안 함 원칙 |


## 6. 구현 뒤 남은 것 (c0e1374)
| 항목 | 상태 | 이유 |
|---|---|---|
| 로봇 동작 검사(TER-003·NFR-002·NFR-009) | 미실행 | 팀원 허락 뒤 `QA_ROBOT=1 npm run qa:api` (D5) |
| 버튼 입력 구분 기록(SFR-012 input_mode='button') | 부분 | 서버가 source를 text·stt_final만 받음 — 백엔드 변경 필요(수정 안 함) |
| 안전 신호 3종·안전 PLC·제어권(lease)·사용자 계정·도구 장착 이력·온도 | "확인 안 됨/수신 없음" 표시 | 서버에 값이 없음(D2 — 지어내지 않음) |
| 반응형 결정 #3(표 행 펼침)·#4(카드 열 수)·#5(DEMO 바 위치) | 미적용/일부 | 2026-09-30 기록상 "추천안 — 아직 확인받지 않음" |
| 피그마 원본 헤더 버튼 문구 | "비상 정지" 그대로 | 코드만 "즉시 정지"(D4) |
| 측정 지표(NFR-001·003·004·005·007·008, TER-005) | 미실행 | 작업 셀 PC에서 평가셋·자원 측정 필요 |

## 7. Codex 리뷰 반영 (2026-10-02)
Codex(gpt-6-sol, 읽기 전용) 지적 10건을 코드로 확인 → 10건 모두 사실(7번은 영향 작음, Codex 제안 `expires_at`은 PC 간 시계 차이로 틀려 `remaining_sec`를 요청 시각부터 세는 쪽으로).
회귀 테스트 `e2e/ui/review.spec.js`(UI-REV-01~09) + `voice.spec.js`(UI-REV-10). 수정 전 코드에서 8개 실패 확인(REV-02·03은 정지 경로 커버리지 보강, 원래도 통과).
전체: 69 통과 / 2 건너뜀(로봇 검사, D5) — 2회 연속.
