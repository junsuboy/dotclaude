# Plan — <project-name>

> 모든 Feature SPEC 확정 후 작성. 실행 순서·마일스톤·리스크.
> `architect` agent 가 02-SPEC/* 와 01-DECOMPOSITION 을 읽고 작성.

**Status:** draft | confirmed
**Generated:** YYYY-MM-DD

## 1. Execution order

01-DECOMPOSITION 의 batch 를 시간 축에 매핑:

| Batch | Features | 시작 조건 | 예상 기간 |
|-------|----------|-----------|----------|
| 1 | `@user-profile`, `@auth-google` | 즉시 | <기간> |
| 2 | `@profile-ui` | Batch 1 완료 | <기간> |

## 2. Milestones

| M | 정의 | Features | 종료 조건 |
|---|------|----------|----------|
| M1 | <이름> | `@a`, `@b` | 모두 90-SUMMARY closed |
| M2 | <이름> | `@c` | ... |

## 3. Risks

| # | 리스크 | 영향 | 완화 |
|---|--------|------|------|
| R1 | 외부 API 변경 가능성 | `@auth-google` 중단 | stub 우선 구현, 통합 후순위 |

## 4. Out-of-band work

이 Project 가 의존하지만 별도 trace 가 필요한 것:

- 인프라 셋업
- 외부 팀 작업
- 결정 대기 (legal/design review)

## 5. Done criteria

Project 전체 close 조건:

- [ ] 모든 Feature `90-SUMMARY` close
- [ ] `91-HANDOFF.md` 작성 (QA 인수)
- [ ] VISION §3 success metric 측정 가능 상태

---

**다음 단계:** Batch 1 시작 → 각 feature 마다 `/ce-feature @<name>` 호출.
