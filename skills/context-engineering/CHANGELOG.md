# Context Engineering Skill — 변경 이력

## 2026-05-21 — 산출물 위치 ~/docs/<type>/ 중앙 집중 고정 (카테고리 5)

- 변경: 산출물 폴더 위치를 `프로젝트 코드와 같이(.claude-contexts) | 중앙 집중(~/docs)` 선택 → `~/docs/<type>/<폴더>/` 단일 고정. 위치를 더 이상 묻지 않음
- 수정 파일: `CLAUDE.md`(산출물 위치 섹션), `CONTEXT_ENGINEERING_DESIGN.md`(L542, L550)
- 영향: 기존 폴더 마이그레이션 불필요 — 모든 기존 feature 가 이미 `~/docs/features/` 에 존재. alongside-code 경로는 실사용된 적 없음
- 마이그레이션: N/A
- 이유: 사용자 실제 컨벤션이 `~/docs/<type>/` 로 정착, 매 호출 위치 질문이 불필요한 마찰

## 2026-05-20 — ce-export 독자 친화 변환(de-jargon) 제약 추가 (카테고리 2)

- 변경: export SKILL.md 에 핵심 정책 #5(외부 문서 self-contained) + "독자 친화 변환" 섹션 신설, Export 흐름 [5] 단계에 de-jargon 패스 삽입
- 내용: 내부 넘버링(`00-`, `91-` …)·파일명·CE 용어(type/phase/agent)를 외부 문서에서 일반 표현으로 치환하는 표·원칙 정의. 작업 폴더 원본은 그대로 유지(export 본문에만 적용)
- 영향: 기존 폴더·넘버링 체계 변경 없음 — 향후 export 렌더링에만 적용
- 마이그레이션: N/A
- 이유: 외부(Jira/Confluence) 독자가 CE workflow 를 몰라도 문서를 이해할 수 있게

## 2026-05-12 — 외부 skill 매트릭스 + type 별 cycle hook (카테고리 1)

Project skill 재설계 직후 보강. 새 agent 추가 없이 기존 자산 재활용 + feature cycle 경량 boost.

- **Project SKILL** 에 "phase × 외부 skill" 호출 매트릭스 추가:
  - P1 Vision 비어있을 때 → `superpowers:brainstorming` 제안
  - P5 feature close 직전 → `superpowers:requesting-code-review` 자동 트리거
  - P5 batch 병렬 → `superpowers:dispatching-parallel-agents` 제안
  - P6 HANDOFF 직전 → `superpowers:verification-before-completion` enforce
- **Feature SKILL** 에 type 별 cycle hook 추가 (경량 — 새 phase 없음, 기존 phase 의 섹션·산출물 boost):
  - `data` → `21-MIGRATION.md` (DDL diff·dry-run·rollback) 자동 제안
  - `ui` → `frontend-design` skill 호출 제안, 디자인 ADR
  - `api` → schema·contract test enforce
  - `infra` → `21-ROLLBACK.md` 자동 제안
  - `integration` → stub/real 분리 enforce
  - `tooling` → self-use 섹션 자동 제안
- **새 agent 추가 X** — 중복·드리프트 회피
- 수정 파일:
  - `skills/context-engineering/project/SKILL.md`
  - `skills/context-engineering/feature/SKILL.md`
- 이유: Project 가 단순 컨테이너 아니라는 점은 P1~P6 으로 정리됐지만, 실 구현 시 type 별 차이 (예: data 마이그레이션, ui 디자인) 가 cycle 에 반영 안 되어 있었음. 경량 hook 으로 보강.

## 2026-05-12 — Project skill 오케스트레이터 재설계 (카테고리 1·2)

- **변경**: Project skill 을 단순 컨테이너 → 기획→분해→스펙→플랜→실행→인수 (P1~P6) 오케스트레이터로 개편
- **추가 산출물 (project/templates/)**:
  - `00-VISION.md` — 한 장 기획서 (입력)
  - `01-DECOMPOSITION.md` — `architect` agent 의 Feature 분해 + DAG
  - `02-SPEC.md` — `spec-writer` 가 feature 별로 채우는 템플릿 (`02-SPEC/@<name>/SPEC.md`)
  - `03-PLAN.md` — 실행 순서·마일스톤·리스크
  - `91-HANDOFF.md` — QA 인수 (50~80줄, dev→QA scope 정책 반영)
- **추가 agents (~/.claude/agents/)**:
  - `architect.md` — 기획 분해·DAG·실행 batch (Plan/general 류, code 안 씀)
  - `spec-writer.md` — Feature 별 AC·계약·엣지케이스 명세 (구현 안 함)
- **Feature type 도입**: `ui/api/data/infra/integration/tooling` — `architect` 가 분류, SPEC frontmatter 에 기록, feature 호출 시 BRIEF 에 주입
- **feature/SKILL.md 수정**:
  - Project 컨텍스트 감지 (CWD 상위에 00-VISION + 01-DECOMPOSITION 존재 시)
  - sibling 02-SPEC/@<name>/SPEC.md 자동 참조 + AC 복사
  - type 태그 주입, Project 의 11-HISTORY 에 feature start/complete 한 줄 동기화
- **호환성**: 기존 `00-OVERVIEW.md` 베이스 Project 폴더는 그대로 인정. 신규부터 새 모델 적용. 강제 마이그레이션 X.
- **수정 파일**:
  - `skills/context-engineering/project/SKILL.md` (전면 개편)
  - `skills/context-engineering/feature/SKILL.md` (Project 인식 + SPEC 주입 + type 태그)
  - `skills/context-engineering/project/templates/` 5개 신규
  - `agents/architect.md`, `agents/spec-writer.md` 신규
- **이유**: 기존 Project 가 "여러 Feature 의 디렉토리 묶음" 수준이라 기획·분해·의존성 관리가 ad-hoc. 한 장 기획에서 출발해 Feature 단위 구현까지 일관된 phase 모델로 추적 가능하게.

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
