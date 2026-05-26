---
name: context-engineering
description: 새 작업을 5 task type (Feature/Project/Research/QnA/IntegrationTest) 중 하나로 분류하고 산출물 폴더 + 핵심 파일을 생성한다. 사용자가 "구현/만들어/추가/조사/비교/프로젝트 시작/통합테스트/실기기 검증/물어볼 게" 같은 요청을 하면 호출.
---

# Context Engineering — 진입점

새 작업을 시작할 때 5 type 중 어디에 속하는지 분류 후 해당 type skill 로 dispatch.

## Type 분류 가이드

| Type | 트리거 키워드 | 슬래시 | 다음 skill |
|------|--------------|--------|-----------|
| Feature | 구현 / 만들어 / 추가 / 기능 | `/ce-feature <name>` | `context-engineering:feature` |
| Project | 프로젝트 / 여러 기능 / 통합 개발 | `/ce-project <name>` | `context-engineering:project` |
| Research | 조사 / 비교 / 검토 (코드 변경 X) | `/ce-research <topic>` | `context-engineering:research` |
| QnA | 단순 질문 / 물어볼 게 | `/ce-qna <topic>` | `context-engineering:qna` |
| IntegrationTest | 통합테스트 / 실기기 검증 / 에픽 검증 / 여러 Feature 한꺼번에 검증 | `/ce-integration-test <name>` | `context-engineering:integration-test` |

## 동작

1. 사용자 입력에서 type + 이름 후보 추출
2. **무조건 AskUserQuestion 으로 확정 받기** (확정 전 어떤 파일·폴더 생성 X)
3. 확정된 type 의 sub-skill 로 dispatch

## 모호한 경우

키워드 매칭 약하면:
```
AskUserQuestion("어떤 작업 유형인가요?",
  A) Feature — 기능 구현
  B) Project — 여러 기능 묶음 (기획→분해→실행)
  C) Research — 조사 (코드 변경 X)
  D) QnA — 단순 질문
  E) IntegrationTest — 여러 Feature 의 실기기/실환경 통합 검증)
```

> IntegrationTest 는 Feature 가 아님. **이미 구현된** N개 Feature 를 실환경에서 한 번에 검증하는 작업. 폴더 위치도 `~/docs/integration-tests/` 로 분리.

## 핵심 원칙

- 사용자 확인 무조건 받기 (호출 직전·phase 전환·큰 갈림길)
- 11-HISTORY append 는 자동 (의미 있는 모든 액션 기록)
- 10/11 분류 애매하면 묻기
- 폴더명: `YYMMDD-<name>` + JIRA 연결 시 `[<JIRA-KEY>]` 마커 추가 (예: `260508-[PROJ-1183]-data-retention`). Confluence 는 JIRA 안에 링크되므로 별도 마커 없음.

## 참조

- 전체 설계: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- 변경 이력: `~/.claude/skills/context-engineering/CHANGELOG.md`
