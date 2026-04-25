<!-- last-reviewed: 2026-04-05 -->
# Hooks (생명주기 훅)

Hooks를 사용하면 Claude Code의 특정 이벤트에 반응하여 자동으로 명령을 실행할 수 있다.

> 참고: 공식 문서 → https://code.claude.com/docs/en/hooks

---

## Hooks란?

자동으로 실행되는 커맨드라인 스크립트.

예시 활용:
- 모든 파일 수정 후 자동 포맷터 실행
- 세션 시작 시 환경 확인
- 특정 파일 수정 시 알림
- 어떤 instruction 파일이 로드됐는지 로깅

---

## settings.json에 Hooks 설정

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "hooks": {
    "SessionStart": [
      {
        "command": "echo 'Claude 세션 시작'",
        "timeout": 5000
      }
    ],
    "PostEdit": [
      {
        "command": "npm run lint --fix",
        "timeout": 30000,
        "filePattern": "**/*.ts"
      }
    ]
  }
}
```

---

## 지원 이벤트

| 이벤트 | 트리거 시점 |
|---|---|
| `SessionStart` | Claude Code 세션 시작 시 |
| `PostEdit` | Claude가 파일을 저장한 후 |
| `PostBash` | Claude가 Bash 명령을 실행한 후 |
| `InstructionsLoaded` | instruction 파일(CLAUDE.md, rules 등)이 로드될 때 |

### InstructionsLoaded 활용

path-specific rules 디버깅이나 어떤 파일이 언제 로드됐는지 추적할 때 유용하다.

```json
{
  "hooks": {
    "InstructionsLoaded": [
      {
        "command": "echo \"Loaded: $CLAUDE_INSTRUCTION_PATH\" >> /tmp/claude-instructions.log"
      }
    ]
  }
}
```

---

## Hook 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `command` | string | 실행할 쉘 명령어 |
| `timeout` | number | 밀리초 단위 타임아웃 (기본값: 10000) |
| `filePattern` | string | PostEdit 전용. 이 패턴에 맞는 파일만 트리거 |

---

## 스킬 내 훅 (Skill-level Hooks)

SKILL.md frontmatter의 `hooks` 필드로 스킬 생명주기에 훅을 설정할 수 있다.

```markdown
---
name: deploy
description: 애플리케이션 배포
hooks:
  before: "echo '배포 시작' | slack-notify"
  after: "echo '배포 완료' | slack-notify"
---

배포 스크립트를 실행하라.
```

스킬 훅 vs settings.json 훅:
- 스킬 훅: 해당 스킬 실행 시에만 동작
- settings.json 훅: 모든 세션/편집에 전역 적용

---

## 실용적인 예시

### 자동 포맷터 (TypeScript/JavaScript)

```json
{
  "hooks": {
    "PostEdit": [
      {
        "command": "npx prettier --write $CLAUDE_FILE_PATH",
        "timeout": 15000,
        "filePattern": "**/*.{ts,tsx,js,jsx}"
      }
    ]
  }
}
```

### 자동 린트 (Python)

```json
{
  "hooks": {
    "PostEdit": [
      {
        "command": "ruff check --fix $CLAUDE_FILE_PATH",
        "timeout": 10000,
        "filePattern": "**/*.py"
      }
    ]
  }
}
```

### 세션 시작 시 환경 확인

```json
{
  "hooks": {
    "SessionStart": [
      {
        "command": "node --version && npm --version",
        "timeout": 5000
      }
    ]
  }
}
```

### Instruction 파일 로드 로깅

```json
{
  "hooks": {
    "InstructionsLoaded": [
      {
        "command": "echo \"$(date): $CLAUDE_INSTRUCTION_PATH\" >> ~/.claude/instruction-log.txt",
        "timeout": 1000
      }
    ]
  }
}
```

---

## 주의사항

- Hook 실패(비 0 종료 코드)는 Claude에게 경고로 표시된다.
- 너무 긴 Hook은 응답성을 저하시킨다. 타임아웃 적절히 설정.
- `$CLAUDE_FILE_PATH` 환경 변수에 수정된 파일 경로가 들어온다.
- Hook은 `.claude/settings.json` (프로젝트) 또는 `~/.claude/settings.json` (전역)에 설정 가능.
