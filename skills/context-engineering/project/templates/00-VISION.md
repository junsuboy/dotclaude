# Vision — <project-name>

> 한 장 기획서. 모든 후속 단계 (분해·명세·계획·구현) 의 입력.
> 분량 가이드: 1~2 페이지. 길어지면 잘못된 단계.

## 1. Problem

해결하려는 문제. 현상 + 영향. (3~5문장)

## 2. Target user

누가 쓰는가. 페르소나·역할·맥락. 너무 많으면 우선순위.

## 3. Success metric

성공/실패를 판정할 **관찰 가능한 지표**. 정성적 ("UX 개선") 금지.

- Primary: <수치 지표>
- Secondary: <보조 지표>

## 4. Scope

### In
- 포함되는 큰 범주 (Feature 가 아님 — 기능 후보는 01-DECOMPOSITION 에서 도출)
- 예: "로그인", "프로필 조회", "결제"

### Out (non-goals)
- 명시적으로 제외하는 것. 분해 단계에서 누락 의심받을 만한 것들 미리 적기.

## 5. Constraints

기술·일정·법규·인력 제약. 분해 단계에서 의존성/우선순위에 영향.

- 기술: <예: 기존 stack 유지>
- 일정: <예: M1 release 까지>
- 법규/보안: <예: PII 저장 금지>
- 외부 의존: <예: 3rd-party API X 필요>

## 6. Assumptions

가정. 틀리면 전체 재설계 필요. 분해 시 architect 가 검증.

- A1: ...
- A2: ...

## 7. Open questions

확정 안 된 것. 분해/명세 진행하면서 채워짐.

- Q1: ...

---

**다음 단계:** `01-DECOMPOSITION.md` 생성 — `architect` agent 가 이 문서를 읽고 Feature 후보 + DAG 제안.
