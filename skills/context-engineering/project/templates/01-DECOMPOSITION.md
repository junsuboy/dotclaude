# Decomposition — <project-name>

> `00-VISION.md` 를 입력으로 `architect` agent 가 도출한 Feature 후보 + DAG.
> 사용자 확정 후 `02-SPEC/@<name>/SPEC.md` 작성 단계로 진입.

**Source vision:** `00-VISION.md` (last reviewed: YYYY-MM-DD)
**Status:** draft | confirmed
**Architect run:** YYYY-MM-DD HH:MM

## 1. Feature candidates

| # | Feature | Type | Scope (1줄) | Why |
|---|---------|------|-------------|-----|
| 1 | `@auth-google` | `integration` | Google OAuth 로그인 | Vision §X — SSO 요구 |
| 2 | `@user-profile` | `data` | 사용자 프로필 스키마 + 마이그레이션 | profile 화면의 의존 |
| 3 | `@profile-ui` | `ui` | 프로필 조회/수정 화면 | Vision §Y |

**Type 정의:**
- `ui` — 화면·컴포넌트·UX flow
- `api` — 서버 엔드포인트·비즈니스 로직
- `data` — 스키마·마이그레이션·쿼리·인덱스
- `infra` — 배포·CI/CD·환경·시크릿
- `integration` — 외부 시스템·3rd-party API
- `tooling` — 내부 CLI·스크립트·dev ergonomics

## 2. Dependency DAG

```
@user-profile ──▶ @profile-ui
@auth-google (independent)
```

또는:

- `@profile-ui` blocked by `@user-profile`
- `@auth-google` — no blockers

## 3. Execution batches

병렬 가능 그룹:

- **Batch 1:** `@user-profile`, `@auth-google`
  - 이유: 서로 독립, 데이터/연동 layer
- **Batch 2:** `@profile-ui`
  - 이유: Batch 1 완료 후 UI

## 4. Open questions

architect 가 vision 만으로 답할 수 없던 것:

- Q1: ...

## 5. Out of scope

VISION non-goals + architect 판단으로 제외:

- ...

---

**다음 단계:**
1. 사용자 확정 → status: `confirmed`
2. 각 Feature 별로 `02-SPEC/@<name>/SPEC.md` 작성 (`spec-writer` agent 호출)
3. SPEC 모두 완료 후 `03-PLAN.md` 작성
