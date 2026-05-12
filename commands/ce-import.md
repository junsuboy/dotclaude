---
description: Jira/Confluence 내용을 작업 폴더로 import (3 flow 자동 분기)
---

context-engineering:export skill 을 import 모드로 호출. 인자 $ARGUMENTS:
- "<source-ref>" — Jira key 또는 Confluence URL. target 미지정이면 Flow B (자동 Feature 생성)
- "<source-ref> to <folder>" — target 지정. Flow A 또는 C 결정 (BRIEF 상태 검사)
