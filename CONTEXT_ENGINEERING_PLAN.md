# Context Engineering — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Context Engineering skill 시스템 빌드. 4 task type (Feature/Project/Research/QnA) 의 산출물 폴더 구조·workflow·meta skill (modify/export) 을 `~/.claude/skills/` 에 구현하고, CLAUDE.md 통합 + git 관리 + data-retention 첫 이전.

**Architecture:** 모든 skill 은 `~/.claude/skills/context-engineering/` 네임스페이스. type 별 subfolder (SKILL.md + templates/). meta skill (modify, export) 도 같은 네임스페이스. 슬래시 명령은 `~/.claude/commands/`. CLAUDE.md 가 자동 분류 가이드. ~/.claude/ 전체 git 관리 (filtered).

**Tech Stack:** Markdown (skill·template·doc), Bash (init·migration script), git, Claude Code skill·slash-command 시스템.

---

## 파일 구조 (생성 / 수정)

```
~/.claude/
├── .git/                                      [Task 0] git init
├── .gitignore                                 [Task 0] 신규
├── CLAUDE.md                                  [Task 10] 수정 (CE 섹션 추가)
├── CONTEXT_ENGINEERING_DESIGN.md              (이미 존재, git 추적)
├── CONTEXT_ENGINEERING_PLAN.md                (이 문서, git 추적)
├── commands/
│   ├── ce-feature.md                          [Task 8] 신규 슬래시
│   ├── ce-project.md                          [Task 8]
│   ├── ce-research.md                         [Task 8]
│   ├── ce-qna.md                              [Task 8]
│   ├── ce-modify.md                           [Task 8]
│   ├── ce-export.md                           [Task 8]
│   └── ce-import.md                           [Task 8]
└── skills/
    └── context-engineering/
        ├── SKILL.md                           [Task 1] 진입점 (분기)
        ├── CHANGELOG.md                       [Task 9] 빈 초기 entry
        ├── feature/
        │   ├── SKILL.md                       [Task 2]
        │   └── templates/                     [Task 2]
        │       ├── 00-BRIEF.md
        │       ├── 01-RESEARCH.md
        │       ├── 02-DESIGN.md
        │       ├── 10-WORK-LOG.md
        │       ├── 11-HISTORY.md
        │       ├── 20-TEST_PLAN.md
        │       ├── 21-TEST_RESULTS.md
        │       ├── 30-QA_HANDOFF.md
        │       └── 90-SUMMARY.md
        ├── project/
        │   ├── SKILL.md                       [Task 3]
        │   └── templates/                     [Task 3]
        │       ├── 00-OVERVIEW.md
        │       ├── 10-WORK-LOG.md
        │       ├── 11-HISTORY.md
        │       └── 90-SUMMARY.md
        ├── research/
        │   ├── SKILL.md                       [Task 4]
        │   └── templates/                     [Task 4]
        │       ├── 00-BRIEF.md
        │       ├── 02-DESIGN.md
        │       ├── 10-WORK-LOG.md
        │       ├── 11-HISTORY.md
        │       ├── 20-METHOD.md
        │       ├── 21-FINDINGS.md
        │       └── 90-SUMMARY.md
        ├── qna/
        │   ├── SKILL.md                       [Task 5]
        │   └── templates/
        │       └── QnA.md                     [Task 5]
        ├── modify/
        │   └── SKILL.md                       [Task 6]
        └── export/
            └── SKILL.md                       [Task 7] (import 포함)
```

data-retention 이전 [Task 11]:
- 현재: `/Users/junsuboy/docs/260508-data-retention/`
- 신규 구조 적용 (이전 시 폴더 위치는 사용자 결정 — 그 자리 그대로 또는 project repo 안 `.claude-contexts/`)

---

## Task 0: ~/.claude/ git baseline

**Files:**
- Create: `~/.claude/.gitignore`
- Init: `~/.claude/.git/`

**Steps:**

- [ ] **Step 1: git init**
```bash
cd ~/.claude && git init
```
Expected: `Initialized empty Git repository in /Users/junsuboy/.claude/.git/`

- [ ] **Step 2: Write .gitignore**

```
# Runtime state — 추적 안 함
cache/
sessions/
shell-snapshots/
file-history/
tasks/
telemetry/
statsig/
backups/
debug/
paste-cache/
session-env/

# Logs / history
history.jsonl
*.log

# Cache files
*-cache.json
mcp-needs-auth-cache.json
stats-cache.json

# Local override / sensitive
settings.local.json

# Plugin cache (외부 관리)
plugins/cache/

# OS / editor
.DS_Store
*.swp
```

`~/.claude/.gitignore` 위 내용으로 생성.

- [ ] **Step 3: 초기 status 확인**
```bash
cd ~/.claude && git status --short | head -30
```
Expected: CLAUDE.md, CONTEXT_ENGINEERING_DESIGN.md, settings.json, commands/init-serena.md, projects/, MEMORY 등 untracked 표시. cache/, sessions/ 등은 보이면 안 됨.

- [ ] **Step 4: 초기 commit**
```bash
cd ~/.claude && git add -A && git commit -m "chore: initial baseline (Context Engineering 빌드 직전)"
```
Expected: `[main (root-commit) ...] chore: ...` 다수 파일 added.

- [ ] **Step 5: branch 확인**
```bash
cd ~/.claude && git log --oneline
```
Expected: 1개 commit.

---

## Task 1: Directory + Entry-point SKILL.md

**Files:**
- Create: `~/.claude/skills/context-engineering/SKILL.md`

**Steps:**

- [ ] **Step 1: 디렉토리 생성**
```bash
mkdir -p ~/.claude/skills/context-engineering/{feature/templates,project/templates,research/templates,qna/templates,modify,export}
```

- [ ] **Step 2: 진입점 SKILL.md 작성**

`~/.claude/skills/context-engineering/SKILL.md`:

```markdown
---
name: context-engineering
description: 새 작업을 4 task type (Feature/Project/Research/QnA) 중 하나로 분류하고 산출물 폴더 + 핵심 파일을 생성한다. 사용자가 "구현/만들어/추가/조사/비교/프로젝트 시작/물어볼 게" 같은 요청을 하면 호출.
---

# Context Engineering — 진입점

새 작업을 시작할 때 4 type 중 어디에 속하는지 분류 후 해당 type skill 로 dispatch.

## Type 분류 가이드

| Type | 트리거 키워드 | 슬래시 | 다음 skill |
|------|--------------|--------|-----------|
| Feature | 구현 / 만들어 / 추가 / 기능 | `/ce-feature <name>` | `context-engineering:feature` |
| Project | 프로젝트 / 여러 기능 / 통합 개발 | `/ce-project <name>` | `context-engineering:project` |
| Research | 조사 / 비교 / 검토 (코드 변경 X) | `/ce-research <topic>` | `context-engineering:research` |
| QnA | 단순 질문 / 물어볼 게 | `/ce-qna <topic>` | `context-engineering:qna` |

## 동작

1. 사용자 입력에서 type + 이름 후보 추출
2. **무조건 AskUserQuestion 으로 확정 받기** (확정 전 어떤 파일·폴더 생성 X)
3. 확정된 type 의 sub-skill 로 dispatch

## 모호한 경우

키워드 매칭 약하면:
```
AskUserQuestion("어떤 작업 유형인가요?",
  A) Feature — 기능 구현
  B) Project — 여러 기능 묶음
  C) Research — 조사 (코드 변경 X)
  D) QnA — 단순 질문)
```

## 핵심 원칙

- 사용자 확인 무조건 받기 (호출 직전·phase 전환·큰 갈림길)
- 11-HISTORY append 는 자동 (의미 있는 모든 액션 기록)
- 10/11 분류 애매하면 묻기
- 폴더명: `YYMMDD-<name>` + 외부 링크 마커 `[CONF]`/`[JIRA]` (알파벳 순)

## 참조

- 전체 설계: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- 변경 이력: `~/.claude/skills/context-engineering/CHANGELOG.md`
```

- [ ] **Step 3: 디렉토리 구조 확인**
```bash
find ~/.claude/skills/context-engineering -type d | sort
```
Expected: 6개 type/메타 dir + 4개 templates dir.

- [ ] **Step 4: commit**
```bash
cd ~/.claude && git add skills/context-engineering/SKILL.md && \
  git commit -m "feat(skill): context-engineering 진입점 SKILL.md"
```

---

## Task 2: Feature skill + 9 templates

**Files:**
- Create: `~/.claude/skills/context-engineering/feature/SKILL.md`
- Create: `~/.claude/skills/context-engineering/feature/templates/00-BRIEF.md`
- Create: 〃 `/01-RESEARCH.md`, `/02-DESIGN.md`, `/10-WORK-LOG.md`, `/11-HISTORY.md`, `/20-TEST_PLAN.md`, `/21-TEST_RESULTS.md`, `/30-QA_HANDOFF.md`, `/90-SUMMARY.md`

**Steps:**

- [ ] **Step 1: feature/SKILL.md 작성**

```markdown
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

### 3. 초기 파일 생성 (M2: 핵심 4 파일만)

- `00-BRIEF.md` — 빈 템플릿
- `10-WORK-LOG.md` — 빈 템플릿
- `11-HISTORY.md` — 빈 템플릿 + 첫 entry "create | <폴더>"
- `90-SUMMARY.md` — 빈 템플릿

나머지 (01, 02, 20, 21, 30) 는 lazy: phase 진입 시 생성.

### 4. 다음 단계

00-BRIEF 본문 작성을 위해 사용자에게:
- "00-BRIEF 채우기 시작? brainstorming skill 도 호출할까?"
- 큰 feature 면 `superpowers:brainstorming` 으로 진입 유도

## Cycle (참조)

```
0X → 20 → 21 → 10 ↔ INTEGRATION ↔ 21 → ... → 30 → 90
```

10 의 결정은:
- 시나리오 변경 → 20 으로
- 코드/설계 변경 → INTEGRATION → 21 재검증

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
```

- [ ] **Step 2: 00-BRIEF.md 템플릿**

```markdown
# 00 — BRIEF

**Frozen.** 작업 시작 시점에 작성하고 이후 변경하지 않는다. 변경 사항은 10-WORK-LOG 에 ADR 로 기록.

## 요구

<사용자가 무엇을 원하는가? 1-2 단락>

## 성공 기준

- [ ] <검증 가능한 항목 1>
- [ ] <검증 가능한 항목 2>

## 컨텍스트 (선택)

<배경, 관련 시스템, 제약 조건>

## 외부 참조 (선택, /ce-import 시 채워짐)

<Jira ticket description, Confluence page 내용 등>
```

- [ ] **Step 3: 01-RESEARCH.md 템플릿**

```markdown
# 01 — RESEARCH

**Frozen.** 사전 조사. 필요할 때만 생성.

## 조사 목적

<왜 조사가 필요한가>

## 발견

<관련 시스템 상태, 코드 패턴, 자료, 측정값 등>

## 결론

<조사로 도출된 사전 결론. 02-DESIGN 작성의 입력>
```

- [ ] **Step 4: 02-DESIGN.md 템플릿**

```markdown
# 02 — DESIGN

**Frozen.** 초기 설계. 이후 변경은 10-WORK-LOG ADR 로만.

## 접근

<핵심 아이디어 1-2 단락>

## 구성 요소

| 컴포넌트 | 역할 | 의존성 |
|---------|------|--------|

## 데이터 흐름 / 인터페이스

<필요 시>

## 영향 범위

<수정·신규 파일 목록 미리보기>

## 위험·트레이드오프

<알려진 위험과 채택 이유>
```

- [ ] **Step 5: 10-WORK-LOG.md 템플릿**

```markdown
# 10 — WORK-LOG

**Living, append-only.** 의미 있는 결정·시행착오·교훈만 ADR 양식으로 추가.

## ADR-N: <한 줄 제목>

**발견 경위:** 어떤 검증·단계에서 발견되었나 (시행착오 흡수)
**결정:** 무엇을 어떻게 하기로
**근거:** 왜
**INTEGRATION:** 관련 commit / 빌드 / 배포 흔적 (예: `abc123`, "재배포 후 검증")

---
```

- [ ] **Step 6: 11-HISTORY.md 템플릿**

```markdown
# 11 — HISTORY

**Living, auto-append.** 의미 있는 모든 액션을 시간순 한 줄씩 기록. 세션 만료 후 복구용.

## YYYY-MM-DD

- HH:MM | <action> | <reference>

<양식 예>
- 14:30 | edit `path/to/file.md` | 섹션 추가
- 14:35 | commit `abc123` | feat: ...
- 14:40 | external command | scp X→server
- 15:00 | ADR-1 추가 | 10-WORK-LOG
```

- [ ] **Step 7: 20-TEST_PLAN.md 템플릿**

```markdown
# 20 — TEST_PLAN

**Living, append-mostly.** 검증 시나리오 누적. 새 결함·요구 발견 시 시나리오 추가.

## S-N: <시나리오 제목>

**목적:** <무엇을 검증>
**사전 조건:** <환경·데이터>
**단계:** <1, 2, 3 ...>
**합격 기준:** <어떻게 되면 통과>

---
```

- [ ] **Step 8: 21-TEST_RESULTS.md 템플릿**

```markdown
# 21 — TEST_RESULTS

**Living, 라운드 누적.** 각 round 별로 어떤 시나리오 통과/실패했는지 기록.

## Round N — YYYY-MM-DD

| 시나리오 | 결과 | 비고 |
|---------|------|------|
| S-1 | ✅ | |
| S-2 | ❌ | <원인 한 줄, 상세는 10-WORK-LOG 의 ADR-X 참조> |

---
```

- [ ] **Step 9: 30-QA_HANDOFF.md 템플릿**

```markdown
# 30 — QA_HANDOFF (dev → QA 인수)

**일회성.** 변경/기대결과/제약만. 테스트 시나리오·명령어·힌트는 포함 X (QA 영역).

## 변경 요약

<1 단락>

## 기대 결과

### 성공 시
- <검증 가능 outcome 1>
- <2>

### 실패 시 (있다면)
- <rollback / 알림 기대>

## 운영자 사용 가이드

| 작업 | 방법 |

## 알려진 제약

- <비-목표·미지원 케이스>

## 검증 환경 요건

| 항목 | 요구 |

## 문의

- 구현 상세: `90-SUMMARY.md`
- 시행착오·결정 근거: `10-WORK-LOG.md`
```

- [ ] **Step 10: 90-SUMMARY.md 템플릿**

```markdown
# 90 — SUMMARY

**Living, milestone 단위.** 현 시스템의 진실 + 종합 보고 통합. close 시 최종화.

**Linked external docs** (LINKS.md 참조):
<export 이후 자동 채워짐>

---

## 한 줄 요약

<feature 의 핵심>

## 산출물

| 위치 | 파일 | 역할 |

## 운영 가이드

| 작업 | 방법 |

## 검증 상태

| 항목 | 상태 | 비고 |

## 알려진 한계

- <비-목표>

## 후속 권고

- <next step>

## 변경 흔적 (commit 표)

| Commit | 내용 |
```

- [ ] **Step 11: 검증**

```bash
ls ~/.claude/skills/context-engineering/feature/
ls ~/.claude/skills/context-engineering/feature/templates/ | wc -l
```
Expected: `SKILL.md`, `templates/` directory. templates 9개 파일.

- [ ] **Step 12: commit**

```bash
cd ~/.claude && git add skills/context-engineering/feature/ && \
  git commit -m "feat(skill): feature SKILL.md + 9 templates"
```

---

## Task 3: Project skill + 4 templates

**Files:**
- Create: `~/.claude/skills/context-engineering/project/SKILL.md`
- Create: `project/templates/{00-OVERVIEW,10-WORK-LOG,11-HISTORY,90-SUMMARY}.md`

**Steps:**

- [ ] **Step 1: project/SKILL.md**

```markdown
---
name: context-engineering:project
description: Project 작업 폴더 (projects/YYMMDD-<name>/) 생성. 여러 Feature 를 묶는 단위. "프로젝트/여러 기능/통합 개발" 키워드나 /ce-project 슬래시로 호출.
---

# Context Engineering — Project

Project = 여러 Feature 의 묶음. 자체 cycle 없음. 하위 Feature 가 자기 cycle 돌고, Project 는 그 결과를 1X/9X 에 누적.

## 동작 흐름

### 1. 호출 시 (M1)

AskUserQuestion 으로 확인:
- type: Project
- 폴더: projects/YYMMDD-<name>/
- 첫 작업: 00-OVERVIEW 작성

### 2. 폴더 존재 검사

기존 폴더 있으면 이어서/새로/취소 분기 (Feature 와 동일 패턴).

### 3. 초기 파일 생성 (핵심 4)

- 00-OVERVIEW.md
- 10-WORK-LOG.md
- 11-HISTORY.md
- 90-SUMMARY.md

`features/` 디렉토리는 첫 sub-feature 생성 시 자동 생성.

### 4. 다음 단계

00-OVERVIEW 작성 후 첫 sub-feature 시작 권장:
- "/ce-feature <name>" 호출 (Project 안 features/@<name>/ 로 자동 라우팅)

## Sub-feature 라우팅

Project 안에서 `/ce-feature <name>` 호출 시:
- 폴더: projects/<project>/features/@<name>/
- Feature 의 모든 동작 적용

## 11-HISTORY 양식 (Project 레벨)

```
## YYYY-MM-DD
- HH:MM | start feature @auth-rotation | 시작
- HH:MM | complete feature @data-retention | close ✅
- HH:MM | cross-feature ADR-1 추가 | 10-WORK-LOG
```

## Auto vs Ask

Feature 와 동일 (전체 설계 §5.4 참조).

## 참조

- 전체 설계 §2.4: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- Templates: `./templates/`
```

- [ ] **Step 2: 00-OVERVIEW.md**

```markdown
# 00 — OVERVIEW

**Frozen.**

## 목표

<프로젝트의 큰 그림 — 무엇을·왜>

## 스코프

### 포함
- <Feature 1>
- <Feature 2>

### 제외 (Non-goal)
- <명시적 제외 항목>

## 마일스톤 (초기 안)

| 마일스톤 | 시점 | 포함 Feature |

## 외부 참조 (선택)

<관련 PRD, 미팅 노트 링크 등>
```

- [ ] **Step 3: 10-WORK-LOG.md** (Feature 와 동일 양식, ADR 외에 cross-feature 결정 포함)

Feature 의 10-WORK-LOG.md 와 동일 내용.

- [ ] **Step 4: 11-HISTORY.md** (양식 예에 "start/complete feature" 포함)

```markdown
# 11 — HISTORY

**Living, auto-append.** Project 레벨 모든 액션 + sub-feature 시작/완료/주요 사건 기록.

## YYYY-MM-DD

- HH:MM | <action> | <reference>

양식 예:
- 09:00 | start feature `@auth-rotation` | features/@auth-rotation/
- 14:30 | complete feature `@data-retention` | 90-SUMMARY closed
- 15:00 | milestone-1 hit | 02-MILESTONES.md (legacy, 통합됨)
- 15:30 | cross-feature ADR-1 | 10-WORK-LOG
```

- [ ] **Step 5: 90-SUMMARY.md** (Project 버전)

```markdown
# 90 — SUMMARY

**Living, milestone 단위.**

**Linked external docs:** (LINKS.md 참조)

---

## 한 줄 요약

<프로젝트 전체>

## 진행률

| Feature | 상태 | 비고 |
| @data-retention | ✅ closed | |
| @auth-rotation | 🟡 in progress | |
| @logging-rework | ⏸ 대기 | |

## 마일스톤 달성

| 마일스톤 | 일정 | 결과 |

## 알려진 한계 / 후속

- <다음 단계 권고>

## 변경 흔적

| Commit/PR | 내용 |
```

- [ ] **Step 6: 검증 + commit**

```bash
ls ~/.claude/skills/context-engineering/project/templates/ | wc -l
# Expected: 4
cd ~/.claude && git add skills/context-engineering/project/ && \
  git commit -m "feat(skill): project SKILL.md + 4 templates"
```

---

## Task 4: Research skill + 7 templates

**Files:**
- Create: `research/SKILL.md`
- Create: `research/templates/{00-BRIEF, 02-DESIGN, 10-WORK-LOG, 11-HISTORY, 20-METHOD, 21-FINDINGS, 90-SUMMARY}.md`

**Steps:**

- [ ] **Step 1: research/SKILL.md**

```markdown
---
name: context-engineering:research
description: Research 작업 폴더 (research/YYMMDD-<topic>/) 생성. 조사·비교·검토 (코드 변경 없음). "조사/비교/검토" 키워드나 /ce-research 슬래시로 호출.
---

# Context Engineering — Research

Research = 질문 → 조사 → 결론. 코드 변경 거의 없음. QA 인수 없음.

## 동작 흐름

### 1. 호출 시 (M1)

AskUserQuestion 으로 확정:
- type: Research
- 폴더: research/YYMMDD-<topic>/
- 첫 작업: 00-BRIEF (조사 주제·질문) 작성

### 2. 폴더 존재 검사

이어서/새로/취소 분기.

### 3. 초기 파일 생성 (핵심 4)

- 00-BRIEF.md
- 10-WORK-LOG.md
- 11-HISTORY.md
- 90-SUMMARY.md

선택 (lazy):
- 02-DESIGN.md (가설 분석 복잡 시)
- 20-METHOD.md (조사 방법론 형식적 시)
- 21-FINDINGS.md (발견이 누적되면 즉시 생성)

## Cycle (단순)

```
00 → (loop: 21 ↔ 10) → 90
```

발견 누적 → 전환점·결정 → 새 방향 조사 → ... → 충분 → SUMMARY.

## 21-FINDINGS 양식

```markdown
## F-N: <한 줄 발견 제목>

**관찰:** 무엇을 봤나
**근거:** 어디서 (파일, 명령, 인용)
**해석:** 무엇을 의미
```

90-SUMMARY 에서 `F-3 참조` 같이 인용.

## Auto vs Ask

Feature 와 동일 (§5.4).

## 참조

- 전체 설계 §2.5, §4: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- Templates: `./templates/`
```

- [ ] **Step 2: 00-BRIEF.md** (Research 버전)

```markdown
# 00 — BRIEF

**Frozen.**

## 조사 주제

<무엇을 알아내려고 하는가>

## 핵심 질문

1. <Q1>
2. <Q2>

## 가설 (선택)

<현재 가지고 있는 가설·예측>

## 결정 영향

<조사 결과가 어떤 결정에 쓰이는가>

## 외부 참조 (선택)

<선행 자료, 관련 ticket·페이지>
```

- [ ] **Step 3: 02-DESIGN.md** (Research 의 가설/방향성)

```markdown
# 02 — DESIGN

**Frozen, 선택.** 사전 가설 분석이 큰 경우만 작성.

## 가설

<주요 가설>

## 검증 가능성

<어떤 데이터·실험이 가설을 지지/반박할 수 있나>

## 대안 가설

<배제하지 말아야 할 다른 설명>
```

- [ ] **Step 4: 10-WORK-LOG.md** (동일 양식)

Feature 의 10-WORK-LOG 와 동일.

- [ ] **Step 5: 11-HISTORY.md** (동일 양식, 양식 예에 "조사 명령" 포함)

```markdown
# 11 — HISTORY

**Living, auto-append.**

## YYYY-MM-DD

양식 예:
- 10:00 | grep `pattern` in repo | 발견 N건
- 10:30 | benchmark X vs Y | results.csv
- 11:00 | F-1 발견 | 21-FINDINGS
- 11:15 | ADR-1 (방향 전환) | 10-WORK-LOG
```

- [ ] **Step 6: 20-METHOD.md** (Research 의 조사 방법론, 선택)

```markdown
# 20 — METHOD

**Frozen, 선택.** 조사 방법론이 형식적일 때 작성.

## 방법

<인터뷰, 코드 분석, 벤치마크, 문헌 조사 등>

## 데이터 수집

<출처, 도구, 절차>

## 분석 기준

<어떤 기준으로 발견을 평가할 것인가>
```

- [ ] **Step 7: 21-FINDINGS.md**

```markdown
# 21 — FINDINGS

**Living, append-only.** 각 발견에 F-N ID 부여. 90-SUMMARY 에서 인용.

## F-N: <한 줄 발견 제목>

**관찰:** 무엇을 봤나
**근거:** 어디서 (파일, 명령, 인용 — 재현 가능하게)
**해석:** 무엇을 의미하나

---
```

- [ ] **Step 8: 90-SUMMARY.md** (Research 버전)

```markdown
# 90 — SUMMARY

**Living.** 결론·권고 중심.

**Linked external docs:** (LINKS.md)

---

## 한 줄 요약

## 답변 (00-BRIEF 의 질문에 대해)

| 질문 | 답 | 근거 |
| Q1 | <한 줄> | F-1, F-3 |

## 권고

1. <권고 1>
   - 근거: F-N
   - 위험·트레이드오프:
2. <권고 2>

## 한계

- <조사가 닿지 못한 영역>
- <후속 조사 필요 항목>
```

- [ ] **Step 9: 검증 + commit**

```bash
ls ~/.claude/skills/context-engineering/research/templates/ | wc -l
# Expected: 7
cd ~/.claude && git add skills/context-engineering/research/ && \
  git commit -m "feat(skill): research SKILL.md + 7 templates"
```

---

## Task 5: QnA skill + 1 template

**Files:**
- Create: `qna/SKILL.md`, `qna/templates/QnA.md`

**Steps:**

- [ ] **Step 1: qna/SKILL.md**

```markdown
---
name: context-engineering:qna
description: QnA 단일 파일 (qna/YYMMDD-<topic>.md) 생성. 단순 질문에 답하고 종료. "물어볼 게/궁금한데/단순 질문" 키워드나 /ce-qna 슬래시로 호출.
---

# Context Engineering — QnA

QnA = 단순 질문 → 답변 → 종료. cycle, HISTORY, WORK-LOG 없음.

## 동작 흐름

### 1. 호출 시 (M1)

AskUserQuestion 으로 확정:
- type: QnA
- 파일: qna/YYMMDD-<topic>.md
- 첫 작업: 질문 + 답변 작성

### 2. 파일 생성

단일 파일. 양식은 template/QnA.md 참조.

### 3. 답변 작성 후 종료

답변 미흡하면 후속 액션으로 격상:
- Feature 로: `/ce-feature <name>`
- Research 로: `/ce-research <topic>`

## 양식

template/QnA.md 참조.

## 참조

- 전체 설계 §2.6: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
```

- [ ] **Step 2: templates/QnA.md**

```markdown
# QnA — <topic>

**Date:** YYYY-MM-DD

## 질문

<원 질문>

## 컨텍스트 (선택)

<배경, 관련 코드 위치 등>

## 답변

<답변 본문>

## 후속 액션 (있다면)

- [ ] Feature 로 격상: features/YYMMDD-<name>
- [ ] Research 로 격상: research/YYMMDD-<topic>
- [ ] 종료 — 더 필요 없음
```

- [ ] **Step 3: 검증 + commit**

```bash
ls ~/.claude/skills/context-engineering/qna/
cd ~/.claude && git add skills/context-engineering/qna/ && \
  git commit -m "feat(skill): qna SKILL.md + template"
```

---

## Task 6: Modify (meta) skill

**Files:**
- Create: `modify/SKILL.md`

**Steps:**

- [ ] **Step 1: modify/SKILL.md 작성**

```markdown
---
name: context-engineering:modify
description: Context Engineering skill 자체의 템플릿·워크플로우·이름·트리거 키워드 등을 수정. "/ce-modify" 슬래시나 "ce-feature 수정/skill 의 X 바꿔" 자연어로 호출.
---

# Context Engineering — Modify (메타 스킬)

Skill 자체 변경 안전하게 처리. 사용 중 발견한 불편 → skill 정의 반영.

## 변경 카테고리·위험도

| 카테고리 | 위험도 | 처리 |
|---------|--------|------|
| 1. 템플릿 내용 추가/수정 | 낮음 | 바로 적용 |
| 2. 워크플로우 단계 추가 | 중간 | 영향 안내 |
| 3. 이름 변경 (파일·skill) | 중간 | 마이그레이션 옵션 제공 |
| 4. 키워드 추가 | 낮음 | 바로 적용 |
| 5. 자동-묻기 정책 | 낮음 | 바로 적용 |
| 6. 워크플로우 단계 제거 | 높음 | 강경 경고 |
| 7. 번호 체계 변경 | 매우 높음 | 마이그레이션 스크립트 필수 |

## 동작 흐름

1. 진입 (/ce-modify 또는 자연어)
2. AskUserQuestion: 어느 type (feature/project/research/qna/qna/메타) + 어떤 카테고리 (1~7)
3. 변경 요청 받기 (자연어)
4. **영향 분석:**
   - 낮음 위험: 변경 사항 미리보기만
   - 중간: 영향받는 기존 폴더 목록 표시 ("진행 중인 features/A, features/B 에 영향")
   - 높음/매우 높음: 강경 경고 + 마이그레이션 계획 명시
5. AskUserQuestion: 적용 방식
   - A) 적용 + 기존 데이터 마이그레이션
   - B) 적용만 (기존 데이터 그대로)
   - C) 드라이런 (시뮬레이션만)
   - D) 취소
6. **git pre-modify commit** (옵션 A): 현재 상태를 git 에 commit 해서 revert 가능하게 함
7. 변경 적용 (skill 파일 수정 + 필요 시 마이그레이션 script 실행)
8. CHANGELOG.md 에 entry 추가
9. **git post-modify commit:** "skill-modify: <카테고리> <간단 설명>"

## 안전 장치

- **git 기반 백업** (`.backup/<date>/` 대신 git pre-modify commit 사용. ~/.claude/ 가 git repo 라 자동 처리.)
- **드라이런** — 실제 변경 안 하고 영향만 출력
- **마이그레이션 가이드** — 이름 변경 시 기존 폴더 rename 옵션 제공

## 마이그레이션 예 (이름 변경)

`10-WORK-LOG` → `10-NOTES` 변경 시:

```
AskUserQuestion:
"기존 폴더의 10-WORK-LOG.md 도 rename?"
- A) 전부 rename (진행 중·close 모두)
- B) 진행 중 (close 안 됨) 만 rename
- C) 그대로 두기 (이름 혼재 허용)
```

선택에 따라 `find` + `mv` 스크립트 실행.

## CHANGELOG.md 양식

```markdown
## YYYY-MM-DD — <카테고리> 변경: <요약>
- 변경: <Before → After>
- 영향: <기존 데이터·진행 작업>
- 마이그레이션: <수행 / 보류 / N/A>
- git: <commit hash>
- 이유: <왜>
```

## 참조

- 전체 설계 §6: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- CHANGELOG: `../CHANGELOG.md`
```

- [ ] **Step 2: 검증 + commit**

```bash
ls ~/.claude/skills/context-engineering/modify/
cd ~/.claude && git add skills/context-engineering/modify/ && \
  git commit -m "feat(skill): modify (meta) SKILL.md"
```

---

## Task 7: Export/Import (meta) skill

**Files:**
- Create: `export/SKILL.md`

**Steps:**

- [ ] **Step 1: export/SKILL.md 작성**

```markdown
---
name: context-engineering:export
description: 작업 폴더의 산출물을 Jira/Confluence 로 업로드(/ce-export) 또는 외부 내용을 작업 폴더로 import(/ce-import). 양방향 동기화 + LINKS.md 메타데이터 관리.
---

# Context Engineering — Export / Import (메타 스킬)

작업 폴더 ↔ Jira/Confluence 간 동기화. 양방향.

## 진입

### Export
```
슬래시:   /ce-export <PROJ-123 또는 Confluence URL>
자연어:   "이거 PROJ-123 에 올려", "Confluence 페이지 https://... 에 업데이트"
```

### Import — 3 Flow

**Flow A — Feature 시작 직후 import** (BRIEF 초안 → 외부 내용으로 덮어쓰기 안전)
```
[1] /ce-feature data-retention 으로 폴더·4 파일 생성
[2] /ce-import PROJ-123 to features/260512-data-retention
[3] 동작:
    - Jira description → 00-BRIEF 본문 직접 흡수
    - LINKS.md 생성
    - 폴더명 rename → [JIRA] 마커 추가
    - 11-HISTORY append
```

**Flow B — Import 부터 시작** (자동 Feature 생성)
```
[1] /ce-import PROJ-123        (target 미지정)
[2] AskUserQuestion:
    "PROJ-123 기반 새 Feature 시작?
     - 폴더: features/YYMMDD-[JIRA]-<auto-name>
     - type: Feature
     - 이름: <Jira 제목에서 추출>
     A) 진행 / B) 변경 / C) 취소"
[3] 확정 후:
    - 폴더 + 4 파일 (생성 시점부터 [JIRA] 마커)
    - 00-BRIEF = Jira description (그대로)
    - LINKS.md 생성
    - 11-HISTORY 첫 entry
```

**Flow C — 진행 중 Feature 에 사후 import** (BRIEF 완성, 덮어쓰기 위험)
```
[1] (이미 진행 중) features/YYMMDD-<name>/
[2] /ce-import PROJ-123 to features/YYMMDD-<name>
[3] AskUserQuestion: "외부 내용 어떻게 통합?
    A) 00-BRIEF 끝에 '외부 참조' 섹션 append (추천)
    B) 별도 파일 (00-BRIEF-EXTERNAL.md) 로 분리
    C) BRIEF 전체 대체 (기존 백업)
    D) LINKS.md 만 추가 (본문 변경 X)"
[4] 동작 + 11-HISTORY append
```

## Export 동작 흐름

```
[1] 컨텍스트 폴더 식별
[2] AskUserQuestion: 업로드 내용?
    - 90-SUMMARY (기본)
    - 30-QA_HANDOFF (QA ticket 용)
    - 둘 다
    - 다른 파일 직접
[3] AskUserQuestion: 업로드 방식?
    - 추가 (append)
    - 덮어쓰기 (overwrite)
    (코멘트 모드 없음)
[4] 포맷 변환 + 미리보기 (앞 30줄)
[5] AskUserQuestion: 최종 확인
[6] mcp__atlassian__* 호출
[7] LINKS.md 생성·갱신
[8] 폴더명 마커 추가 (rename, 알파벳 순)
[9] 11-HISTORY 한 줄 append
```

## 인자 패턴 인식

| 패턴 | 처리 |
|------|------|
| `PROJ-123` | Jira issue (key) |
| `https://*.atlassian.net/browse/PROJ-123` | Jira issue (URL) |
| `https://*.atlassian.net/wiki/spaces/X/pages/12345` | Confluence 기존 페이지 |
| (인자 없음 + 사용자가 새 페이지 의도) | Confluence 새 페이지 생성 |

## LINKS.md 양식

```markdown
# Linked External Documents

## Jira
- key: PROJ-123
- url: https://yourcompany.atlassian.net/browse/PROJ-123
- last_sync: YYYY-MM-DD HH:MM
- mode: append   (또는 overwrite)

## Confluence
- page_id: 12345
- url: https://yourcompany.atlassian.net/wiki/spaces/DOCS/pages/12345/Title
- title: <페이지 제목>
- last_sync: YYYY-MM-DD HH:MM
- mode: overwrite
```

## 폴더명 마커 규칙

- 알파벳 순: `[CONF]` < `[JIRA]`
- 첫 export 시점에 자동 rename
- 두 번째 시스템 추가되면 마커 추가 + 알파벳 순 재정렬
- LINKS.md 와 마커 일관성 유지

## 인증

MCP (`mcp__atlassian__*`) 가 처리. 미인증 시 `mcp__atlassian__authenticate` 호출 안내.

## 참조

- 전체 설계 §7: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
```

- [ ] **Step 2: 검증 + commit**

```bash
ls ~/.claude/skills/context-engineering/export/
cd ~/.claude && git add skills/context-engineering/export/ && \
  git commit -m "feat(skill): export/import (meta) SKILL.md"
```

---

## Task 8: Slash commands

**Files:**
- Create: `~/.claude/commands/{ce-feature,ce-project,ce-research,ce-qna,ce-modify,ce-export,ce-import}.md`

**Steps:**

- [ ] **Step 1: ce-feature.md**

```markdown
---
description: Context Engineering — Feature 작업 시작 (features/YYMMDD-<name>/)
---

context-engineering:feature skill 을 호출. 인자 $ARGUMENTS 가 있으면 이름 후보로 사용.
```

- [ ] **Step 2: ce-project.md, ce-research.md, ce-qna.md** (동일 패턴)

각 파일은 위와 같은 양식, type 만 바꿈.

ce-project.md:
```markdown
---
description: Context Engineering — Project 작업 시작 (projects/YYMMDD-<name>/)
---

context-engineering:project skill 을 호출. 인자 $ARGUMENTS 가 있으면 이름 후보로 사용.
```

ce-research.md:
```markdown
---
description: Context Engineering — Research 작업 시작 (research/YYMMDD-<topic>/)
---

context-engineering:research skill 을 호출. 인자 $ARGUMENTS 가 있으면 주제 후보로 사용.
```

ce-qna.md:
```markdown
---
description: Context Engineering — QnA 단일 파일 작성 (qna/YYMMDD-<topic>.md)
---

context-engineering:qna skill 을 호출. 인자 $ARGUMENTS 가 있으면 주제로 사용.
```

- [ ] **Step 3: ce-modify.md**

```markdown
---
description: Context Engineering skill 자체 수정 (템플릿·워크플로우·이름·키워드·정책)
---

context-engineering:modify skill 을 호출. 인자 $ARGUMENTS 는 수정 대상·요청을 자유 형식으로 받음.
```

- [ ] **Step 4: ce-export.md**

```markdown
---
description: 작업 폴더 산출물을 Jira/Confluence 로 업로드
---

context-engineering:export skill 을 호출. 인자 $ARGUMENTS:
- Jira issue key (PROJ-123) 또는 URL
- Confluence 페이지 URL
- 인자 없으면 Confluence 새 페이지 생성 의도로 해석
```

- [ ] **Step 5: ce-import.md**

```markdown
---
description: Jira/Confluence 내용을 작업 폴더로 import (3 flow 자동 분기)
---

context-engineering:export skill 을 import 모드로 호출. 인자 $ARGUMENTS:
- "<source-ref>" — Jira key 또는 Confluence URL. target 미지정이면 Flow B (자동 Feature 생성)
- "<source-ref> to <folder>" — target 지정. Flow A 또는 C 결정 (BRIEF 상태 검사)
```

- [ ] **Step 6: 검증 + commit**

```bash
ls ~/.claude/commands/ | grep "^ce-"
# Expected: 7개 (ce-feature, ce-project, ce-research, ce-qna, ce-modify, ce-export, ce-import)
cd ~/.claude && git add commands/ce-*.md && \
  git commit -m "feat(commands): Context Engineering 슬래시 명령 7개"
```

---

## Task 9: CHANGELOG.md init

**Files:**
- Create: `~/.claude/skills/context-engineering/CHANGELOG.md`

**Steps:**

- [ ] **Step 1: CHANGELOG.md 작성**

```markdown
# Context Engineering Skill — 변경 이력

## 2026-05-12 — 초기 빌드

- Context Engineering 시스템 첫 빌드
- 4 type skill (feature/project/research/qna)
- 2 meta skill (modify/export)
- 7 슬래시 명령
- CLAUDE.md 통합
- ~/.claude/ git 관리 baseline 구축
- 설계 근거: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- 빌드 plan: `~/.claude/CONTEXT_ENGINEERING_PLAN.md`
- git: <초기 commit hash>
- 이유: 매 작업마다 산출물 형식·순서·저장 위치가 ad-hoc 했음. 일관된 컨테이너로 추적·재현·재개 용이하게.
```

- [ ] **Step 2: commit**

```bash
cd ~/.claude && git add skills/context-engineering/CHANGELOG.md && \
  git commit -m "docs(skill): Context Engineering CHANGELOG init"
```

---

## Task 10: CLAUDE.md 업데이트

**Files:**
- Modify: `~/.claude/CLAUDE.md`

**Steps:**

- [ ] **Step 1: 현재 CLAUDE.md 확인**

```bash
cat ~/.claude/CLAUDE.md
```

기존 내용 확인 (한국어 응답 설정 등).

- [ ] **Step 2: Context Engineering 섹션 추가 (파일 끝에 append)**

```markdown

## Context Engineering

새 작업 시작 시 4 type 중 하나로 분류:

| Type | 트리거 키워드 예 | 슬래시 |
|------|----------------|--------|
| Feature | 구현 / 만들어 / 추가 / 기능 | `/ce-feature <name>` |
| Project | 프로젝트 / 여러 기능 / 통합 개발 | `/ce-project <name>` |
| Research | 조사 / 비교 / 검토 (코드 변경 X) | `/ce-research <topic>` |
| QnA | 단순 질문 / 물어볼 게 | `/ce-qna <topic>` |

**호출 정책:**
- Claude 가 type + 이름 후보 도출 후 **무조건 AskUserQuestion 으로 확정 받기**
- 확정 전 어떤 파일·폴더도 만들지 말 것
- 진행 중 phase 전환·분류 모호 케이스도 무조건 확인
- 11-HISTORY / 21-TEST_RESULTS / 명백한 10-WORK-LOG ADR 은 자동 (사용자에게 묻지 않음)

**산출물 위치:** 작업 폴더는 다음 중 사용자 컨벤션에 맞게:
- 프로젝트 코드와 같이: `<project>/.claude-contexts/<type>/<폴더>/`
- 또는 사용자 중앙 집중: `~/docs/<type>/<폴더>/`

**관련 메타 skill:**
- `/ce-modify` — Skill 자체 템플릿·구조 수정
- `/ce-export <ref>` — Jira/Confluence 로 업로드
- `/ce-import <ref> [to <folder>]` — Jira/Confluence 에서 가져오기 (3 flow 자동 분기)

전체 설계: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
```

- [ ] **Step 3: 검증**

```bash
grep -A 3 "Context Engineering" ~/.claude/CLAUDE.md | head -10
```
Expected: 위 섹션 나타남.

- [ ] **Step 4: commit**

```bash
cd ~/.claude && git add CLAUDE.md && \
  git commit -m "feat(claude-md): Context Engineering 섹션 추가"
```

---

## Task 11: data-retention 첫 이전 (실사용 예시)

**현재 위치:** `/Users/junsuboy/docs/260508-data-retention/`

**Files (rename + 내용 갱신):**
- `00-DISCUSSION.md` → `00-BRIEF.md` (내용 흡수)
- `01-RESEARCH.md` → (그대로)
- `02-PLAN.md` → `02-DESIGN.md`
- `12-SESSION_LOG.md` → `10-WORK-LOG.md` (ADR 부분만) + `11-HISTORY.md` (timeline)
- `20-TEST_PLAN.md` → (그대로)
- `21-TEST_RESULTS.md` → (그대로)
- `30-QA_HANDOFF.md` → (그대로)
- `90-REPORT.md` → `90-SUMMARY.md`

**Steps:**

- [ ] **Step 1: AskUserQuestion — 폴더 위치**

```
"data-retention 작업 폴더를 어디에 둘까요?
A) 그대로: /Users/junsuboy/docs/260508-data-retention/
B) 프로젝트 안: ~/Dev/firescout-3.0-utils/.claude-contexts/features/260508-data-retention/
C) 다른 위치 (수동 지정)"
```

선택 결과를 `<DEST>` 라 칭함.

- [ ] **Step 2: 백업 (이전 전)**

```bash
SRC=/Users/junsuboy/docs/260508-data-retention
cp -r "$SRC" "$SRC.backup-$(date +%Y%m%d-%H%M)"
```

- [ ] **Step 3: 폴더 이동 (B 선택 시)**

B 라면:
```bash
mkdir -p ~/Dev/firescout-3.0-utils/.claude-contexts/features/
mv "$SRC" ~/Dev/firescout-3.0-utils/.claude-contexts/features/260508-data-retention
DEST=~/Dev/firescout-3.0-utils/.claude-contexts/features/260508-data-retention
```

A 라면 `DEST=$SRC`.

- [ ] **Step 4: rename — 02-PLAN → 02-DESIGN**

```bash
mv "$DEST/02-PLAN.md" "$DEST/02-DESIGN.md"
```

`02-DESIGN.md` 의 헤더를 `# 02 — DESIGN` 으로 갱신:
- 기존 `# 데이터 보관 정책 구현 계획` 같은 첫 줄을 `# 02 — DESIGN (구 02-PLAN)` 으로

- [ ] **Step 5: 00-DISCUSSION → 00-BRIEF (내용 흡수)**

기존 `00-DISCUSSION.md` 내용을 새 `00-BRIEF.md` 로:
```bash
mv "$DEST/00-DISCUSSION.md" "$DEST/00-BRIEF.md"
```

헤더 갱신: `# 00 — BRIEF` 로. 본문 변경 X (정책 내용 그대로).

- [ ] **Step 6: 12-SESSION_LOG 분리**

`12-SESSION_LOG.md` 안에는 ADR-1, ADR-2, ADR-3 + 세션 재개 결과 1~5 가 섞여 있음.

**6-1.** `10-WORK-LOG.md` 신규 작성: ADR-1~3 만 옮기기

ADR-1 (MinIO alias), ADR-2 (rollback image), ADR-3 (gpg TTY) 의 3개 entry. 본 plan 의 Step 8 양식대로:

```markdown
# 10 — WORK-LOG

## ADR-1: MinIO local alias 보장

**발견 경위:** 192.168.2.188 1차 install 시도에서 cleanup-install.sh 의 mc ilm rule list 가 Access Denied 로 실패. 원인은 onpremise-minio 컨테이너 내부 mc 에 `local` alias 가 자동 설정 안 됨 (docker-compose.yml 의 minio-init 는 별도 컨테이너에 `minio` 라는 다른 이름으로만 등록).
**결정:** cleanup-install.sh 와 cleanup.sh 에서 mc 명령 호출 전 `mc alias set local http://localhost:9000 ...` 으로 idempotent 보장. 자격증명은 cleanup.conf 에 외부화.
**근거:** docker-compose.yml 수정은 영향 범위 큼. 클라이언트 스크립트 한 줄 추가가 최소 변경.
**INTEGRATION:** commit `45b870a fix(onpremise): MinIO local alias 보장`. 패키지 재빌드 후 192.168.2.188 재배포 (Task 7 통과).

## ADR-2: rollback-install.sh 이미지 정리 리스트 보완

**발견 경위:** 192.168.2.188 1차 시도 후 정리 중 rollback-install.sh 가 `gcr.io/cadvisor/cadvisor`, `prom/node-exporter`, `onpremise-postgres`, `firescout-3.0-{backend,frontend,backoffice}:latest`, `smoke-fire-inference-api` 이미지를 정리 못 함.
**결정:** 설치 흐름에서 생성되는 모든 이미지 태그를 명시적으로 리스트에 추가.
**근거:** label 기반 추적 방식은 영향 범위 큼. 명시적 리스트가 단순.
**INTEGRATION:** commit `6b457da fix(onpremise): rollback-install.sh 이미지 정리 리스트 보완`.

## ADR-3: docker-install.sh GPG TTY 의존 제거

**발견 경위:** 192.168.2.188 2차 install 시도에서 `gpg --dearmor` 가 기존 keyring 파일 덮어쓰기 확인을 `/dev/tty` 로 요청 → TTY 없는 환경에서 실패.
**결정:** `sudo gpg --dearmor` → `sudo gpg --batch --yes --dearmor`. 비대화형 처리.
**근거:** 한 줄 변경. 데이터 보관 PR scope 외이지만 install 안정성에 직접 영향.
**INTEGRATION:** commit `7835825 fix(install): docker-install.sh gpg --dearmor TTY 의존 제거`. 향후 패키지부터 적용.
```

**6-2.** `11-HISTORY.md` 신규 작성: "세션 재개 결과 N" sections 을 timeline 으로 변환

```markdown
# 11 — HISTORY

## 2026-05-08

- 13:00 | Test 1 (PostgreSQL 파티셔닝) 시작 | 192.168.1.225
- 13:30 | Test 1 ✅ 완료 (39,991행 분배)
- 14:00 | Test 2 (event DELETE) ✅
- 14:15 | Test 3 (MinIO ILM) ✅
- 14:30 | Test 4 (cleanup.sh) — 무한루프 발견
- 15:00 | 와일드카드 미해석 진단 + mc find --exec 로 수정
- 15:15 | 복제본 검증 — 5033 객체 1회 iteration 삭제
- 15:30 | Test 5 (cron 발화) 검증
- 16:00 | 동적 마이그레이션 (Scenario 1, 2, 3) 검증
- 16:30 | 통합 작업: source-files-onpremise/ 6 파일 추가 + package-onpremise.sh 수정
- 17:00 | bash syntax 검증, 시나리오 1·2·3 재검증

## 2026-05-11

- 15:00 | utils repo commit `872dc91` (초기 통합) + push
- 15:30 | 192.168.2.188 install 타겟 깨끗 정리
- 15:58 | scp 패키지 → 192.168.2.188
- 16:14 | install.sh 1차 시도 (PID 1020128)
- 16:18 | install.sh 실패 — ILM Access Denied → rollback 자동 실행 ✅
- 16:20 | ADR-1 추가 (10-WORK-LOG)
- 16:25 | commit `45b870a` fix(onpremise): MinIO local alias 보장 + push
- 16:25 | ADR-2 추가
- 16:25 | commit `6b457da` fix(onpremise): rollback 이미지 리스트 + push
- 16:30 | 192.168.2.188 잔여 이미지 정리

## 2026-05-12

- 16:00 | scp 새 패키지 → 192.168.2.188
- 16:10 | install.sh 2차 시도
- 16:11 | gpg TTY 에러 — ADR-3 추가 + 즉시 fix
- 16:15 | commit `7835825` fix(install): gpg TTY 제거 + push
- 16:20 | install.sh 3차 시도 (PID 1020713)
- 16:32 | install.sh 완전 성공 ✅
- 16:35 | sanity check: 컨테이너 8개, 파티션 4개, ILM 365일×2, cleanup 시스템
- 16:40 | feature close (90-SUMMARY 검증 표 갱신)
```

**6-3.** `12-SESSION_LOG.md` 삭제:

```bash
rm "$DEST/12-SESSION_LOG.md"
```

- [ ] **Step 7: 90-REPORT → 90-SUMMARY**

```bash
mv "$DEST/90-REPORT.md" "$DEST/90-SUMMARY.md"
```

본문 헤더를 `# 90 — SUMMARY (구 90-REPORT)` 로 갱신.

상단 "Linked external docs" 섹션 빈 채로 추가 (export 안 했으니).

- [ ] **Step 8: 폴더 구조 검증**

```bash
ls "$DEST"
```

Expected (정확한 새 이름 목록):
```
00-BRIEF.md
01-RESEARCH.md
02-DESIGN.md
10-WORK-LOG.md
11-HISTORY.md
20-TEST_PLAN.md
21-TEST_RESULTS.md
30-QA_HANDOFF.md
90-SUMMARY.md
```

(추가 백업 폴더는 다음 step 에서 정리)

- [ ] **Step 9: 백업 정리 (이전 검증 후)**

```bash
ls "$SRC.backup-"*
# 확인 후 OK 면:
rm -rf "$SRC.backup-"*
```

- [ ] **Step 10: 상호 참조 갱신 (다른 파일에서 옛 이름 참조 있다면)**

```bash
grep -rn "00-DISCUSSION\|02-PLAN\|12-SESSION_LOG\|90-REPORT" "$DEST" 2>/dev/null
```

만약 결과 있으면 새 이름으로 sed 치환:
```bash
cd "$DEST" && \
  for f in *.md; do
    sed -i.bak \
      -e 's/00-DISCUSSION/00-BRIEF/g' \
      -e 's/02-PLAN/02-DESIGN/g' \
      -e 's/12-SESSION_LOG/10-WORK-LOG/g' \
      -e 's/90-REPORT/90-SUMMARY/g' \
      "$f"
    rm "$f.bak"
  done
```

- [ ] **Step 11: 옵션 B 선택했을 경우 — project repo 안에 git add**

```bash
cd ~/Dev/firescout-3.0-utils && \
  git add .claude-contexts/features/260508-data-retention/ && \
  git status
```

사용자가 push 여부 결정.

옵션 A (~/docs) 선택 경우 — 사용자 영역이라 별도 git 처리 X.

- [ ] **Step 12: ~/.claude commit (data-retention 이전 흔적)**

```bash
cd ~/.claude && git add CONTEXT_ENGINEERING_PLAN.md && \
  git commit -m "chore: data-retention 첫 이전 완료 (Task 11)"
```

(이 plan 자체가 갱신됐으면 같이 commit)

---

## 빌드 완료 검증

모든 task 완료 후:

```bash
echo "=== Skill 구조 ==="
find ~/.claude/skills/context-engineering -name "*.md" | sort

echo "=== Slash commands ==="
ls ~/.claude/commands/ce-*.md

echo "=== Git log ==="
cd ~/.claude && git log --oneline | head -20

echo "=== CLAUDE.md ==="
grep "Context Engineering" ~/.claude/CLAUDE.md
```

Expected:
- skill 디렉토리에 SKILL.md + templates 합쳐 25개+ markdown
- 7개 슬래시 명령
- ~12개 commit
- CLAUDE.md 에 Context Engineering 섹션 존재

---

## 후속 (이 plan 외)

- 첫 실사용 시 발견되는 불편 → `/ce-modify` 로 반영
- plugin 으로 분리 검토 (현재는 ~/.claude/ 내부, 향후 외부 repo 로 전환 가능)
- 다른 외부 시스템 (Notion, Linear, Slack) 통합 → LINKS.md 양식 확장
