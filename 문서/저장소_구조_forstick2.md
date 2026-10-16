# forstick2 저장소 구조

대상: `kimpau0511/Fourstick` GitHub 저장소 (현재 원격 `master`, 로컬 `forstick2-review/`와 동일 상태).
아래 역할 설명은 전부 근거가 있다 — `CODE_RULES.md` 1번 표, 각 파일 docstring, `README.md`에서
그대로 가져왔다. 파일 목록은 `git ls-files` 실제 출력이다.

```
forstick2/
├── README.md                      # 실행방법·전제환경 (문서)
├── requirements-stt.txt           # STT 의존성 목록
├── forstick_uiux_prototype.html   # UI/UX 프로토타입 (단일 파일)
│
├── core/                          # [도메인] ROS/로봇/시뮬레이터를 모르는 순수 파이썬 모델
│   ├── task_plan.py                 # 작업 계획 데이터 구조
│   ├── reason_codes.py              # 실패·거부 사유 — 유일한 출처
│   ├── policy.py                    # 수치·정책 값(하드코딩 금지 원칙 강제)
│   ├── stop_contract.py             # 정지(STOP) 계약
│   ├── capability_profile.py        # 로봇 능력 프로파일
│   ├── robot_profile.py / skill_catalog.py / resource_catalog.py
│   ├── geometry.py / motion.py / frames.py / boundary.py
│   ├── execution_state.py / execution_result.py / termination.py
│   ├── grasp_observation.py / hardware_input.py / aperture_model.py
│   └── asset_manifest.py / provenance.py / constants.py
│
├── planning/                      # [AI 계획생성] STT → LLM → Task Plan 변환
│   ├── pipeline.py                  # 전체 파이프라인 조립
│   ├── plan_provider.py             # LLM provider 인터페이스
│   ├── vllm_provider.py             # vLLM 연동 구현체
│   ├── openai_compat.py             # OpenAI 호환 API 연동
│   ├── mock_provider.py             # 테스트용 mock
│   ├── slot_extractor.py            # 자연어 → 슬롯 추출
│   ├── prompt.py / output_parser.py / redaction.py
│   ├── attempt_runner.py / evaluation.py
│   └── workcell_command_eval.py
│
├── validation/                    # [안전검증] 판정 로직 전담, 로봇을 직접 움직이지 않음
│   ├── execution_permit.py          # 실행 허가 판정 (핵심)
│   ├── safety_validator.py          # 안전 규칙 검사
│   ├── geometry_check.py            # 기하학적 도달 가능성
│   ├── capability_precheck.py       # 능력 사전 점검
│   ├── hardware_readiness.py        # 하드웨어 준비 상태
│   ├── pick_place_gate.py / pick_place_plan.py / place_intent.py
│   ├── outcome_verifier.py          # 실행 결과 검증
│   ├── request_plan_consistency.py  # 요청↔계획 일치 검증
│   └── simulation_e2e.py / simulation_demo_state.py
│
├── robots/                        # [로보틱스/ROS] 로봇별 어댑터
│   ├── registry.py                  # 로봇 어댑터 등록
│   ├── base/                        # 어댑터 인터페이스(robot_adapter.py)
│   ├── fr3_gazebo/                  # FR3-WMS + Gazebo 시뮬레이터 어댑터
│   ├── moveit/                      # MoveIt2 연동
│   ├── hardware/                    # 실물 하드웨어 어댑터
│   └── fake/                        # 테스트용 가짜 어댑터
│
├── stt/                            # [음성인식] 백엔드 교체 가능한 STT 계층
│   ├── protocol.py                  # STT 백엔드 인터페이스
│   ├── faster_whisper_backend.py    # faster-whisper 구현체
│   ├── whisper_backend.py
│   ├── silero_vad_backend.py / vad.py   # 음성 구간 검출(VAD)
│   ├── streaming_session.py         # 스트리밍 세션 관리
│   ├── transcription_guard.py
│   └── evaluation.py
│
├── server/                         # [백엔드 API] HTTP/WebSocket 라우팅, 흐름 조립
│   ├── asgi.py                       # ★ ASGI 진입점 — `uvicorn --factory server.asgi:create_app`
│   ├── api.py                        # 요청 1건을 흐름(계획생성→검증→승인→허가→실행)으로 변환
│   ├── config.py                     # 서버 설정(ServerConfig)
│   ├── runtime.py                    # 런타임 조립(build_runtime)
│   ├── schemas.py                    # Pydantic 요청/응답 스키마
│   ├── scene_view.py                 # 장면(scene) 뷰 데이터
│   ├── sim_demo_commands.py / sim_demo_jobs.py
│   └── routes/                       # 경로별 핸들러
│       ├── common.py                   # 공통 유틸(EventHub, Response 등)
│       ├── planning.py / execution.py / robot.py
│       ├── scene.py / session.py / stt.py / sim_demo.py
│
├── storage/                        # [영속 계층] SQLite, 항상 repository.py를 거침
│   ├── repository.py                 # 저장소 접근 단일 창구
│   ├── records.py / mappers.py
│   └── sqlite/                       # SQLite 구현
│
├── html/                           # [프론트엔드] PWA — "Forstick · Robot Command Console"
│   ├── index.html                    # ★ 프론트엔드 진입점
│   └── static/                       # CSS/JS/아이콘
│
├── config/                         # [설정] 로봇·시뮬레이션·하드웨어 구성 파일
│   ├── loader.py
│   ├── gazebo/ · moveit/ · hardware/ · workcell/ · profiles/ · mounting/ · assets/
│
├── scripts/                        # [로보틱스 도구] Gazebo/MoveIt 실행·측정·검증 스크립트
│   ├── run_web_workcell.sh           # 웹 UI 실행 (README 기준 실행 순서 3번째)
│   ├── run_moveit_workcell.sh        # MoveIt2 실행 (2번째)
│   ├── run_gazebo_fr3_2f85_workcell_gui.sh  # Gazebo 작업 셀 (1번째)
│   ├── derive_*.py / measure_*.py    # 로봇 기구학·파지 포즈 도출
│   ├── verify_*.py / check_*.py      # 검증 스크립트군
│   └── demo_*.py/.sh                 # 시연용 스크립트
│   # 주의(CODE_RULES.md): 제품 코드에서 이 폴더를 import하지 않음 — 실행용, 라이브러리 아님
│
├── tests/                          # [테스트] 3단 분리
│   ├── unit/                         # 순수 로직, 관측값 주입으로 결정적 검증
│   ├── integration/                  # 여러 계층을 묶은 테스트
│   ├── contract/                     # 어댑터의 인터페이스 계약 준수 검증
│   ├── model/ · web/
│   └── run_all.py
│
├── examples/                       # 설정·STT·Task Plan 예시 데이터
└── fixtures/                       # 테스트/평가용 샘플 데이터(planning·stt·workcell_eval)
```

## 진입점·의존관계 (CODE_RULES.md 1번 근거)

- **진입점은 두 개**: 백엔드는 `server/asgi.py`(`uvicorn --factory server.asgi:create_app`), 프론트엔드는 `html/index.html`.
- **의존 방향은 한쪽으로만 흐른다**: `server` → `validation`/`planning`/`robots` → `core`. `core`는 다른 어떤 계층도 알지 못한다(원문 그대로).
- **`scripts/`는 제품 코드가 아니다**: 제품 코드에서 이 폴더를 import하지 않는다 — 실행용 스크립트일 뿐 라이브러리가 아니다(CODE_RULES.md 1번 규칙).
