---
name: context-engineering:export
description: 작업 폴더의 산출물을 Jira/Confluence 로 업로드(/ce-export) 또는 외부 내용을 작업 폴더로 import(/ce-import). JIRA 가 단일 anchor — 폴더 마커·LINKS.md·sync 모두 JIRA 키 중심. Confluence 는 선택적이며 반드시 JIRA 티켓 안에 링크가 자동 삽입됨.
---

# Context Engineering — Export / Import (메타 스킬)

작업 폴더 ↔ Jira/Confluence 간 동기화. 양방향. **JIRA 중심.**

## 핵심 정책 (2026-05-12 개정)

1. **JIRA 는 모든 외부 동기화의 anchor.** 어떤 export/import 도 JIRA 키 없이는 진행하지 않는다.
2. **Confluence 는 선택적.** 사용해도 되지만 반드시 JIRA 티켓과 함께. Confluence 페이지 생성/업데이트 시 그 링크가 JIRA description 끝부분(또는 코멘트)에 자동 삽입된다.
3. **폴더 마커 = JIRA 키 자체.** `[PROJ-1183]` 처럼 티켓 번호를 그대로 마커로 사용. 더 이상 `[CONF]`, `[JIRA]` 같은 시스템명 마커를 쓰지 않는다.
4. **LINKS.md 는 유지.** 양식만 JIRA 우선·Confluence 종속으로 재구성.
5. **외부 문서는 자립적(self-contained)이어야 한다.** 읽는 사람이 CE workflow 를 몰라도 이해되게 쓴다. 내부 넘버링(`00-`, `91-` …)·파일명·워크플로우 용어(type, phase, agent 이름)를 그대로 노출하지 않고 일반적인 글로 풀어쓴다. → 아래 [독자 친화 변환] 참조.

## 진입

### Export
```
슬래시:   /ce-export <PROJ-123 또는 Confluence URL>
자연어:   "이거 PROJ-123 에 올려", "Confluence 페이지 https://... 에 업데이트"
```

JIRA 키 없이 Confluence URL 만 들어오면:
- LINKS.md 에 JIRA 키가 이미 있으면 그걸 사용
- 없으면 `AskUserQuestion` 으로 JIRA 키를 받는다. 사용자가 거부하면 export 중단.

### Import — 3 Flow

**Flow A — Feature 시작 직후 import** (BRIEF 초안 → 외부 내용으로 덮어쓰기 안전)
```
[1] /ce-feature data-retention 으로 폴더·4 파일 생성
[2] /ce-import PROJ-123 to features/260512-data-retention
[3] 동작:
    - Jira description → 00-BRIEF 본문 직접 흡수
    - LINKS.md 생성 (Jira 섹션 채움)
    - 폴더명 rename → features/260512-[PROJ-123]-data-retention
    - 11-HISTORY append
```

**Flow B — Import 부터 시작** (자동 Feature 생성)
```
[1] /ce-import PROJ-123        (target 미지정)
[2] AskUserQuestion:
    "PROJ-123 기반 새 Feature 시작?
     - 폴더: features/YYMMDD-[PROJ-123]-<auto-name>
     - type: Feature
     - 이름: <Jira 제목에서 추출>
     A) 진행 / B) 변경 / C) 취소"
[3] 확정 후:
    - 폴더 + 4 파일 (생성 시점부터 [PROJ-123] 마커)
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
[4] 동작 + 폴더명 rename + 11-HISTORY append
```

**Confluence import.** 동일 3 flow 적용. 단, target 폴더 LINKS.md 에 JIRA 키가 없으면 먼저 JIRA 키를 묻는다.

## Export 동작 흐름

```
[1] 컨텍스트 폴더 식별 (또는 명시 지정)
[2] 인자에서 대상 시스템 추론:
    - 인자가 JIRA 키 / JIRA URL → JIRA export
    - 인자가 Confluence URL → Confluence export (JIRA 키 확보 후)
    - 인자 없음 → AskUserQuestion: 어디로? (JIRA / Confluence)
[3] AskUserQuestion: 업로드 내용?
    - 90-SUMMARY (기본)
    - 30-QA_HANDOFF (QA ticket 용)
    - 둘 다
    - 다른 파일 직접
[4] AskUserQuestion: 업로드 방식?
    - 추가 (append)
    - 덮어쓰기 (overwrite)
    (코멘트 모드 없음)
[5] 독자 친화 변환(de-jargon) + 포맷 변환 + 미리보기 (앞 30줄)
    - 내부 넘버링·파일명·CE 용어를 일반 표현으로 치환 (아래 [독자 친화 변환] 표)
    - 미리보기 시점에 변환 결과를 보여줘 사용자가 검수 가능하게 함
[6] AskUserQuestion: 최종 확인
[7] mcp__atlassian__* 호출
    - JIRA 모드: editJiraIssue 또는 addCommentToJiraIssue
    - Confluence 모드:
        a. createConfluencePage 또는 updateConfluencePage
        b. JIRA 티켓 description 끝에 'Related Confluence: <url>' append
           (이미 같은 URL 있으면 skip, 양식: `## Linked Confluence\n- <title>: <url>`)
[8] LINKS.md 생성·갱신
[9] 폴더명 마커 추가/갱신 (rename) — JIRA 키만 사용
[10] 11-HISTORY 한 줄 append
```

## 인자 패턴 인식

| 패턴 | 처리 |
|------|------|
| `PROJ-123` | Jira issue (key) |
| `https://*.atlassian.net/browse/PROJ-123` | Jira issue (URL) |
| `https://*.atlassian.net/wiki/spaces/X/pages/12345` | Confluence 기존 페이지 (+ JIRA 키 확보 필요) |
| (인자 없음 + 사용자가 Confluence 새 페이지 의도) | Confluence 새 페이지 생성 (+ JIRA 키 확보 필요) |

## LINKS.md 양식 (개정 — JIRA 필수, Confluence 종속)

```markdown
# Linked External Documents

## Jira  (필수 — anchor)

- key: PROJ-123
- url: https://yourcompany.atlassian.net/browse/PROJ-123
- project: PROJ
- title: <Jira 제목>
- assignee: <이름>
- status_at_last_sync: <상태>

### Sync history

- YYYY-MM-DD HH:MM — direction: import|export, mode: <설명>, source: <파일|n/a>

## Confluence  (선택 — 0..N개)

- page_id: 12345
- url: https://yourcompany.atlassian.net/wiki/spaces/DOCS/pages/12345/Title
- title: <페이지 제목>
- linked_in_jira: yes  (JIRA description/comment 에 링크 삽입됨)

### Sync history

- YYYY-MM-DD HH:MM — direction: export, mode: overwrite, source: 90-SUMMARY.md
```

## 폴더명 마커 규칙 (개정)

- 형태: `[<JIRA-KEY>]` — 예: `260508-[PROJ-1183]-data-retention`
- 첫 JIRA export/import 시점에 자동 rename
- Confluence 단독 연결로는 마커 추가 X (JIRA 가 anchor 이므로)
- 동일 작업에 JIRA 키가 여러 개 묶이는 케이스는 비표준. 발생 시 첫 키만 폴더 마커로 쓰고 나머지는 LINKS.md 본문에 기록.
- 기존 `[JIRA]` / `[CONF]` 시스템명 마커는 deprecated — `ce-export` 호출 시 LINKS.md 의 key 를 읽어 자동 마이그레이션.

## 독자 친화 변환 (De-jargon) — export 필수

외부로 내보내는 문서는 **CE workflow 를 모르는 사람도 단독으로 이해**할 수 있어야 한다.
업로드 직전 [5] 단계에서 다음을 적용한다.

### 치환 표

| 내부 마커 / 용어 | 외부 문서 표현 |
|---|---|
| `00-VISION`, `00-BRIEF` | 개요·배경·목표 |
| `01-DECOMPOSITION` | 기능 분해·범위 |
| `02-SPEC`, `SPEC.md` | 상세 명세 |
| `03-PLAN` | 구현 계획 |
| `90-SUMMARY` | 작업 요약 |
| `30-QA_HANDOFF`, `91-HANDOFF` | QA 인수 / 검증 항목 |
| `10-WORK-LOG` | 작업 메모 |
| `11-HISTORY`, `21-TEST_RESULTS` | 변경 이력 / 테스트 결과 |
| feature·project·research·qna **type** | (언급 안 함 — 문서 성격을 자연어로) |
| phase P1~P6, architect·spec-writer **agent** | (언급 안 함) |

### 원칙

1. **헤딩의 `NN-` 접두 번호 제거.** `## 02-SPEC` → `## 상세 명세`.
2. **자기참조 금지.** "이 SPEC 에 따르면", "P3 계획 단계에서", "feature 폴더의…" 같은 내부 맥락 표현을 쓰지 않는다.
3. **약어·내부 코드네임 풀어쓰기.** 처음 등장 시 한 번은 풀어서 설명.
4. **외부 시스템 키는 그대로.** JIRA 키(`PROJ-123`)·Confluence 링크는 변환 대상이 아님.
5. **모호하면 변환 후 미리보기로 검수.** 사용자가 미리보기에서 잔여 전문용어를 잡을 수 있게 한다.

> 적용 범위: export 로 나가는 본문에만 적용. 작업 폴더 원본 파일(`90-SUMMARY.md` 등)은 넘버링·용어 그대로 유지한다.

## 인증

MCP (`mcp__atlassian__*`) 가 처리. 미인증 시 `mcp__atlassian__authenticate` 호출 안내.

## Confluence 링크 자동 삽입 — 세부

Confluence export 가 성공한 직후:

1. `mcp__atlassian__getJiraIssue` 로 현재 description 가져오기
2. description 에 `## Linked Confluence` 섹션이 없으면 추가, 있으면 같은 URL 인지 확인
3. 새 항목이면 `- <title>: <url>` 한 줄 append
4. `mcp__atlassian__editJiraIssue` 로 업데이트

실패 시 (권한 / 네트워크 등) export 자체는 성공으로 두되 11-HISTORY 에 `confluence-link-injection-failed` 로 기록하고 사용자에게 경고.

## 참조

- 전체 설계: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md` §7
- 변경 이력: `~/.claude/skills/context-engineering/CHANGELOG.md`
