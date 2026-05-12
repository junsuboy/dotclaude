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
