# Context Engineering Skill — 변경 이력

## 2026-05-12 — JIRA 중심 컨벤션 개정

- export/import 정책 변경: JIRA 가 단일 anchor, Confluence 는 선택적 (반드시 JIRA 안에 링크 자동 삽입)
- 폴더 마커 컨벤션 변경: `[CONF]` / `[JIRA]` 시스템명 마커 → `[<JIRA-KEY>]` 티켓 키 마커 (예: `[PROJ-1183]`)
- LINKS.md 양식 갱신: Jira 섹션 필수·Confluence 종속
- 기존 `[JIRA]` 마커 폴더는 자동 마이그레이션 (LINKS.md 의 key 를 읽어 rename)
- 수정 파일:
  - `skills/context-engineering/SKILL.md` (마커 규칙)
  - `skills/context-engineering/export/SKILL.md` (전반)
  - `CLAUDE.md` (Context Engineering 섹션)
  - `CONTEXT_ENGINEERING_DESIGN.md` (§7 갱신)
- 이유: Confluence/JIRA 마커가 별도로 늘면 폴더명이 비대해지고, 외부에서 첫 찾기는 보통 JIRA 티켓번호로 시작함. JIRA 키 자체를 마커로 두면 폴더-티켓 1:1 식별이 즉시 됨.
- 백업: `~/.claude/skills/context-engineering/.backup/<timestamp>/`

## 2026-05-12 — 초기 빌드

- Context Engineering 시스템 첫 빌드
- 4 type skill (feature/project/research/qna)
- 2 meta skill (modify/export)
- 7 슬래시 명령
- CLAUDE.md 통합
- ~/.claude/ git 관리 baseline 구축
- 설계 근거: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- 빌드 plan: `~/.claude/CONTEXT_ENGINEERING_PLAN.md`
- git: a3ae88d (initial baseline)
- 이유: 매 작업마다 산출물 형식·순서·저장 위치가 ad-hoc 했음. 일관된 컨테이너로 추적·재현·재개 용이하게.
