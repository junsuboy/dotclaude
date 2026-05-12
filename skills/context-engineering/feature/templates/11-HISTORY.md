# 11 — HISTORY

**Living, auto-append.** 의미 있는 모든 액션을 시간순 한 줄씩 기록. 세션 만료 후 복구용.

## YYYY-MM-DD

- HH:MM | <action> | <reference>

<양식 예>
- 14:30 | edit `path/to/file.md` | 섹션 추가
- 14:35 | commit `abc123` | feat: ...
- 14:40 | external command | scp X→server
- 15:00 | ADR-1 추가 | 10-WORK-LOG
