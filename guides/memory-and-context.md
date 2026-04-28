<!-- last-reviewed: 2026-04-27 -->
# 컨텍스트 윈도우 및 메모리 관리

Claude Code에는 두 가지 메모리 시스템이 있다:
- **CLAUDE.md**: 내가 직접 작성하는 지속 지침 파일
- **Auto Memory**: Claude가 세션 경험을 바탕으로 자동으로 작성하는 노트

> 참고: 공식 문서 → https://code.claude.com/docs/en/memory

---

## CLAUDE.md vs Auto Memory

| | CLAUDE.md | Auto Memory |
|---|---|---|
| 작성자 | 사람 | Claude |
| 내용 | 지침과 규칙 | 학습 내용과 패턴 |
| 로드 방식 | 루트 CLAUDE.md는 세션마다 전체 로드. 하위 디렉터리 CLAUDE.md는 on-demand 로드 | 세션마다 상위 200줄 또는 25KB 중 먼저 도달하는 것 로드 |
| 전달 방식 | 시스템 프롬프트가 아닌 사용자 메시지로 전달됨 | — |
| 용도 | 코딩 표준, 워크플로, 아키텍처 | 빌드 명령, 디버깅 인사이트 |

---

## Auto Memory (자동 메모리)

Claude가 교정 내용, 선호도, 빌드 명령, 디버깅 패턴을 스스로 저장한다.
미래 대화에 유용할 것이라 판단할 때만 저장한다.

### 저장 위치

```
~/.claude/projects/<project>/memory/
├── MEMORY.md              # 인덱스 파일 (세션 시작 시 상위 200줄 또는 25KB 로드)
├── debugging.md           # 디버깅 패턴
└── api-conventions.md     # API 설계 결정
```

`<project>` 경로는 git 레포를 기준으로 파생된다.
- git 레포 밖에 있으면 프로젝트 루트 경로를 사용한다.
- git worktree와 서브디렉터리는 모두 같은 auto memory 디렉터리를 공유한다.

`MEMORY.md`의 상위 200줄 또는 25KB(먼저 도달하는 기준) 만큼 세션마다 자동 로드된다.
나머지 파일은 Claude가 필요 시 on-demand로 읽는다.

### Auto Memory 관리

```
/memory    # 로드된 파일 목록 + auto memory 토글 + auto memory 폴더 열기 링크
```

`/memory` 명령으로 할 수 있는 것:
- 현재 로드된 CLAUDE.md, CLAUDE.local.md, rules 파일 목록 확인
- Auto Memory 활성화/비활성화 토글
- Auto memory 폴더 열기 링크
- 파일 선택 시 에디터에서 열기

Auto Memory는 plain markdown이므로 언제든 직접 편집·삭제할 수 있다.

### Auto Memory 저장 위치 변경

`autoMemoryDirectory` 설정으로 저장 경로를 변경할 수 있다.
이 설정은 user/local settings에서만 허용되며 project settings에서는 설정 불가.

```json
{
  "autoMemoryDirectory": "/custom/path/to/memory"
}
```

### Auto Memory 끄기

settings.json에 추가:
```json
{
  "autoMemoryEnabled": false
}
```

또는 환경 변수: `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`

---

## CLAUDE.md 크기 가이드라인

공식 권장: 파일당 200줄 이내.

| 프로젝트 규모 | 전략 |
|---|---|
| 소형 (문서, 학습) | 루트 CLAUDE.md 100줄 이내 |
| 중형 (일반 웹앱) | 루트 150줄 + `.claude/rules/` 분리 |
| 대형 (복잡한 시스템) | 루트 CLAUDE.md 간결히 + path-specific rules로 분산 |

### CLAUDE.md HTML 주석 활용

CLAUDE.md 내 block-level HTML 주석(`<!-- -->`)은 컨텍스트 주입 전 자동으로 제거된다.
메타 정보나 주석을 남기되 Claude에게 전달하지 않을 때 활용한다.

```markdown
<!-- last-reviewed: 2026-04-27 -->
<!-- 이 섹션은 백엔드 팀 전용입니다. 프론트엔드 개발 시 무시. -->
# 프로젝트 지침
```

> 주의: 코드블록 내 주석은 그대로 보존된다.

---

## 컨텍스트 윈도우 관리

### `/clear` 가장 강력한 도구

```
/clear    # 컨텍스트 완전 초기화 (CLAUDE.md는 재로드됨)
```

사용 시기:
- 무관한 작업으로 전환할 때
- Claude가 같은 실수를 두 번 이상 반복할 때
- 응답이 점점 모호·일반적이 될 때

### `/compact` 핵심만 남기고 압축

```
/compact                         # 자동 압축
/compact API 변경사항에 집중      # 지침 포함 압축
```

자동 compaction은 컨텍스트가 95% 찰 때 트리거된다.

`/compact` 후 루트 CLAUDE.md는 자동으로 재주입되지만, 하위 디렉터리의 CLAUDE.md는 재주입되지 않는다. compaction 이후에도 하위 rules가 유지되어야 한다면 루트 CLAUDE.md에 핵심 내용을 포함시켜라.

CLAUDE.md에 compaction 지침을 추가하면 압축 시 보존할 내용을 제어할 수 있다:

```markdown
<!-- compaction 지침 -->
When compacting, always preserve:
- The full list of modified files
- Current task progress and next steps
```

### `/rewind` 체크포인트로 되돌리기

Claude가 변경할 때마다 자동으로 체크포인트가 생성된다.
`Esc + Esc` 또는 `/rewind`로 체크포인트 메뉴를 열 수 있다.

복원 옵션 4가지:
1. **대화만 복원** — 코드 변경은 유지하고 대화 히스토리만 되감기
2. **코드만 복원** — 대화는 유지하고 파일만 체크포인트 시점으로 복원
3. **둘 다 복원** — 대화와 코드 모두 체크포인트 시점으로 복원
4. **선택 메시지부터 요약** — 특정 시점 이후를 압축하여 요약

> 체크포인트는 세션 종료 후에도 유지된다.

---

## 추가 디렉터리 CLAUDE.md 로드

`--add-dir`로 추가한 디렉터리의 CLAUDE.md는 기본적으로 로드되지 않는다.
로드하려면 환경변수를 설정해야 한다:

```bash
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir /other/project
```

---

## 세션 재개

```bash
claude --continue    # 최근 대화 재개
claude --resume      # 대화 목록에서 선택
```

---

## 컨텍스트 리셋이 필요한 신호

- Claude가 앞서 정한 규칙을 어기기 시작할 때
- 응답이 점점 일반적·모호해질 때
- 같은 오류를 두 번 이상 교정했을 때

---

## 문제 해결

### 어떤 instruction 파일이 로드됐는지 확인

`InstructionsLoaded` 훅을 사용하면 어떤 파일이 언제 로드됐는지 로깅할 수 있다.
path-specific rules 디버깅에 유용하다.

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
