<!-- last-reviewed: 2026-04-27 -->
# CLAUDE.md 효과적으로 작성하는 방법

CLAUDE.md는 Claude Code가 프로젝트를 열 때 자동으로 읽는 지시 파일이다.
잘 작성된 CLAUDE.md 하나로 매번 같은 설명을 반복하지 않아도 된다.

> 참고: 공식 문서 → https://code.claude.com/docs/en/memory

---

## 핵심 원칙

1. **구체적으로 쓸 것**: "좋은 코드를 작성하라"는 무의미. "함수는 20줄 이하로 제한하라" 처럼 측정 가능하게.
2. **200줄 이내로 유지할 것**: 공식 권장 기준. 길수록 순응도가 떨어진다. 넘으면 `@import` 또는 `.claude/rules/`로 분리.
3. **프로젝트 특화 정보만 담을 것**: Claude가 코드를 보면 알 수 있는 내용, 일반 관행은 제외.
4. **명령형으로 쓸 것**: "~하라" 형태가 "~해주세요"보다 명확하다.
5. **git에 커밋할 것**: 팀 전체가 혜택을 받는다.

> 팁: Claude Code에서 `/init`을 실행하면 CLAUDE.md 초안을 자동 생성해 준다.
> `CLAUDE_CODE_NEW_INIT=1` 환경 변수를 설정하면 CLAUDE.md + skills + hooks를 한 번에 설정하는 인터랙티브 플로우가 활성화된다.

---

## 포함할 것 vs 제외할 것

| 포함할 것 | 제외할 것 |
|---|---|
| Claude가 추측할 수 없는 Bash 명령어 | Claude가 코드를 보면 알 수 있는 것 |
| 기본값과 다른 코드 스타일 규칙 | 표준 언어 관행 |
| 테스트 방법 및 선호 테스트 러너 | 자주 변하는 정보 |
| 팀 에티켓 (브랜치, PR 규칙 등) | 긴 설명이나 튜토리얼 |
| 아키텍처 결정 | "깔끔한 코드를 작성하라" 같은 자명한 규칙 |

---

## 권장 최소 구성

```markdown
# 프로젝트 이름

## 프로젝트 개요
한 두 문장으로 목적과 기술 스택 요약.

## 핵심 명령어
```bash
npm run dev    # 개발 서버
npm test       # 테스트
npm run build  # 빌드
```

## 코드 규칙
들여쓰기, 네이밍 등 프로젝트 고유 규칙.
```

---

## 파일 임포트 (`@` 구문)

CLAUDE.md에서 `@path/to/file` 구문으로 다른 파일을 임포트할 수 있다.

```markdown
@README.md
@package.json
- API 설계: @docs/api-design.md
```

> 외부 임포트는 첫 사용 시 Claude Code가 승인 다이얼로그를 표시한다.

### @import 경로 해석 규칙

- 경로는 **작업 디렉터리 기준이 아닌 해당 파일 위치 기준**으로 해석된다.
- 재귀 임포트는 최대 **5단계**까지 허용된다.
- `@import`는 파일 구조를 정리하는 효과만 있고 컨텍스트 절감 효과는 없다. 컨텍스트를 줄이려면 path-scoped rules를 활용하라.

### AGENTS.md 호환 패턴

다른 에이전트가 `AGENTS.md`를 사용하는 경우, CLAUDE.md에서 임포트하여 중복 없이 활용할 수 있다.

```markdown
# CLAUDE.md
@AGENTS.md
```

---

## `.claude/rules/` 모듈형 규칙 정리

규칙이 많아지면 `.claude/rules/` 디렉터리에 주제별로 분리한다.

```
.claude/
├── CLAUDE.md           # 핵심 지침 (간결하게)
└── rules/
    ├── code-style.md   # 코드 스타일
    ├── testing.md      # 테스트 규칙
    └── security.md     # 보안 요구사항
```

### User-level rules (개인 전역 규칙)

`~/.claude/rules/`에 개인용 전역 rules를 설정할 수 있다.
프로젝트 rules보다 먼저 로드되며 모든 프로젝트에 적용된다.

```
~/.claude/
└── rules/
    ├── personal-style.md   # 개인 코딩 스타일
    └── workflow.md         # 개인 워크플로 규칙
```

symlink를 활용하면 여러 프로젝트에서 같은 rules 파일을 공유할 수 있다:

```bash
ln -s ~/shared-rules/security.md .claude/rules/security.md
```

### Path-specific rules (경로별 조건부 규칙)

YAML frontmatter의 `paths` 필드로 특정 파일에만 적용되는 규칙을 만들 수 있다.

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API 개발 규칙
- 모든 엔드포인트에 입력 유효성 검사 필수
- 표준 에러 응답 형식 사용
```

`paths` 필드가 없는 규칙은 모든 파일에 무조건 로드된다.

---

## CLAUDE.md 배치 위치

| 위치 | 경로 | 적용 범위 |
|---|---|---|
| 프로젝트 (팀 공유) | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | git으로 공유 |
| 개인 (모든 프로젝트) | `~/.claude/CLAUDE.md` | 내 모든 프로젝트 |
| 개인 (이 프로젝트만, 비공개) | `./CLAUDE.local.md` | git 제외, 자동 `.gitignore` |
| 조직 전체 정책 (macOS) | `/Library/Application Support/ClaudeCode/CLAUDE.md` | 엔터프라이즈 관리형 |
| 조직 전체 정책 (Linux/WSL) | `/etc/claude-code/CLAUDE.md` | 엔터프라이즈 관리형 |
| 조직 전체 정책 (Windows) | `C:\Program Files\ClaudeCode\CLAUDE.md` | 엔터프라이즈 관리형 |

---

## 대형 모노레포 설정

대형 모노레포에서 무관한 팀의 CLAUDE.md를 제외하려면 `claudeMdExcludes` 설정을 사용한다.
`.claude/settings.local.json`에 추가:

```json
{
  "claudeMdExcludes": [
    "teams/frontend/**",
    "teams/data-science/**"
  ]
}
```

---

## 문제 해결

| 증상 | 원인 | 해결책 |
|---|---|---|
| Claude가 규칙을 반복적으로 어김 | CLAUDE.md가 너무 길어 규칙이 묻힘 | 무자비하게 줄이기. "IMPORTANT"/"YOU MUST" 사용 |
| `/compact` 후 지침이 사라짐 | 대화에서만 말하고 CLAUDE.md에 없음 | CLAUDE.md에 추가 |
| 규칙 충돌 | 여러 파일에 상충되는 지침 | `/memory`로 로드 목록 확인 후 정리 |
| 어떤 rules 파일이 로드됐는지 모름 | path-specific rules 디버깅 필요 | `InstructionsLoaded` 훅으로 로그 확인 |

---

## 참고

- 템플릿: [../claude-md-templates/general.md](../claude-md-templates/general.md)
- Skills (고급 확장): [skills-and-commands.md](skills-and-commands.md)
