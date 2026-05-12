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
