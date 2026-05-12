# SPEC — <feature-name>

> Feature 별 명세서. `02-SPEC/@<feature-name>/SPEC.md` 위치.
> `spec-writer` agent 가 작성. 구현자는 이 문서만 보고 만들 수 있어야 함.

**Type:** <ui | api | data | infra | integration | tooling>
**Status:** draft | confirmed | implementing | done
**Blocked by:** <feature names, or "none">
**Blocks:** <feature names, or "none">
**Owner:** <implementer / TBD>

## 1. Goal

한 단락. 이 feature 가 무엇을 deliver 하는가, 누가 쓰는가, 왜.

## 2. Scope

### In
- 포함되는 것 (bullet)

### Out
- 제외되는 것, 특히 인접 feature 와의 경계

## 3. Acceptance criteria

관찰 가능·테스트 가능한 형태로:

1. Given <state>, when <action>, then <result>.
2. ...

## 4. Contract

(feature type 에 맞는 섹션만 채우기)

### UI
- 화면/컴포넌트
- 사용자 flow (state + transition)
- 입력/출력/에러 상태
- 참조 디자인·기존 컴포넌트

### API
- Endpoint: METHOD /path
- Request schema
- Response schema (성공·에러 코드)
- AuthN/AuthZ
- Idempotency / rate limit

### Data
- 영향받는 테이블 (DDL diff)
- 인덱스
- 마이그레이션 전략 (backfill·online·lock 영향)
- 보존·개인정보 분류

### Infra
- 생성/변경 리소스
- Config·secret
- 롤백 경로

### Integration
- 외부 시스템·버전
- 인증 방식
- 실패 모드 (timeout·partial·quota)
- 로컬 stub·sandbox

### Tooling
- 명령 surface
- 입출력
- 실행 위치 (CI·로컬·둘 다)

## 5. Edge cases

빈 입력, max 입력, 동시 호출, 네트워크 실패, partial state, 권한 거부. 해당 없으면 사유 명시.

## 6. Dependencies

- 영향 코드 경로 (grep 기반)
- 외부 라이브러리
- 다른 feature 의존성

## 7. Open questions

- Q1: ...

## 8. Non-goals

리뷰어가 포함됐다고 오해할 만한 것 명시 제외.

---

**구현 단계:** 이 SPEC 확정 후 `/ce-feature @<name>` 호출 → `features/@<name>/` 폴더 자동 생성 + 이 SPEC 자동 주입.
