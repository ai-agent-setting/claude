# Claude Code 설정 검토 기준

SKILL.md에서 참조하는 영역별 상세 검토 기준.

---

## 영역 1: CLAUDE.md

### 필수 항목 (없으면 감점)

| 항목 | 기준 | 근거 |
|---|---|---|
| 파일 존재 | `CLAUDE.md` 또는 `.claude/CLAUDE.md` 존재 | 없으면 Claude가 프로젝트를 이해 못 함 |
| 빌드/테스트 명령어 | 코드블록에 빌드·실행·테스트 명령어 포함 | 없으면 Claude가 수정 후 검증 불가 |
| 200줄 이내 | 파일당 200줄 이하 | 공식 권장. 초과 시 순응도 저하 |
| 구체적 규칙 | "좋은", "깔끔한", "깨끗한" 등 모호한 표현 없음 | 측정 불가 규칙은 무의미 |

### 권장 항목 (없어도 감점 최소)

| 항목 | 기준 |
|---|---|
| 프로젝트 개요 | 한 두 문장으로 목적·기술 스택 설명 |
| `@import` 활용 | 200줄 초과 시 `@path/to/file`로 분리 |
| `.claude/rules/` 사용 | 규칙이 많을 때 주제별 파일로 분리 |
| 아키텍처 설명 | 핵심 디렉터리 구조와 역할 |

### 안티패턴 (발견 시 명시)

- "YOU MUST", "IMPORTANT" 없이 중요 규칙이 묻혀있음
- `CLAUDE.local.md`가 `.gitignore`에 없음
- `/compact` 후 사라지는 일회성 지침만 있고 CLAUDE.md에 없음
- 자주 변하는 정보(버전 번호, 임시 URL) 포함

---

## 영역 2: settings.json

### 필수 항목

| 항목 | 기준 |
|---|---|
| `$schema` | `"https://json.schemastore.org/claude-code-settings.json"` |
| 민감 파일 deny | `.env`, `.env.*`, `secrets/`, API 키 파일을 `deny` 목록에 포함 |

### 권장 항목

| 항목 | 기준 |
|---|---|
| `permissions.allow` | 자주 쓰는 Bash 명령어 미리 허가 (매번 승인 팝업 방지) |
| `permissions.deny` | `curl`, `wget`, `rm -rf` 등 위험 명령어 제한 |
| `autoMemoryEnabled` | 명시적으로 설정 (기본값: true) |
| `hooks` | SessionStart/PostEdit 훅 구조 정의 (없어도 됨) |

### 검사 예시

```json
✅ Good:
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "deny": ["Read(./.env)", "Read(./secrets/**)", "Bash(rm -rf *)"]
  }
}

❌ Bad:
{} // 빈 파일 또는 $schema 없음
```

---

## 영역 3: Skills

### 필수 항목

| 항목 | 기준 |
|---|---|
| frontmatter `name` | 스킬 이름 필드 존재 |
| frontmatter `description` | Claude가 자동 실행 여부 판단에 사용. 없으면 오작동 가능 |
| 500줄 이내 | 공식 권장. 초과 시 companion file로 분리 |

### 중요 항목

| 항목 | 기준 |
|---|---|
| `disable-model-invocation: true` | 배포, 커밋, 외부 API 호출, 슬랙 메시지 등 부작용 스킬에 필수 |
| `context: fork` | 코드베이스 조사·분석 스킬에 권장 (메인 컨텍스트 보호) |
| `allowed-tools` 제한 | 불필요한 도구 접근 방지 |

### 권장 항목

| 항목 | 기준 |
|---|---|
| `$ARGUMENTS` 활용 | 동적 입력을 받는 스킬은 `$ARGUMENTS` 사용 |
| companion files | 검토 기준 등 긴 참조 내용은 별도 파일로 분리 |
| `user-invocable: false` | 다른 스킬에서만 호출되는 스킬에 설정 |

---

## 영역 4: Rules

필수 아님. 있을 때만 검증.

### 검토 항목

| 항목 | 기준 |
|---|---|
| path-specific frontmatter | 경로별 규칙 파일에 `paths:` YAML frontmatter 존재 |
| 전역 규칙 수 | `paths:` 없는 파일이 5개 초과 → 과도한 전역 규칙 경고 |
| 규칙 충돌 | 동일 내용에 대해 상충되는 규칙 존재 여부 |

### 예시

```markdown
✅ Good:
---
paths:
  - "src/api/**/*.ts"
---
# API 규칙
...

❌ Bad:
# 경로 없이 모든 파일에 적용되는 규칙들이 10개 이상의 파일로 분산됨
```

---

## 영역 5: Agents

선택 사항. 없으면 감점 없음.

### 있을 때 검토 항목

| 항목 | 기준 |
|---|---|
| `context: fork` | 에이전트는 별도 컨텍스트 실행 권장 |
| `allowed-tools` | 명시적으로 제한 (미제한 에이전트는 보안 위험) |
| `user-invocable: false` | 에이전트는 직접 호출 방지 권장 |

---

## 영역 6: Submodule

### 검토 항목

| 항목 | 기준 | 점수 |
|---|---|---|
| `.gitmodules` 존재 | claude-best-practice submodule 항목 포함 | 4점 |
| `.claude-best-practice/` 비어있지 않음 | `git submodule update --init` 실행 여부 | 3점 |
| 최신 버전 | 로컬 커밋 = 원격 최신 커밋 | 3점 |

submodule이 없어도 Claude Code는 동작하지만, 이 레포의 가이드·스킬을 활용하려면 submodule 추가를 권장한다.

---

## 우선순위 분류 기준

| 우선순위 | 기준 |
|---|---|
| 높음 | 보안 위험 (민감 파일 미제한, 부작용 스킬 미보호) 또는 Claude 동작 불가 수준 |
| 중간 | 베스트 프랙티스 위반 (200줄 초과, `$schema` 누락, description 누락) |
| 낮음 | 권장 개선 사항 (companion file 미사용, `$ARGUMENTS` 미활용 등) |
