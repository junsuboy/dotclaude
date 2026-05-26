---
name: context-engineering:integration-test
description: 에픽 (또는 임의 대상군) 산하 여러 Feature 의 실기기/실환경 통합 검증 작업 폴더 (integration-tests/YYMMDD-[EPIC]-<name>/) 생성 + 시나리오/증거 로그 템플릿 초기화 + 후조치 프로토콜 안내. "통합테스트/integration test/실기기 검증/에픽 검증" 키워드나 /ce-integration-test 슬래시로 호출. Feature 가 아님 — 여러 Feature 를 한 번에 검증하는 별도 작업 단위.
---

# Context Engineering — Integration Test

Integration Test = 이미 구현된 N개 Feature 를 **실기기/실환경에서 한 번에 검증**하는 작업 단위. Feature 가 아니라 Feature 들의 **검증 결과물**.

핵심 산출물 3종: `00-SCENARIO.md` (계획·후조치 프로토콜·INT 스텝) + `01-EVIDENCE-LOG.md` (실행·결과·커버리지·후조치) + `evidence/` (원본 증거).

## 언제 호출하는가

- 에픽 (예: PROJ-1175) 산하 다수 Feature 가 dev 완료 → QA 인수 전 **통합 동작 확인** 필요
- 단일 Feature 가 아니라 **여러 Feature 가 함께 동작**하는 시나리오 검증
- 실기기/스테이징/폐쇄망 등 **단위 테스트로는 못 잡는 결함** 발견 목적
- 후조치 발생 시 **현장 핫픽스 → 형상 반영 → 재검증** 사이클 추적 필요

> Feature 가 아닌 이유: 신규 코드 산출이 아니라 기존 N개 Feature 검증. `~/docs/integration-tests/` 에 격리.

## 동작 흐름

### 1. 호출 시 (M1 — 무조건 확정 받기)

이름·에픽 후보 추출 후 AskUserQuestion 으로 확인:
- type: Integration Test
- 폴더: `integration-tests/YYMMDD-[<EPIC-KEY>]-<name>/` (오늘 날짜 prefix, JIRA 에픽 키 마커)
- 시작: `00-SCENARIO` 골격 작성

옵션: A) 진행 / B) 폴더명·에픽 변경 / C) 취소

> **JIRA 에픽 키 anchor 정책** (외부 동기화 시): 에픽 키 없이는 `/ce-export` 불가. 호출 시 에픽 키가 모호하면 `unknown` 으로 두고 추후 `/ce-import` 또는 수동 보강.

### 2. 폴더 존재 검사

이미 같은 이름 폴더 있으면:
- AskUserQuestion: 이어서 진행 / 새 통합테스트로 (다른 이름) / 취소
- "이어서" 시: `11-HISTORY.md` 마지막 N줄 + `01-EVIDENCE-LOG.md` 의 마감 체크리스트 + 후조치 표 → 요약 후 다음 지시 받기

### 3. 초기 파일 생성 (M2)

산출물 위치: **`~/docs/integration-tests/YYMMDD-[<EPIC>]-<name>/`** (고정 — 위치 묻지 않음)

```
integration-tests/YYMMDD-[EPIC]-<name>/
├── 00-SCENARIO.md       ← 템플릿 복사 (대상 Feature 표·토폴로지·후조치 프로토콜·INT 스텝)
├── 01-EVIDENCE-LOG.md   ← 템플릿 복사 (실행정보·사전점검·결과표·커버리지·부수발견·후조치·마감)
├── 11-HISTORY.md        ← 빈 템플릿 + 첫 entry "create | <폴더>"
├── 90-SUMMARY.md        ← 마감 시 작성 (lazy)
└── evidence/            ← 빈 디렉터리 (증거 파일 수집처)
```

> 10-WORK-LOG 는 기본 생성 X. 통합테스트의 결정은 보통 후조치 표에 흡수됨. 필요 시 lazy 생성.

### 4. 시나리오 작성 가이드 (사용자)

`00-SCENARIO.md` 의 6개 섹션을 순서대로 채움:
1. **대상 Feature 표** — 티켓·요약·PR·매핑할 INT 번호 미리 예약 (INT-XX 칸은 §5 작성 후 확정)
2. **토폴로지** — ASCII 다이어그램 + 환경·네트워크 가정 + 경로 규약
3. **후조치 프로토콜** — 템플릿 그대로 (5단계 — 수정 X)
4. **증거 규약** — 템플릿 그대로 (파일명·tee 패턴)
5. **실행 시나리오** — Phase 별 INT-XX (절차/기대/성공증거 3개 항목 필수)
6. **완료 조건** — 템플릿 그대로

시나리오는 사용자가 직접 작성 (JIRA 자동 끌어오기 X). 외부 자료가 필요하면 `/ce-import <에픽키>` 별도 호출.

### 5. 실행 흐름 (사용자 + Claude 협업)

```
사전 점검 (정적·기기 불필요)
   └─ 16개 코드 포인트 grep 등 → 01-EVIDENCE-LOG 사전점검 표
        ↓
환경 준비 (클린 슬레이트)
   └─ 삭제 대상만 명시 제거 (autoremove 금지)
        ↓
Phase 0 → Phase 1 → ... → Phase N
   └─ 각 INT-XX 실행 → 증거 tee → 01-EVIDENCE-LOG 결과표 + 상태 마커 갱신
        ↓
FAIL 발생 시 → 후조치 프로토콜 (5단계)
   ①실패확정 → ②현장핫픽스 → ③형상반영(PR머지) → ④재검증 → ⑤문서동기화
        ↓
Feature 커버리지 매트릭스 검증 (매핑 INT 전부 PASS?)
   └─ 부수 발견 정리 (대상 외 발견 → 별도 티켓 권고)
        ↓
마감 — 90-SUMMARY 작성 + 각 Feature 폴더 30-QA_HANDOFF/90-SUMMARY 갱신
```

### 6. 상태 마커 (`01-EVIDENCE-LOG` 결과표)

| 마커 | 의미 |
|------|------|
| ⬜ | 미실행 |
| 🟡 | 진행 중 |
| ✅ | PASS |
| ❌ | FAIL (후조치 미진행) |
| 🔁 | 후조치 후 재PASS (②현장+③형상 모두 기록됨) |

> ❌ → 🔁 전환 조건: 후조치 표에 ②현장핫픽스 + ③형상반영 양쪽 모두 기록 + ④재검증 PASS 증거.

## 후조치 프로토콜 (강제 — 5단계)

`00-SCENARIO.md §3` 참조. 핵심:

> **현장 수정으로 끝내지 않는다.** 현장에서 핫픽스로 통과시켜도 그 변경이 원천 소스에 반영되지 않으면 다음 빌드에서 재발. ②현장 + ③형상 모두 기록돼야 INT-XX = PASS (🔁).

각 후조치는 `01-EVIDENCE-LOG` 후조치 표의 한 행. 해당 Feature 의 `11-HISTORY` 에도 PR 링크 한 줄 동기화 권장.

## 부수 발견 (대상 외)

통합테스트 중 **검증 범위(대상 N개 Feature) 밖**에서 발견된 결함·노이즈는 `01-EVIDENCE-LOG` 부수 발견 표에 별도 기록.

- 즉시 고치지 않음 (스코프 외)
- 영향·권고 명시 → 별도 JIRA 티켓 권고
- 90-SUMMARY 후속 권고 섹션에 요약

## Auto vs Ask 정책

| 시점 | 묻기 |
|------|------|
| skill 호출 직전 | ✅ 무조건 |
| 폴더 존재 (이어서/새로/취소) | ✅ |
| Phase 시작·종료 | ✅ |
| 후조치 진입 (FAIL 발생) | ✅ |
| 후조치 ③ 형상 반영 PR 머지 권한 행동 | ✅ |
| 마감 체크리스트 확정 | ✅ |
| 11-HISTORY append | ❌ 자동 |
| 01-EVIDENCE-LOG 결과표 상태 마커 갱신 | ❌ 자동 |
| evidence/ 파일 추가 | ❌ 자동 |

## 11-HISTORY append 트리거 (자동)

- 사전 점검 결과 / Phase 전환 / INT 스텝 PASS·FAIL / 후조치 단계 진행 / PR 머지 / 재검증 결과 / 세션 종료 직전 요약

양식: `HH:MM | <action> | <reference>`

## 폴더 명명 규약

`integration-tests/YYMMDD-[<EPIC-KEY>]-<짧은이름>/`

예:
- `260520-[PROJ-1175]-onpremise-v1.1/` — 에픽 키 있음
- `260601-[unknown]-payment-flow/` — 에픽 키 아직 미정 (추후 `/ce-import` 로 보강)

> JIRA 에픽 키는 외부 동기화 anchor. 매핑된 Feature 티켓은 `00-SCENARIO §1` 표에 나열.

## 외부 동기화

- `/ce-export <폴더>` — 마감 후 에픽 JIRA 코멘트로 결과 요약 + 90-SUMMARY 첨부 (Confluence 옵션)
- `/ce-import <에픽키>` — 에픽 description / 자식 이슈 목록을 `00-SCENARIO §1` 의 시드로 가져오기

`context-engineering:export` / `context-engineering:import` skill 참조.

## 다른 type 과 차이

| 차이점 | Feature | Integration Test |
|--------|---------|------------------|
| 산출물 본질 | 신규 코드·문서 | 기존 N개 Feature 의 **검증 결과** |
| 폴더 위치 | `~/docs/features/` | `~/docs/integration-tests/` |
| 핵심 파일 | 00-BRIEF / 10-WORK-LOG / 90-SUMMARY | 00-SCENARIO / 01-EVIDENCE-LOG / evidence/ |
| 실패 처리 | 해당 feature 내부 ADR·재시도 | **후조치 5단계** (현장 → 형상 → 재검증) |
| 외부 anchor | feature JIRA 키 | **에픽** JIRA 키 |
| 종료 조건 | feature AC 통과 | **모든 INT PASS + 매핑된 Feature 커버리지 100%** |

> Project 의 P6 Handoff 와는 별개. Project Handoff 는 dev→QA 인수 문서(50~80줄), Integration Test 는 실기기 결과의 트레이스/증거 컨테이너. 큰 Project 라면 P6 직전에 Integration Test 폴더를 1개 더 만들어 병행 가능.

## 참조

- 전체 설계: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- 변경 이력: `~/.claude/skills/context-engineering/CHANGELOG.md`
- 모범 사례: `~/docs/integration-tests/260520-[PROJ-1175]-onpremise-v1.1/`
- Templates: `./templates/`
