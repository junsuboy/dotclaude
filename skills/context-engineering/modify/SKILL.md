---
name: context-engineering:modify
description: Context Engineering skill 자체의 템플릿·워크플로우·이름·트리거 키워드 등을 수정. "/ce-modify" 슬래시나 "ce-feature 수정/skill 의 X 바꿔" 자연어로 호출.
---

# Context Engineering — Modify (메타 스킬)

Skill 자체 변경 안전하게 처리. 사용 중 발견한 불편 → skill 정의 반영.

## 변경 카테고리·위험도

| 카테고리 | 위험도 | 처리 |
|---------|--------|------|
| 1. 템플릿 내용 추가/수정 | 낮음 | 바로 적용 |
| 2. 워크플로우 단계 추가 | 중간 | 영향 안내 |
| 3. 이름 변경 (파일·skill) | 중간 | 마이그레이션 옵션 제공 |
| 4. 키워드 추가 | 낮음 | 바로 적용 |
| 5. 자동-묻기 정책 | 낮음 | 바로 적용 |
| 6. 워크플로우 단계 제거 | 높음 | 강경 경고 |
| 7. 번호 체계 변경 | 매우 높음 | 마이그레이션 스크립트 필수 |

## 동작 흐름

1. 진입 (/ce-modify 또는 자연어)
2. AskUserQuestion: 어느 type (feature/project/research/qna/메타) + 어떤 카테고리 (1~7)
3. 변경 요청 받기 (자연어)
4. **영향 분석:**
   - 낮음 위험: 변경 사항 미리보기만
   - 중간: 영향받는 기존 폴더 목록 표시 ("진행 중인 features/A, features/B 에 영향")
   - 높음/매우 높음: 강경 경고 + 마이그레이션 계획 명시
5. AskUserQuestion: 적용 방식
   - A) 적용 + 기존 데이터 마이그레이션
   - B) 적용만 (기존 데이터 그대로)
   - C) 드라이런 (시뮬레이션만)
   - D) 취소
6. **git pre-modify commit** (옵션 A): 현재 상태를 git 에 commit 해서 revert 가능하게 함
7. 변경 적용 (skill 파일 수정 + 필요 시 마이그레이션 script 실행)
8. CHANGELOG.md 에 entry 추가
9. **git post-modify commit:** "skill-modify: <카테고리> <간단 설명>"

## 안전 장치

- **git 기반 백업** (`.backup/<date>/` 대신 git pre-modify commit 사용. ~/.claude/ 가 git repo 라 자동 처리.)
- **드라이런** — 실제 변경 안 하고 영향만 출력
- **마이그레이션 가이드** — 이름 변경 시 기존 폴더 rename 옵션 제공

## 마이그레이션 예 (이름 변경)

`10-WORK-LOG` → `10-NOTES` 변경 시:

```
AskUserQuestion:
"기존 폴더의 10-WORK-LOG.md 도 rename?"
- A) 전부 rename (진행 중·close 모두)
- B) 진행 중 (close 안 됨) 만 rename
- C) 그대로 두기 (이름 혼재 허용)
```

선택에 따라 `find` + `mv` 스크립트 실행.

## CHANGELOG.md 양식

```markdown
## YYYY-MM-DD — <카테고리> 변경: <요약>
- 변경: <Before → After>
- 영향: <기존 데이터·진행 작업>
- 마이그레이션: <수행 / 보류 / N/A>
- git: <commit hash>
- 이유: <왜>
```

## 참조

- 전체 설계 §6: `~/.claude/CONTEXT_ENGINEERING_DESIGN.md`
- CHANGELOG: `../CHANGELOG.md`
