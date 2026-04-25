---
name: check-freshness
description: guides/ 파일의 last-reviewed 날짜를 확인하여 오래된 파일을 감사한다
context: fork
allowed-tools:
  - Read
  - Bash
---

guides/ 디렉터리의 모든 마크다운 파일을 감사하라.

절차:
1. guides/ 디렉터리의 모든 .md 파일 목록을 가져와라
2. 각 파일의 `<!-- last-reviewed: YYYY-MM-DD -->` 태그를 읽어라
3. 오늘 날짜를 기준으로 경과 일수를 계산하라

결과를 다음 형식으로 보고하라:

## 신선도 감사 결과

| 파일 | last-reviewed | 경과일 | 상태 |
|---|---|---|---|
| [파일명] | [날짜] | [일수] | 최신/점검 필요/오래됨 |

기준:
- 90일 이내: 최신
- 90~180일: 점검 필요
- 180일 초과: 오래됨 (업데이트 권장)

오래된 파일에 대해 `/update-guides [파일명]` 실행을 제안하라.
