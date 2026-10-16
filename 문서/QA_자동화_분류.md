# 포스틱 QA 자동화 분류 (2026-10-08)

- 기준: `기획서_템플릿(포스틱).md` 요구사항(SFR·NFR·TER·화면) + 10-02 이후 추가 기능
- 분류 원본: Codex(gpt-6-sol) 정적 조사 → Opus 교차 확인. 근거 파일:줄은 Codex 원본 표에 있다.
- 실행기: `Fourstick/dashboard2/e2e/run_qa.sh` — 결과는 `dashboard2/e2e-results/qa_summary.md`
- 이전 표: `QA_요구사항_추적표.md`(10-02) — "무엇을 검사하는가"는 그 표, "어떻게 자동으로 돌리는가"는 이 문서

## 등급과 실행 방식

| 등급 | 뜻 | 자동 실행 | 명령 |
|---|---|---|---|
| A | 자동 시험 있음, 서버 없이 돈다 | ✅ `run_qa.sh` 1단계 | `bash dashboard2/e2e/run_qa.sh` |
| B(읽기) | 작업 셀 서버에 읽기·확인 카드 생성/취소만 | ✅ `run_qa.sh server` 2단계 | `bash dashboard2/e2e/run_qa.sh server` |
| B·D(PC1) | 모델 서버(PC2 GPU)·MoveIt 필요, **로봇은 안 움직임**(코드 확인: `workcell_command_eval.py:13` "execute 호출 안 함") | ⚠ **평가기 고장** — 10-06 `a37909d`가 `Api.gate_for`에 `intent` 인자를 더했는데 `scripts/workcell_command_eval.py:170` 래퍼는 2개만 받아 모든 문장이 TypeError → BLOCK으로 기록된다(10-08 PC1 실행에서 확인, 서버 실제 판정은 allow). 래퍼를 `*rest`로 받게 하면 정상 채점됨(임시 사본으로 확인). 수정은 팀원 확인 뒤 | PC1에서 `./scripts/run_workcell_command_eval.sh --set sealed --provider real`(결과 폴더는 저장소 안이어야 함) |
| E | Gazebo 로봇을 실제로 움직임 | ⛔ 자동 실행 안 함(QA 결정 D5) | 팀원 허락 뒤 `QA_ROBOT=1 npm run qa:api`, `scripts/verify_*_e2e.py` |
| F | 사람 판단(현장 이해도·인터뷰) | 자동화 불가 | — |

## 요구사항별 등급 (요약)

| 등급 | 요구사항 |
|---|---|
| A | SFR-002·003·005·006·008·010·011·014, NFR-006, TER-001·002·006, 화면설계서 가~㉕ 전부(모의 응답 기준) |
| B·D(PC1) | SFR-001(실제 STT 모델), SFR-004(실제 LLM), SFR-007(MoveIt), NFR-001(p95 5초), NFR-003(GPU), NFR-004·005·008(봉인 평가셋), NFR-007(watchdog 실측), TER-005(봉인 210건) |
| E | SFR-009(실제 정지), SFR-013(가상 실행), NFR-002(정지 반응), NFR-009(실행 성공률), TER-003(E2E), TER-004(정지 반응 통합) |
| C → 이번에 자동화 | PWA(설치·API 비캐시·오프라인), 로그인 세션 만료·재로그인·다른 탭 로그아웃, 시뮬레이션 창 직접 열기·바깥 클릭 닫기 |
| 남은 C(기능 자체 없음) | SFR-012 터치 스킬 버튼 — 대시보드에 기능이 없어 시험 대상 없음 |
| F | 화면 실제 데이터에 대한 작업자 이해도 |

## 이번에 추가한 자동 시험

| 파일 | 시험 ID | 확인하는 것 |
|---|---|---|
| `e2e/ui/pwa.spec.js` | UI-PWA-01~06 | 매니페스트·아이콘 크기, dev에서 SW 미등록, 프리캐시 6개, `/v1/*`·`/health`·`/v1/auth/me` 비캐시, 오프라인 껍데기, 옛 캐시 삭제 |
| `e2e/ui/session-simwindow.spec.js` | UI-SESS-01~06, UI-SIMWIN-01·02 | 만료 창 위에서도 즉시 정지 요청 나감, 만료 중 명령 잠금(inert), 401 구분, 재로그인 복귀·거절, 다른 탭 로그아웃 전파, 창 바깥 닫기 시 정지 요청 없음, 패널에서 창 열기 |

## 서버 API 검사(2단계)

- 실서버는 **일반 모드**(`FORSTICK2_SIM_DEMO_COMMAND=0`, 팀 확인). API-05·06은 서버 모드를 보고 시연 경로 또는 `/v1/plan`으로 같은 요구를 검사한다(`1f2d16e`). 승인·실행 요청은 보내지 않는다.
- 주소: `FORSTICK2_BACKEND_URL=https://<운영 서버 주소> bash dashboard2/e2e/run_qa.sh server` — spec 기본값(옛 주소)은 닿지 않아 전부 "미실행"이 된다.

## 작업별 모델 배정

| 작업 | 담당 | 이유 |
|---|---|---|
| 요구사항 분류·근거 수집, 새 시험 리뷰 | Codex(gpt-6-sol, 읽기 전용) | 독립 시각의 교차 검증 |
| 자동화 설계, 실험 설계·원인 판단, 최종 보고 | Claude Opus | 판단이 필요한 일 |
| 명세가 분명한 시험 코드 작성 | Claude Sonnet(서브에이전트) | 단순 코딩은 Sonnet(메모리 원칙) |
| 시험 실행·결과 표 | 스크립트(모델 없음) | 반복 작업에 모델 불필요 |
