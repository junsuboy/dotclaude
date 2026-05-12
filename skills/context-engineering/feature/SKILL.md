---
name: context-engineering:feature
description: Feature 작업 폴더 (features/YYMMDD-<name>/) 생성 + 핵심 4 파일 초기화 + Feature cycle 안내. "구현/만들어/추가/기능" 키워드나 /ce-feature 슬래시로 호출.
---

# Context Engineering — Feature

Feature = 단일 기능 구현 단위. cycle: 20 → 21 → 10 ↔ INTEGRATION ↔ 21 → ... → 모두 통과 → 30 → 90.

## 동작 흐름

### 1. 호출 시 (M1)

이름 후보 추출 후 AskUserQuestion 으로 무조건 확인:
- type: Feature
- 폴더: features/YYMMDD-<name>/ (오늘 날짜 prefix)
- 첫 작업: 00-BRIEF 작성

옵션: A) 진행 / B) 폴더명·type 변경 / C) 취소

### 2. 폴더 존재 검사

이미 같은 이름 폴더 있으면:
- AskUserQuestion: 이어서 진행 / 새 Feature 로 (다른 이름) / 취소
- "이어서" 시: 11-HISTORY 마지막 N줄 + 10-WORK-LOG 최근 ADR + 90-SUMMARY 의 검증 상태 → 요약 후 다음 지시 받기

### 3. Project 컨텍스트 감지

호출 시점의 CWD 또는 상위 경로에 `00-VISION.md` + `01-DECOMPOSITION.md` 가 있으면 **Project 내부 호출**로 판단:

- 폴더 위치: `projects/<project>/features/@<name>/` (Project 안)
- vs. 단독 feature: `features/YYMMDD-<name>/` (기존)

Project 내부면 추가 동작:
- sibling `02-SPEC/@<name>/SPEC.md` 존재 검사
- 있으면 `00-BRIEF.md` frontmatter 에 `spec: ../../02-SPEC/@<name>/SPEC.md` 자동 추가
- SPEC §3 (Acceptance criteria) 를 `00-BRIEF.md` 의 AC 섹션에 복사 (편집은 SPEC 쪽에서, BRIEF 는 참조용 사본 명시)
- SPEC 의 `type` 태그를 `00-BRIEF.md` frontmatter 에 `type: <ui|api|data|infra|integration|tooling>` 으로 기록

SPEC 없으면 standalone 동작 (단독 feature 처럼).

### 4. 초기 파일 생성 (M2: 핵심 4 파일만)

- `00-BRIEF.md` — 빈 템플릿 (Project 내부면 SPEC 참조·type 태그 주입)
- `10-WORK-LOG.md` — 빈 템플릿
- `11-HISTORY.md` — 빈 템플릿 + 첫 entry "create | <폴더>"
- `90-SUMMARY.md` — 빈 템플릿

나머지 (01, 02, 20, 21, 30) 는 lazy: phase 진입 시 생성.

### 5. Feature type 태그

Project 내부 호출이면 SPEC 의 type 을 그대로 사용. 단독 호출이면:
- 사용자에게 묻기 (AskUserQuestion: ui/api/data/infra/integration/tooling)
- 또는 type 분류 생략 (BRIEF frontmatter `type: untyped`)

향후 type 별 cycle 변형 (예: `data` 는 마이그레이션 검증 단계 추가) 의 기반.

### 6. 다음 단계

00-BRIEF 본문 작성을 위해 사용자에게:
- Project 내부: "SPEC 기반 BRIEF 초안이 채워졌습니다. 검토 후 20-SCENARIO 로 진입?"
- 단독: "00-BRIEF 채우기 시작? `superpowers:brainstorming` 도 호출할까?"

### 7. Project 11-HISTORY 동기화

Project 내부면 feature 생성·종료 시 상위 Project 의 `11-HISTORY.md` 에도 한 줄 append:

```
- HH:MM | start feature @<name> | features/@<name>
- HH:MM | complete feature @<name> | close ✅
```

## Cycle (참조)

```
0X → 20 → 21 → 10 ↔ INTEGRATION ↔ 21 → ... → 30 → 90
```

10 의 결정은:
- 시나리오 변경 → 20 으로
- 코드/설계 변경 → INTEGRATION → 21 재검증

## Type 별 Cycle hook (경량)

`type` frontmatter 값에 따라 cycle 의 특정 시점에서 추가 동작·산출물 제안. 강제 X — 사용자가 skip 가능.

| Type | Hook 시점 | 추가 산출물 / 동작 | 강도 |
|------|-----------|-------------------|------|
| `data` | 20-SCENARIO 작성 직후 | `21-MIGRATION.md` 생성 제안 — DDL diff, dry-run plan, rollback 시나리오, lock 영향 추정 | 자동 제안 |
| `ui` | 00-BRIEF 확정 직후 | `frontend-design` skill 호출 제안 (디자인 안 도출), 결정 시 `10-WORK-LOG` 에 디자인 ADR 추가 | 자동 제안 |
| `api` | SPEC §4b 또는 20-SCENARIO 작성 시 | request/response schema 명시 강제 + 21 에 contract test 케이스 (정상·400·401·5xx) 포함 | enforce |
| `infra` | 20-SCENARIO 작성 직후 | `21-ROLLBACK.md` 생성 제안 — 롤백 명령·검증·blast radius | 자동 제안 |
| `integration` | 20-SCENARIO 작성 시 | stub 모드 / real 모드 분리 명시, sandbox 검증 항목 21 에 포함 | enforce |
| `tooling` | 21-TEST_RESULTS 작성 시 | self-use 섹션 추가 — 만든 도구를 직접 1회 이상 사용한 결과 기록 | 자동 제안 |
| `untyped` | — | hook 없음 | — |

호출 정책:
- "자동 제안" — feature skill 이 해당 시점에서 AskUserQuestion 으로 묻기
- "enforce" — 사용자 동의 없이도 산출물 섹션을 템플릿에 자동 추가 (작성은 사용자)
- 모든 hook 은 추가 phase 가 아니라 기존 phase 의 섹션·파일 boost. cycle 자체는 동일.

## Auto vs Ask 정책

| 시점 | 묻기 |
|------|------|
| skill 호출 직전 | ✅ 무조건 |
| phase 전환 | ✅ |
| 10/11 분류 애매 | ✅ |
| 10 ADR 명백 | ❌ 자동 |
| 11-HISTORY append | ❌ 자동 |
| 21 append | ❌ 자동 |
| close | ✅ |

## 11-HISTORY append 트리거 (자동)

- 파일 수정 / commit·push / 외부 명령 / 계획 변경 / 검증 수행 / ADR 추가 / 세션 종료 직전 요약

양식: `HH:MM | <action> | <reference>`

## 외부 동기화

`/ce-export` 또는 `/ce-import` 로 Jira/Confluence 연동. 상세는 `context-engineering:export` skill.

## 참조

- 전체 설계 §2.3, §4, §5: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- Templates: `./templates/`
