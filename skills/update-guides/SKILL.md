---
name: update-guides
description: 공식 Claude Code 문서를 확인하여 가이드 업데이트를 제안한다
context: fork
allowed-tools:
  - Read
  - WebFetch
disable-model-invocation: false
---

$ARGUMENTS 관련 가이드를 업데이트하라.

인자가 없으면 전체 가이드를 점검하라.

절차:
1. guides/ 디렉터리의 관련 파일을 읽어라
2. 파일 상단의 `last-reviewed` 날짜를 확인하라
3. 해당 공식 문서를 WebFetch로 가져와라:
   - memory: https://code.claude.com/docs/en/memory
   - skills: https://code.claude.com/docs/en/skills
   - best-practices: https://code.claude.com/docs/en/best-practices
   - settings: https://code.claude.com/docs/en/settings
   - overview: https://code.claude.com/docs/en/overview
4. 현재 가이드와 공식 문서를 비교하라
5. 변경이 필요한 부분을 다음 형식으로 보고하라:

## 업데이트 보고서

### [파일명]
- **현재**: [기존 내용]
- **공식 문서**: [최신 내용]
- **권장 변경**: [구체적인 수정 방향]

주의: 이 스킬은 제안만 한다. 실제 파일 수정은 직접 하라.
