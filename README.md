# dotclaude — Personal Claude Code Configuration

개인 Claude Code 설정 + **Context Engineering** skill 시스템.

매 작업마다 산출물 형식·순서·저장 위치가 달라지는 ad-hoc 문제 해결. 모든 작업을 4 type 중 하나로 분류하고, 정해진 numbering·workflow·산출물 구조로 진행한다. 산출물 폴더가 곧 작업의 컨텍스트가 되어 추적·재현·재개가 쉬워진다.

---

## 빠른 시작

새 작업 시작 시 자연어 또는 슬래시 명령으로 진입:

| Type | 트리거 예 | 슬래시 |
|------|----------|--------|
| **Feature** | "X 구현해줘" / "X 기능 추가" | `/ce-feature <name>` |
| **Project** | "X 프로젝트 시작" / "여러 기능 통합" | `/ce-project <name>` |
| **Research** | "X 조사해" / "비교 검토" (코드 변경 X) | `/ce-research <topic>` |
| **QnA** | "물어볼 게" / 단순 질문 | `/ce-qna <topic>` |

**호출 흐름:**

1. Claude 가 type + 이름 후보 도출
2. **무조건 AskUserQuestion 으로 확정 받기** (사용자 컨펌 전 파일·폴더 생성 X)
3. 확정 시 핵심 4 파일 생성 후 작업 진행

산출물 위치: `~/docs/<type>/YYMMDD-<name>/` 또는 `<project>/.claude-contexts/<type>/YYMMDD-<name>/`

---

## 시스템 구조

```
~/.claude/
├── CLAUDE.md                                ← 전역 지시 (한국어 + Context Engineering 트리거)
├── CONTEXT_ENGINEERING_DESIGN.md            ← 전체 설계 (조사·결정·근거)
├── CONTEXT_ENGINEERING_PLAN.md              ← 빌드 이력
├── commands/
│   ├── ce-feature.md / ce-project.md / ce-research.md / ce-qna.md
│   ├── ce-modify.md
│   ├── ce-export.md / ce-import.md
│   └── ...
└── skills/
    └── context-engineering/
        ├── SKILL.md                         ← 진입점 (type 분류)
        ├── feature/    SKILL.md + 9 templates
        ├── project/    SKILL.md + 4 templates
        ├── research/   SKILL.md + 7 templates
        ├── qna/        SKILL.md + template
        ├── modify/     SKILL.md             ← 메타: skill 자체 수정
        ├── export/     SKILL.md             ← 메타: Jira/Confluence 동기화
        └── CHANGELOG.md                     ← skill 변경 이력
```

---

## 동작 방식

### Numbering 규칙

| 영역 | 역할 | 변경성 |
|------|------|--------|
| 0X | 초기 입력·계획 (BRIEF, RESEARCH, DESIGN) | frozen (한 번 작성) |
| 10 | curated WORK-LOG (ADR + 교훈) | living, append-only |
| 11 | raw HISTORY (시간순 모든 액션) | living, auto-append |
| 2X | 검증 (TEST_PLAN/RESULTS) 또는 발견 (FINDINGS) | living |
| 30 | QA 인수 (Feature 전용) | 일회성 |
| 90 | 종합 SUMMARY | living, milestone 단위 |

핵심 4 파일 (`00`, `10`, `11`, `90`) 은 즉시 생성, 나머지는 phase 진입 시 lazy 생성.

### Feature cycle (핵심)

```
0X frozen → 20 → 21 → 10 ↔ INTEGRATION ↔ 21 → ... → 모두 통과 → 30 → 90
```

10 의 결정은 두 분기:
- **시나리오 변경** → 20 으로 (검증 범위 추가)
- **코드/설계 변경** → INTEGRATION (적용·빌드·배포) → 21 재검증

### 묻기 vs 자동

| 시점 | 묻기 |
|------|------|
| Skill 호출 직전 (type / 이름 / 폴더 생성) | ✅ 무조건 |
| Phase 전환 | ✅ |
| 10/11 분류 애매 | ✅ |
| 10-WORK-LOG ADR (명백) | ❌ 자동 |
| 11-HISTORY append | ❌ 자동 |
| 21-TEST_RESULTS append | ❌ 자동 |
| Skill close | ✅ |
| 외부 export 실행 | ✅ |

큰 갈림길과 분류 모호 케이스만 확인, mechanical 한 기록은 자동.

---

## 4 Type 비교

| Type | doc 수 | Cycle | 폴더 |
|------|--------|-------|------|
| Feature | 9 | 있음 (INTEGRATION 포함) | `features/YYMMDD-<name>/` |
| Project | 4 + features/ | 없음 (sub-feature 단위) | `projects/YYMMDD-<name>/` |
| Research | 5 (+선택 2) | 단순 (Investigate loop) | `research/YYMMDD-<topic>/` |
| QnA | 1 (단일 파일) | 없음 | `qna/YYMMDD-<topic>.md` |

상세 설계는 [`CONTEXT_ENGINEERING_DESIGN.md`](./CONTEXT_ENGINEERING_DESIGN.md) §2 참조.

---

## 메타 Skill

### `/ce-modify` — Skill 자체 수정

운영 중 발견한 불편 → skill 정의 반영. 위험도 7 카테고리 (템플릿 내용 ~ 번호 체계 변경), git pre/post-modify commit 으로 안전 백업, 드라이런·마이그레이션 옵션. CHANGELOG 자동 기록.

### `/ce-export` / `/ce-import` — Jira·Confluence 동기화

작업 폴더 ↔ 외부 시스템 양방향:
- **Export:** 90-SUMMARY 또는 30-QA_HANDOFF 를 Jira description / Confluence 페이지에 append 또는 overwrite
- **Import (3 flow):**
  - **A** — Feature 시작 직후 import (BRIEF 빈 → 외부 내용 흡수)
  - **B** — Import 부터 시작 (자동 Feature 생성, 폴더명에 `[JIRA]` 마커)
  - **C** — 진행 중 Feature 에 사후 import (BRIEF 보존, append 추천)
- 외부 링크 메타데이터는 `LINKS.md` 에 저장
- 폴더명에 `[CONF]`, `[JIRA]` 마커 (알파벳 순) 자동 추가

---

## 폴더명 컨벤션

```
YYMMDD-<name>                              기본
YYMMDD-[CONF]-<name>                       Confluence 연동
YYMMDD-[JIRA]-<name>                       Jira 연동
YYMMDD-[CONF][JIRA]-<name>                 둘 다 (알파벳 순)
```

---

## 실사용 예시 — `260508-data-retention`

FireScout 3.0 on-premise 데이터 보관 정책 구현. 첫 실사용 케이스.

위치: `~/docs/features/260508-data-retention/`

```
00-BRIEF.md          요구 + 정책 결정
01-RESEARCH.md       용량 분석
02-DESIGN.md         초기 설계
10-WORK-LOG.md       ADR-1 (MinIO alias) / ADR-2 (rollback img list) / ADR-3 (gpg TTY)
11-HISTORY.md        2026-05-08 ~ 05-12 timeline (~28 entries)
20-TEST_PLAN.md      검증 시나리오
21-TEST_RESULTS.md   라운드별 결과
30-QA_HANDOFF.md     QA 인수
90-SUMMARY.md        최종 요약 + 운영 가이드
```

작업 흐름의 결정과 시행착오가 어떻게 어디에 누적되는지 참조 가능.

---

## Git 관리

`~/.claude/` 전체가 git repo (filtered). 백업·다중 기기 동기화·롤백 가능.

**추적 대상:** CLAUDE.md, skills/, commands/, settings.json, 디자인·플랜 docs, auto-memory (projects/<project>/memory/)

**무시 대상:** cache/, sessions/, history.jsonl, todos/, shell-snapshots/, telemetry/, plugins/cache/, per-session memory 등 (상세는 `.gitignore` 참조)

modify skill 이 skill 변경 전·후 자동 commit → 항상 rollback 가능.

---

## 참조 문서

- [**CONTEXT_ENGINEERING_DESIGN.md**](./CONTEXT_ENGINEERING_DESIGN.md) — 전체 설계, 결정 근거, mermaid 다이어그램
- [**CONTEXT_ENGINEERING_PLAN.md**](./CONTEXT_ENGINEERING_PLAN.md) — 빌드 task plan (역사적 산출물)
- [**skills/context-engineering/CHANGELOG.md**](./skills/context-engineering/CHANGELOG.md) — Skill 변경 이력
- [**CLAUDE.md**](./CLAUDE.md) — 전역 지시 (Claude 가 매 세션 자동 로드)
- 각 type SKILL.md — `skills/context-engineering/{feature,project,research,qna,modify,export}/SKILL.md`
