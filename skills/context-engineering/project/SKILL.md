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
