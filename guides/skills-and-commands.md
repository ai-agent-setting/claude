<!-- last-reviewed: 2026-04-05 -->
# Skills 및 명령어 시스템

Claude Code에서 반복적인 워크플로를 `/슬래시` 명령으로 재사용할 수 있다.

> 참고: 공식 문서 → https://code.claude.com/docs/en/skills

---

## Skills vs Commands

| | Commands (구형) | Skills (신형) |
|---|---|---|
| 경로 | `.claude/commands/*.md` | `.claude/skills/<name>/SKILL.md` |
| 상태 | 여전히 동작 | 권장 방식 |
| 기능 | 기본 | frontmatter, 포크, 서브에이전트 지원 |

`.claude/commands/*.md` 방식은 하위 호환이 유지되므로 마이그레이션은 선택 사항이다.

---

## SKILL.md 파일 구조

```
.claude/
└── skills/
    └── my-skill/
        ├── SKILL.md        # 진입점 (필수)
        ├── context.md      # 롤플레이 컨텍스트 등 지원 파일
        └── template.txt    # 스킬 내에서 참조하는 파일
```

### SKILL.md frontmatter 필드

```markdown
---
name: my-skill
description: 이 스킬이 하는 일 (Claude가 자동 실행 여부 판단에 사용)
context: fork           # fork = 별도 컨텍스트에서 실행 (메인 대화 오염 방지)
disable-model-invocation: false  # true = /skill-name 입력해도 자동 실행 안 됨
allowed-tools:          # 이 스킬이 사용할 수 있는 도구 목록
  - Read
  - Bash
model: claude-opus-4-6  # 특정 모델 지정 (기본값: 현재 선택 모델)
user-invocable: true    # false = 사용자가 직접 실행 불가 (다른 스킬에서만 호출 가능)
argument-hint: "[파일경로] [옵션]"  # 자동완성 시 표시될 힌트
effort: medium          # 노력 수준: low / medium / high / max
agent: general-purpose  # context: fork 시 사용할 서브에이전트 타입
hooks:                  # 스킬 생명주기 훅
  before: "echo 'starting'"
  after: "echo 'done'"
paths:                  # 특정 파일 패턴에서만 활성화 (생략 시 항상 활성)
  - "src/**/*.ts"
shell: bash             # 인라인 쉘 명령 실행 쉘: bash / powershell
---

# 스킬 지침
여기에 스킬 실행 지침을 작성한다.
$ARGUMENTS는 사용자가 /skill-name 뒤에 입력한 텍스트로 치환된다.
```

---

## 내장 스킬 (Bundled Skills)

Claude Code에 기본 포함된 스킬들:

| 스킬 | 설명 |
|---|---|
| `/batch` | 대규모 변경을 병렬로 오케스트레이션. git worktree 사용 |
| `/claude-api` | Claude API 레퍼런스 로드 (anthropic 임포트 시 자동 활성화) |
| `/debug` | 현재 오류 디버깅 워크플로 시작 |
| `/loop [interval] <prompt>` | 지정 간격으로 반복 실행 (예: `/loop 5m check deploy`) |
| `/simplify [focus]` | 병렬 3개 리뷰 에이전트 실행 후 수정 |

---

## $ARGUMENTS 사용법

`$ARGUMENTS`는 `/skill-name` 뒤에 사용자가 입력한 텍스트로 치환된다.

```
/review src/auth.ts
→ SKILL.md에서 $ARGUMENTS = "src/auth.ts"

/review src/auth.ts 보안 관점에서
→ $ARGUMENTS = "src/auth.ts 보안 관점에서"
```

### 위치 기반 인수 접근

`$ARGUMENTS[N]`으로 특정 인덱스 접근, `$N` 단축 형식도 지원된다.

```
/deploy main production
→ $ARGUMENTS[0] = "main"   ($1 단축형 동일)
→ $ARGUMENTS[1] = "production"  ($2 단축형 동일)
```

---

## 동적 컨텍스트 주입 (Shell Injection)

`` !`<command>` `` 문법으로 SKILL.md 내에서 쉘 명령 결과를 동적으로 주입할 수 있다.
Claude 실행 전 전처리 단계에서 치환된다.

```markdown
현재 브랜치: !`git branch --show-current`
최근 커밋: !`git log --oneline -5`

위 컨텍스트를 바탕으로 $ARGUMENTS를 리뷰하라.
```

---

## 내장 변수

| 변수 | 설명 |
|---|---|
| `$ARGUMENTS` | `/skill-name` 뒤 전체 텍스트 |
| `$ARGUMENTS[N]` / `$N` | N번째 인수 (0-indexed) |
| `${CLAUDE_SESSION_ID}` | 현재 세션 ID |
| `${CLAUDE_SKILL_DIR}` | 스킬 디렉터리 경로 |

---

## 스킬 우선순위 및 위치

같은 이름의 스킬이 여러 위치에 있을 경우 우선순위:

```
enterprise > personal > project
```

플러그인 스킬은 `plugin-name:skill-name` 네임스페이스로 구분된다.

### 스킬 검색 위치

1. `.claude/skills/` — 프로젝트 레벨
2. `~/.claude/skills/` — 개인 전역
3. 플러그인 스킬 — `plugin-name:skill-name` 형식

**모노레포 자동 발견**: 하위 디렉터리 파일 편집 시 해당 디렉터리의 `.claude/skills/`에서 스킬을 자동 발견한다.

---

## 스킬 예시

### 기본 리뷰 스킬

```markdown
---
name: review
description: 코드 리뷰를 수행한다
argument-hint: "[파일경로]"
---

$ARGUMENTS 파일 또는 변경사항을 리뷰하라.

다음 항목을 순서대로 검토하라:
1. 버그와 논리 오류
2. 보안 취약점
3. 성능 문제
4. 코드 스타일 및 가독성

각 항목별로 구체적인 개선 제안을 제공하라.
```

### 포크 기반 조사 스킬 (메인 컨텍스트 보호)

```markdown
---
name: investigate
description: 코드베이스를 조사하고 요약을 반환한다
context: fork
agent: Explore
allowed-tools:
  - Read
  - Bash
---

$ARGUMENTS 에 대해 조사하라.

조사 후 다음 형식으로 요약하라:
- 발견한 내용
- 관련 파일 목록
- 권장 다음 단계
```

### 자동 실행 방지 스킬 (배포 등)

```markdown
---
name: deploy
description: 애플리케이션을 프로덕션에 배포한다
disable-model-invocation: true
---

배포 전 다음을 확인하라:
1. 모든 테스트 통과
2. 변경 로그 업데이트
3. 버전 태그 추가

확인 후: npm run deploy
```

### 동적 컨텍스트 활용 스킬

```markdown
---
name: branch-review
description: 현재 브랜치의 변경사항을 리뷰한다
context: fork
---

브랜치: !`git branch --show-current`
변경 파일: !`git diff --name-only HEAD~1`

위 변경사항을 리뷰하라.
```

---

## 전역 스킬 모음

`~/.claude/skills/`에 설치된 스킬은 모든 프로젝트에서 자동으로 사용 가능하다.

| 스킬 | 설명 |
|---|---|
| `/review` | 코드 리뷰 |
| `/summarize` | 파일/변경사항 요약 |
| `/explain` | 코드 설명 |
| `/update-guides` | 공식 문서 확인 후 가이드 업데이트 제안 |
| `/check-freshness` | guides/ 파일의 최신 여부 감사 |
| `/review-claude-config` | Claude Code 설정 파일 품질 검토 |
