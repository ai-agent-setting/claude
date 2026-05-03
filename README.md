# Claude Code 사용 워크플로

> last-reviewed: 2026-04-30

---

## 세션 라이프사이클

### 시작

별도 준비 없이 바로 작업한다. Codex 플러그인의 `SessionStart` 훅이 백그라운드에서 런타임을 초기화한다.

### 작업 중

Claude가 응답을 완료하면 macOS 알림이 뜬다 → 다른 창에서 작업 중에도 완료 여부를 확인할 수 있다.
Claude가 입력이나 허가를 기다릴 때도 알림이 온다 → 자리를 비웠다가 돌아와도 놓치지 않는다.

컨텍스트가 약 70%(~140K 토큰)에 도달하면 별도 알림이 온다. 이 시점에 `/checkpoint`를 실행해 세션을 정리하고 `/clear`로 새 세션을 시작한다.

### 종료 전

Codex Stop-time Review Gate가 활성화되어 있어, Claude가 코드를 수정한 턴이 끝날 때마다 Codex가 변경사항을 자동 검증한다. 이슈가 있으면 `BLOCK`을 반환한다.

---

## 자주 쓰는 스킬

| 스킬 | 언제 | 설명 |
|------|------|------|
| `/checkpoint` | 컨텍스트 70% 알림 시 | 세션 정리 + 로그 기록 + 재개 프롬프트 출력 |
| `/review` | 코드 변경 후 | 코드·변경사항 리뷰 |
| `/summarize` | 파악이 필요한 파일·범위 | 코드베이스 일부 요약 |
| `/explain` | 개념·코드 이해가 필요할 때 | 쉽게 풀어서 설명 |
| `/review-claude-config` | 설정 변경 후 | ~/.claude 설정 품질 전체 검토 |
| `/check-freshness` | 주기적으로 | guides/ 파일 최신성 감사 |
| `/update-guides` | 주기적으로 | 공식 문서 기반 가이드 업데이트 제안 |

---

## Codex 연동 (OpenAI Codex 플러그인)

Claude Code 안에서 Codex를 보조 리뷰어·실행자로 쓴다. Codex CLI(`gpt-5.5`, reasoning_effort=medium)가 별도 프로세스로 실행되어 Claude의 컨텍스트와 충돌하지 않는다.

| 커맨드 | 언제 |
|--------|------|
| `/codex:review` | 커밋 전 표준 코드 리뷰 |
| `/codex:adversarial-review` | 보안·설계 약점 탐색 (인증, 경쟁 조건, 롤백 안전성 등) |
| `/codex:rescue` | 디버깅·수정 작업을 Codex에 위임 |
| `/codex:status` / `/codex:result` | 백그라운드 작업 진행 확인 |

백그라운드 활용: `/codex:review --background` 실행 후 Claude로 다른 작업 → 나중에 `/codex:result`로 확인.

---

## 자동화 훅 요약

| 이벤트 | 동작 |
|--------|------|
| `Stop` | "답변 완료" macOS 알림 |
| `Notification` | "입력·허가 대기" macOS 알림 |
| `PreCompact` | "컨텍스트 70% 도달" 알림 + 자동 압축 차단 |
| Codex `Stop` | Stop-time Review Gate (코드 변경 시 자동 검증) |
| Codex `SessionStart/End` | Codex 런타임 초기화·정리 |

---

## 전역 행동 규칙 (CLAUDE.md 요약)

- **한국어 AI 티 방지**: 번역투·관용구·구조 패턴 자체 점검 후 제거
- **작업 전 확인**: 구현 방식이 둘 이상이거나 범위가 불명확하면 먼저 질문
- **작업 로그 자동 기록**: 설정·스킬 수정, 오류 해결 시 `logs/`에 자동 기록
- **훅 LLM 호출 금지**: 훅 커맨드 안에서 Claude API·claude CLI 호출 불가

---

## 설정 파일 지도

```
~/.claude/
├── CLAUDE.md              # 전역 행동 규칙
├── settings.json          # 훅, 플러그인, 모델, autoCompact 설정
├── rules/
│   ├── ai-tell-taxonomy.md    # 한국어 AI 티 분류 체계
│   └── rewriting-playbook.md  # 윤문 처방집
├── skills/                # 재사용 스킬 (checkpoint, review 등)
├── guides/                # 참조 가이드 문서
├── logs/                  # 자동 작업 로그 (worklog, troubleshooting, decisions)
├── memory/                # 세션 간 기억 (user, feedback, project, reference)
└── plugins/               # Codex 등 외부 플러그인
```
