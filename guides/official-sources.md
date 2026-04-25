<!-- last-reviewed: 2026-04-05 -->
# 공식 문서 및 업데이트 소스

Claude Code 관련 공식 문서 URL과 업데이트 모니터링 방법.

---

## 공식 문서 URL 목록

| 문서 | URL | 설명 |
|---|---|---|
| 개요 | https://code.claude.com/docs/en/overview | Claude Code 전반 소개 |
| 메모리 & CLAUDE.md | https://code.claude.com/docs/en/memory | CLAUDE.md, Auto Memory, Rules |
| Skills | https://code.claude.com/docs/en/skills | Skills 시스템, SKILL.md 형식 |
| 베스트 프랙티스 | https://code.claude.com/docs/en/best-practices | 공식 권장 워크플로 |
| 설정 | https://code.claude.com/docs/en/settings | settings.json 필드 전체 |
| 릴리즈 노트 | https://code.claude.com/docs/en/release-notes | 버전별 변경사항 |
| 훅 | https://code.claude.com/docs/en/hooks | 생명주기 훅 상세 가이드 |
| 서브에이전트 | https://code.claude.com/docs/en/sub-agents | 서브에이전트 설정 및 활용 |
| 권한 설정 | https://code.claude.com/docs/en/permissions | allowlist, sandbox, auto mode |
| 세션 관리 | https://code.claude.com/docs/en/sessions | 세션 재개, /rewind, /compact |
| 공통 워크플로 | https://code.claude.com/docs/en/common-workflows | 자주 쓰는 워크플로 패턴 |
| 내장 명령 | https://code.claude.com/docs/en/commands | 내장 명령 레퍼런스 |
| 플러그인 | https://code.claude.com/docs/en/plugins | 플러그인 개발 및 사용 |

> 구버전 URL (https://docs.anthropic.com/... claude-code/...)은 더 이상 사용하지 않는다.

---

## 가이드 최신 여부 확인

이 레포 각 가이드 파일 상단에 `<!-- last-reviewed: YYYY-MM-DD -->` 태그가 있다.

Claude Code의 `/check-freshness` 스킬 실행:
```
/check-freshness
```

수동 확인:
1. https://code.claude.com/docs/en/release-notes 에서 최신 변경사항 확인
2. 영향을 받는 가이드의 `last-reviewed` 날짜 확인
3. 날짜가 오래됐으면 `/update-guides [주제]` 실행

---

## 가이드 업데이트 절차

### `/update-guides` 스킬 사용

```
/update-guides memory
/update-guides skills
/update-guides all
```

이 스킬은:
1. 공식 문서를 WebFetch로 가져온다
2. 현재 가이드와 비교한다
3. 변경이 필요한 부분의 diff를 제안한다
4. 직접 파일을 수정하지 않는다 — 리뷰 후 직접 적용하라

### 수동 업데이트

1. 공식 문서 URL에서 변경사항 파악
2. 관련 가이드 파일 수정
3. `<!-- last-reviewed: YYYY-MM-DD -->` 날짜 업데이트
4. `CHANGELOG.md` 업데이트
5. commit & push

---

## 변경이 많은 영역

자주 변경되는 영역을 우선 확인하라:

- **Skills**: 기능이 활발히 개발 중 (frontmatter 필드, 내장 스킬)
- **Settings**: 새 필드가 자주 추가됨
- **Sub-agents**: 에이전트 타입, 파일 형식 변경 가능성
- **Release Notes**: 버전마다 새 기능 발표

안정적인 영역 (자주 바뀌지 않음):
- CLAUDE.md 작성 원칙
- 기본 프롬프트 엔지니어링 원칙
