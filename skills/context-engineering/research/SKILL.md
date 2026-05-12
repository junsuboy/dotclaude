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
