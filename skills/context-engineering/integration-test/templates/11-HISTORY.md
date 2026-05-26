# 11 — HISTORY

**Living, auto-append.** 의미 있는 모든 액션을 시간순 한 줄씩 기록. 세션 만료 후 복구용.

## YYYY-MM-DD

- HH:MM | <action> | <reference>

<양식 예>
- 10:00 | create integration-test | 폴더 생성
- 11:30 | 사전 점검 | N/N ✅
- 14:00 | INT-01 PASS | evidence/INT-01_*.log
- 15:30 | INT-12 FAIL → 후조치 #1 시작 | EVIDENCE-LOG 후조치
- 16:45 | PR #25 머지 (형상 반영) | a636963
- 17:30 | INT-12 재검증 PASS (🔁) | evidence/INT-12_retest.log
