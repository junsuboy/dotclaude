---
name: context-engineering:project
description: 한 장 기획서 (00-VISION) → Feature 분해 → SPEC → PLAN → 실행 → QA 인수 의 end-to-end orchestrator. 여러 Feature 묶음을 다루는 Project 작업 (projects/YYMMDD-<name>/) 시작·관리. "프로젝트/여러 기능/통합 개발/기획" 키워드나 /ce-project 슬래시로 호출.
---

# Context Engineering — Project

Project = 기획서 한 장에서 시작해 Feature 를 분해·명세·계획·구현하는 오케스트레이터. QA 직전 (`91-HANDOFF`) 까지 책임. 단순 컨테이너가 아님.

## Phase 모델 (P1 ~ P6)

| Phase | 산출물 | 책임 | Agent / Skill |
|-------|--------|------|---------------|
| **P1 Vision** | `00-VISION.md` | 한 장 기획서 작성 | 사용자 직접 OR `superpowers:brainstorming` |
| **P2 Decomposition** | `01-DECOMPOSITION.md` | Feature 후보 + type + DAG | `architect` agent |
| **P3 Spec** | `02-SPEC/@<name>/SPEC.md` | Feature 별 명세 (AC·계약·엣지) | `spec-writer` agent (feature 마다 호출) |
| **P4 Plan** | `03-PLAN.md` | 실행 순서·마일스톤·리스크 | `architect` agent |
| **P5 Execute** | `features/@<name>/` | 각 Feature 구현 | `/ce-feature` (SPEC 자동 주입) |
| **P6 Handoff** | `91-HANDOFF.md` | QA 인수 (50~80줄) | 사용자 + 자동 초안 |

## 외부 skill 호출 매트릭스

기존 superpowers / frontend-design 자산을 재활용. Project skill 이 직접 invoke.

| 시점 | 호출 skill | 동작 | 자동 vs 제안 |
|------|-----------|------|-------------|
| P1 진입, VISION 이 비어있을 때 | `superpowers:brainstorming` | Vision 항목 (problem/user/metric/scope) 채우기 대화 | **제안** (사용자가 직접 쓸 수도) |
| P1 → P2 직전 VISION 검증 | `superpowers:brainstorming` | 가정·non-goal·open question rubber duck | 제안 |
| P2 직전 architect 호출 전 | `superpowers:writing-plans` (옵션) | 큰 Project 면 plan-first | 제안 |
| P3 SPEC 모두 confirmed → P4 직전 | (없음 — architect 가 처리) | — | — |
| P5 각 feature `90-SUMMARY` close 직전 | `superpowers:requesting-code-review` | feature 단위 리뷰 받고 결과 반영 후 close | **자동 트리거 + 사용자 확정** |
| P5 batch 병렬 실행 시 | `superpowers:dispatching-parallel-agents` | 독립 feature 동시 작업 | 제안 |
| P6 HANDOFF 작성 직전 | `superpowers:verification-before-completion` | "완료" 라고 적기 전 검증 | **자동 enforce** |

호출 정책:
- "자동 트리거" = Project skill 이 해당 시점에 도달하면 사용자에게 묻지 않고 invoke (사용자는 결과만 검토)
- "제안" = AskUserQuestion 으로 "이 skill 호출할까요?" 묻기
- 사용자가 명시적으로 "skill 없이" 라고 하면 skip

## 폴더 구조

```
projects/YYMMDD-<name>/
├── 00-VISION.md
├── 01-DECOMPOSITION.md
├── 02-SPEC/
│   ├── @auth-google/SPEC.md
│   └── @profile-ui/SPEC.md
├── 03-PLAN.md
├── 10-WORK-LOG.md         ← Project 레벨 ADR (cross-feature 결정)
├── 11-HISTORY.md          ← Project 레벨 history (feature 시작/종료 단위)
├── features/
│   ├── @auth-google/      ← /ce-feature 가 생성 (SPEC 자동 로드)
│   └── @profile-ui/
├── 90-SUMMARY.md
└── 91-HANDOFF.md          ← QA 직전 인수 문서
```

## 동작 흐름

### 1. 호출 (P0 — 진입)

AskUserQuestion 으로 확정:
- type: Project
- 폴더: `projects/YYMMDD-<name>/`
- 시작 phase: **P1 (새 기획)** | **P2~ (기획 이미 있음)**

폴더 존재 시: 이어서 / 새로 / 취소 분기.

### 2. P1 Vision

초기 파일 생성 (lazy — 필요한 것만):
- `00-VISION.md` (템플릿)
- `10-WORK-LOG.md`
- `11-HISTORY.md` (`HH:MM | create project | <폴더>`)
- `90-SUMMARY.md`

사용자에게:
- "기획 직접 작성?" vs "`superpowers:brainstorming` 호출?"
- VISION 작성 완료 시 사용자 확정 → P2 진입 제안

### 3. P2 Decomposition

`architect` agent 호출:
```
Agent(subagent_type: "architect",
      prompt: "Decompose 00-VISION.md into Features.
               Vision path: <abs path>
               Output to: 01-DECOMPOSITION.md
               Return the proposal Markdown.")
```

결과를 `01-DECOMPOSITION.md` 에 저장 → 사용자 확정 → `status: confirmed`.

### 4. P3 Spec (feature 마다)

01-DECOMPOSITION 의 각 Feature 에 대해 `spec-writer` 호출:
```
Agent(subagent_type: "spec-writer",
      prompt: "Write SPEC for @<name>.
               Type: <type>
               Scope: <one-liner>
               Vision: <path>
               Decomposition: <path>
               Output to: 02-SPEC/@<name>/SPEC.md")
```

병렬 dispatch 권장 (`superpowers:dispatching-parallel-agents`).
모든 SPEC `draft` → 사용자 리뷰 → `confirmed` 일괄 전환.

### 5. P4 Plan

`architect` 재호출:
```
Agent(subagent_type: "architect",
      prompt: "Build execution plan.
               Read: 01-DECOMPOSITION.md, 02-SPEC/*/SPEC.md
               Output: 03-PLAN.md
               Include: batches, milestones, risks, done criteria.")
```

### 6. P5 Execute

각 Feature 별로 `/ce-feature @<name>` 호출. **반드시 Project 폴더 내에서 호출** — Project 컨텍스트 인식 시 feature skill 이 자동으로:
- `features/@<name>/` 폴더 생성
- 해당 `02-SPEC/@<name>/SPEC.md` 를 `00-BRIEF.md` 의 입력으로 자동 참조
- feature type 태그 처리 (type 별 추가 단계 — future)

병렬 가능 batch 는 사용자에게 권장.

### 7. P6 Handoff

모든 Feature `90-SUMMARY` close 후:
- `91-HANDOFF.md` 초안 자동 생성 (각 SPEC §3 AC + 90-SUMMARY 의 변경 요약 합성)
- 사용자 검수 → `status: ready-for-qa`
- 11-HISTORY 에 `handoff to QA` append

## Sub-feature 라우팅 (P5 내부)

Project 안에서 `/ce-feature @<name>` 호출 시 feature skill 의 동작:
- 폴더: `projects/<project>/features/@<name>/` (CWD 가 Project 폴더면 자동 인식)
- `00-BRIEF.md` 초기화 시 sibling `02-SPEC/@<name>/SPEC.md` 존재하면 frontmatter 에 `spec: ../../02-SPEC/@<name>/SPEC.md` 추가 + AC 섹션 자동 복사
- feature 의 11-HISTORY append 는 자기 폴더로, Project 의 11-HISTORY 에는 `start/complete feature @<name>` 만 누적

## Auto vs Ask

| 시점 | 묻기 |
|------|------|
| Project 생성 직전 | ✅ 무조건 |
| Phase 전환 (P1→P2 등) | ✅ 무조건 |
| Feature 후보 확정 (P2 결과) | ✅ |
| SPEC 확정 (P3 결과, feature 마다) | ✅ |
| Plan 확정 (P4 결과) | ✅ |
| Feature 실행 순서 (P5 batch 시작) | ✅ |
| HANDOFF 초안 확정 (P6) | ✅ |
| 11-HISTORY append | ❌ 자동 |
| 10 ADR (명백할 때) | ❌ 자동 |

## 11-HISTORY 양식 (Project 레벨)

```
## YYYY-MM-DD
- HH:MM | create project | projects/260512-checkout
- HH:MM | P1 → P2 | architect dispatched
- HH:MM | decomposition confirmed | 5 features
- HH:MM | start feature @auth-google | features/@auth-google
- HH:MM | complete feature @auth-google | close ✅
- HH:MM | handoff to QA | 91-HANDOFF ready
```

## 호환성 (기존 Project 와)

기존에 단순 컨테이너로 만들어진 projects/ 폴더 (00-OVERVIEW 만 있는):
- 그대로 둠. 새 phase 모델 강제 X
- 새 작업부터 적용 — 기존 폴더 마이그레이션 옵션은 `/ce-modify` 로 별도 처리

`00-OVERVIEW.md` 템플릿은 deprecated (`00-VISION.md` 로 대체). 기존 폴더에 `00-OVERVIEW` 있으면 그대로 인정.

## 참조

- 전체 설계 §2.4, §6: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- Templates: `./templates/`
- Agents: `~/.claude/agents/architect.md`, `~/.claude/agents/spec-writer.md`
