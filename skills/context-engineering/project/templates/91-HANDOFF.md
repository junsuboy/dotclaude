# Handoff — <project-name>

> dev → QA 인수 문서. 50~80줄. 변경/기대결과/제약만. 테스트 시나리오·명령어는 QA 영역.
> 모든 Feature close 후 작성.

**Status:** ready-for-qa | in-qa | accepted
**Handoff date:** YYYY-MM-DD
**Source:** `90-SUMMARY.md` + `02-SPEC/*/SPEC.md`

## 1. 변경 요약

이 Project 가 시스템에 가한 변경 (사용자 관점, 3~5 bullet):

- ...

## 2. 기대 동작

각 Feature 별 핵심 acceptance (전체 AC 아님, 대표 1~2개):

| Feature | 기대 동작 |
|---------|----------|
| `@auth-google` | Google 계정으로 로그인 가능, 세션 24h 유지 |
| `@profile-ui` | 프로필 조회·수정 가능, 변경 즉시 반영 |

## 3. 제약·전제

QA 가 알아야 할 환경/데이터/권한 조건:

- 전제: <예: 신규 가입 직후 사용자 한정>
- 제약: <예: Safari 14 이하 미지원>
- 데이터: <예: staging DB 의 test user pool 사용>

## 4. 영향 범위

기존 기능 중 회귀 가능성 있는 영역:

- ...

## 5. Known issues / 미해결

- ...

## 6. 참조

- VISION: `00-VISION.md`
- 전체 SPEC: `02-SPEC/`
- 변경 이력: `11-HISTORY.md`
- 코드 변경: <PR/commit 링크>

---

**QA 진행 후:** 결과는 QA 측 trace 에. 이 Project 폴더에는 inbound 만 기록 (이슈 발견 시 11-HISTORY 에 append).
