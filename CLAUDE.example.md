# CLAUDE.md — 공개용 예시 (Public Example)

> 실제 운영 중인 `CLAUDE.md`는 회사 컨텍스트(내부 URL·프로젝트 키·리뷰어 지정 등)를 포함하므로 추적하지 않는다.
> 이 파일은 구조를 보여주기 위한 sanitized 예시다. `<...>` 와 `PROJ-*` 는 placeholder.

## 절대 규칙 (도구 호출)
- 도구 호출 블록 앞에 잉여 토큰/단어를 절대 출력하지 말 것. 토큰이 붙으면 호출이 실행되지 않고 명령어 텍스트만 보인다.
- 도구는 정상 호출 블록으로만 출력한다. 설명이 필요하면 완결된 문장 뒤에 곧바로 호출한다.

## 절대 규칙 (서브에이전트 구현)
- 코드 구현·수정을 서브에이전트에 맡길 때는 **반드시 격리 worktree** 에서 실행시킨다 (`git worktree add` 로 전용 워킹트리 + 전용 브랜치). 공유 워킹트리를 동시 점유하면 커밋이 엉뚱한 브랜치에 안착한다.

## 언어
- 한국어로 응답. 산출물(plan/design/doc)도 모두 한국어 — skill 의 영어 템플릿을 그대로 복제하지 않는다.

## 문서(Markdown) 다이어그램
- 저장되는 MD 문서 안의 다이어그램·플로우차트·구조도·시퀀스도는 ASCII 박스아트 대신 **mermaid 코드블록**으로 작성한다 (폰트·뷰어에 따라 정렬이 깨지므로).
- 채팅 답변(터미널 출력)에는 mermaid 를 쓰지 않는다 — 텍스트·표·리스트로 설명한다.

## Context Engineering

새 작업 시작 시 5 type 중 하나로 분류:

| Type | 트리거 키워드 예 | 슬래시 |
|------|----------------|--------|
| Feature | 구현 / 만들어 / 추가 / 기능 | `/ce-feature <name>` |
| Project | 프로젝트 / 기획 / 여러 기능 / 통합 개발 | `/ce-project <name>` |
| Research | 조사 / 비교 / 검토 (코드 변경 X) | `/ce-research <topic>` |
| QnA | 단순 질문 | `/ce-qna <topic>` |
| IntegrationTest | 통합테스트 / 실기기 검증 | `/ce-integration-test <name>` |

**호출 정책:**
- Claude 가 type + 이름 후보 도출 후 **무조건 AskUserQuestion 으로 확정 받기**
- 확정 전 어떤 파일·폴더도 만들지 말 것
- 11-HISTORY / 21-TEST_RESULTS / 명백한 10-WORK-LOG ADR 은 자동 (묻지 않음)

**산출물 위치 (고정):** 항상 `~/docs/<type>/YYMMDD-<name>/` 중앙 집중. 위치를 묻지 않는다.

**외부 동기화 정책:** JIRA 가 모든 외부 동기화의 단일 anchor — JIRA 키 없이는 export/import 진행하지 않는다. Confluence 는 선택적이며, 사용 시 JIRA 티켓에 링크를 자동 삽입한다. 폴더 마커 = JIRA 키 자체 (예: `260508-[PROJ-1183]-data-retention`).

전체 설계: `CONTEXT_ENGINEERING_DESIGN.md`

## Git / PR 컨벤션
- main/master 직접 push 금지 — 항상 feature branch → PR → review → merge.
- 브랜치명에 이슈 키 포함: `feature/<PROJ-KEY>-<name>` (예: `feature/PROJ-1234-login`), `bugfix/…`, `hotfix/…`.
- 이슈 연결을 branch + PR title(`[PROJ-KEY] 제목`) + PR body(클릭 가능한 이슈 링크) 모두에 넣는다.
- merge 전 review 1명 approve + required CI 통과.
