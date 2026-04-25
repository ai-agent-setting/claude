<!-- last-reviewed: 2026-04-05 -->
# 새 프로젝트에 Claude 세팅하는 절차

> 참고: 공식 문서 → https://code.claude.com/docs/en/best-practices#configure-your-environment

---

## 빠른 세팅

```bash
cd your-project
claude
/init   # 코드베이스 분석 후 CLAUDE.md 자동 생성
```

---

## 단계별 상세 설명

### Step 1: `/init`으로 자동 생성 (권장)

```
claude      # Claude Code 실행
/init       # CLAUDE.md 자동 생성
```

Claude가 빌드 시스템, 테스트 프레임워크, 코드 패턴을 분석하여 초안을 만든다.
이미 CLAUDE.md가 있다면 개선안을 제안한다.

> 팁: `CLAUDE_CODE_NEW_INIT=1` 환경 변수를 설정하면 `/init`이 CLAUDE.md + skills + hooks를 한 번에 설정하는 인터랙티브 멀티 페이즈 플로우로 실행된다.
>
> ```bash
> CLAUDE_CODE_NEW_INIT=1 claude
> /init
> ```

### Step 2: CLAUDE.md 수정

- [ ] 프로젝트 이름, 개요 (한 두 문장)
- [ ] 실제 빌드/실행/테스트/린트 명령어
- [ ] 프로젝트 특화 규칙
- [ ] 핵심 디렉터리/파일 구조

CLAUDE.md 작성 요령 → [claude-md-authoring.md](claude-md-authoring.md)

### Step 3: `.claude/skills/` 설정 (선택)

```
.claude/
├── skills/
│   ├── review/SKILL.md
│   ├── summarize/SKILL.md
│   └── explain/SKILL.md
└── settings.json
```

자세히: [skills-and-commands.md](skills-and-commands.md)

### Step 4: 권한 설정 (선택)

`/permissions` 명령으로 allowlist를 관리하거나, 직접 settings.json에 작성한다:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(npm run lint)", "Bash(npm test *)"],
    "deny": ["Read(./.env)", "Bash(curl *)"]
  }
}
```

유용한 권한 관련 명령:
- `/permissions` — allowlist 대화형 관리
- `/sandbox` — OS 레벨 격리 (sandbox 모드 활성화)
- auto mode — 권한 요청 자동 승인 (신뢰할 수 있는 환경에서만 사용)

### Step 5: 검증

- [ ] "이 프로젝트가 뭐야?" → CLAUDE.md 내용을 바르게 인식하는가?
- [ ] 빌드/테스트 명령어를 올바르게 실행하는가?
- [ ] `/memory`로 로드된 파일 목록 확인

---

## CLAUDE.local.md 개인 설정 (비공개)

```markdown
# CLAUDE.local.md (git 제외됨)
- 내 로컬 개발 서버: http://localhost:3001
```

---

## 참고

- [Skills 가이드](skills-and-commands.md)
