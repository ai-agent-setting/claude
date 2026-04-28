---
name: review-claude-config
description: 프로젝트의 Claude Code 관련 설정 파일 전체를 베스트 프랙티스 기준으로 검토하고 점수화된 보고서를 출력한다
context: fork
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
---

대상 프로젝트 경로: $ARGUMENTS (인자가 없으면 현재 디렉터리 `.` 기준)

검토 기준은 아래 파일을 먼저 읽어라:
- criteria.md (이 파일과 같은 디렉터리)

---

# 실행 절차

## Step 0: 파일 목록 수집

다음 파일/디렉터리를 탐색하라:

```bash
# CLAUDE.md 위치 탐색 (루트 + .claude/ 하위)
find . -maxdepth 2 -name "CLAUDE.md" -o -name "CLAUDE.local.md" 2>/dev/null

# .claude/ 전체 구조
find .claude -type f 2>/dev/null | sort

# .gitmodules
cat .gitmodules 2>/dev/null || echo "MISSING"
```

## Step 1: 5개 영역 검토

각 영역을 criteria.md의 기준에 따라 검토하고, 항목별 Pass/Fail을 기록하라.

### 영역 1: CLAUDE.md

찾은 모든 CLAUDE.md와 CLAUDE.local.md를 읽어라.
각 파일에 대해:
- 줄 수 측정: `wc -l` 또는 라인 카운트
- 빌드/테스트/실행 명령어 섹션 존재 여부
- 규칙이 구체·측정 가능한지 (모호한 표현 예: "좋은", "깔끔한", "깨끗한" 등)
- `@path` import 활용 여부 (200줄 초과 시)
- `.claude/rules/` 사용 여부 (규칙이 많을 때)

### 영역 2: settings.json

`.claude/settings.json`을 읽어라 (없으면 "미설정"으로 기록).
- `$schema` 필드 존재 여부
- `permissions.allow` 설정 여부
- `permissions.deny`에 `.env`, `secrets/` 등 민감 파일 포함 여부
- `hooks` 필드 구조 여부 (있다면 구조 검증)
- `autoMemoryEnabled` 설정 여부

### 영역 3: Skills (.claude/skills/)

`.claude/skills/**/SKILL.md` 파일 전체를 읽어라 (없으면 "미설정").
각 SKILL.md에 대해:
- frontmatter에 `name`, `description` 필드 존재 여부
- 줄 수 500줄 이내 여부
- 부작용(배포, 커밋, 외부 알림)이 있는 스킬에 `disable-model-invocation: true` 여부
- `$ARGUMENTS` 활용 여부
- `context: fork` 사용이 적절한 스킬에 설정 여부

### 영역 4: Rules (.claude/rules/)

`.claude/rules/*.md` 파일들을 읽어라 (없으면 "미설정").
각 파일에 대해:
- path-specific 규칙에 YAML frontmatter `paths:` 필드 존재 여부
- 규칙 파일 간 충돌 가능성 여부
- `paths:` 없는 파일은 "전역 규칙"으로 분류 (과도하게 많으면 경고)

### 영역 5: Agents (.claude/agents/)

`.claude/agents/**/SKILL.md` 파일들을 읽어라 (없으면 "미설정" — 감점 없음).
각 파일에 대해:
- `context: fork` 설정 여부 (에이전트는 fork 권장)
- `allowed-tools` 명시적 제한 여부 (미제한이면 경고)
- `user-invocable: false` 설정 여부 (에이전트는 직접 호출 방지 권장)

---

## Step 3: 보고서 출력

아래 형식으로 출력하라. 상태 이모지: ✅ (8점 이상) / ⚠️ (5~7점) / ❌ (4점 이하) / ➖ (미설정·해당 없음)

```
============================================================
 Claude Code 설정 검토 결과
 대상: [프로젝트 경로]  |  검토일: [오늘 날짜]
============================================================

| # | 영역              | 상태 | 점수  | 요약                              |
|---|-------------------|------|-------|-----------------------------------|
| 1 | CLAUDE.md         | [상태] | [x/10] | [한 줄 요약]                    |
| 2 | settings.json     | [상태] | [x/10] | [한 줄 요약]                    |
| 3 | Skills            | [상태] | [x/10] | [한 줄 요약]                    |
| 4 | Rules             | [상태] | [x/10] | [한 줄 요약]                    |
| 5 | Agents            | [상태] | [x/10] | [한 줄 요약]                    |
|   | **종합**          |      | [x/50] |                                   |

------------------------------------------------------------
 개선 제안 (우선순위 순)
------------------------------------------------------------

[높음]
1. ...

[중간]
2. ...

[낮음]
3. ...

------------------------------------------------------------
 다음 단계
------------------------------------------------------------
- 개선 적용 후 재검토: /review-claude-config [경로]
- CLAUDE.md 작성 가이드: ~/.claude/guides/claude-md-authoring.md
- Skills 가이드: ~/.claude/guides/skills-and-commands.md
- 안티패턴 참고: ~/.claude/guides/common-pitfalls.md
============================================================
```

### 점수 산정 방식

각 영역은 10점 만점. 검토 항목 수로 나눠 백분율 환산.

- **CLAUDE.md**: 파일 존재(2점), 200줄 이내(2점), 빌드/테스트 명령어(2점), 구체적 규칙(2점), 적절한 길이 관리(`@import`/`rules/`)(2점)
- **settings.json**: 파일 존재(2점), `$schema`(2점), deny 민감 파일(3점), permissions 설정(3점)
- **Skills**: 스킬 없으면 5점(보통). 있다면 frontmatter 완성도, 줄 수, 부작용 처리 기준으로 산정
- **Rules**: 없으면 7점(보통 — 필수 아님). 있다면 frontmatter 완성도 기준
- **Agents**: 없으면 8점(보통 — 선택 사항). 있다면 fork/allowed-tools 기준
- **Agents**: 없으면 8점(보통 — 선택 사항). 있다면 fork/allowed-tools 기준
