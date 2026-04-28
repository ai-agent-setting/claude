<!-- last-reviewed: 2026-04-27 -->
# 자주 하는 실수와 안티패턴

> 참고: 공식 문서 → https://code.claude.com/docs/en/best-practices#avoid-common-failure-patterns

---

## CLAUDE.md 관련

### 부엌 싱크대 세션 (Kitchen sink session)

증상: 하나의 작업을 하다가 무관한 것을 물어보고, 다시 첫 번째 작업으로 돌아간다.
해결: 무관한 작업 사이에 `/clear`를 실행하라.

### CLAUDE.md를 너무 길게 쓴다 (Over-specified CLAUDE.md)

증상: 수백줄의 CLAUDE.md를 작성했지만 Claude가 중간 내용을 무시한다.
원인: 공식 권장은 200줄 이내.
해결: 핵심만 남기고 `.claude/rules/`나 `@import`로 분리. 중요 규칙에는 "IMPORTANT"/"YOU MUST" 사용.

### `@import`로 컨텍스트를 줄이려 한다

오해: `@import`로 파일을 분리하면 컨텍스트가 절감된다.
사실: `@import`는 구조 정리 효과만 있고 컨텍스트 절감 효과는 없다. 임포트된 파일 전체가 로드된다.
해결: 컨텍스트를 줄이려면 path-scoped rules를 활용하라. 해당 파일이 편집될 때만 로드된다.

### `/compact` 후 지침이 사라진다

원인: 그 지침을 대화에서만 말했고 CLAUDE.md에 추가하지 않았다.
해결: CLAUDE.md에 추가하라. compaction에서 완전히 살아남는다.

추가 팁: CLAUDE.md에 compaction 지침을 추가하면 압축 시 보존할 내용을 제어할 수 있다:

```markdown
When compacting, always preserve:
- The full list of modified files
- Current task progress and next steps
- Active decisions and their rationale
```

### 빌드/테스트 명령어를 넣지 않는다

증상: Claude가 코드 수정 후 어떻게 테스트하는지 매번 묻는다.
해결: 항상 명령어 코드블록 포함.

---

## 프롬프트 관련

### 같은 오류를 두 번 이상 교정한다

원인: 컨텍스트가 실패 시도로 오염된다.
해결: `/clear` 후 더 구체적인 프롬프트로 재시작.

### 신뢰 후 검증 갭 (Trust-then-verify gap)

증상: Claude가 그럴듯해 보이지만 엣지 케이스를 처리하지 않는 구현을 제시한다.
해결: 항상 검증(테스트, 스크린샷, 스크립트) 제공. 검증 불가하면 배포하지 마라.

### 무한 탐색 (Infinite exploration)

증상: Claude가 수백 개의 파일을 읽어 컨텍스트를 소비한다.
해결: 조사 범위를 좁게 지정하거나 서브에이전트에 위임하라.

### 에러 메시지 없이 "왜 안 되나요"라고 묻는다

해결: 에러 전문 + 실행 명령어 + 관련 코드를 함께 제공.

---

## 세션 관리 관련

### 병렬 세션 없이 복잡한 작업

해결: 구현 세션과 리뷰 세션을 분리하라. 리뷰 Claude는 자신이 방금 작성한 코드에 편향되지 않는다.

---

## Skills 관련

### SKILL.md를 너무 크게 만든다

공식 권장: SKILL.md는 500줄 이내. 세부 내용은 별도 파일로 분리.

### 부작용이 있는 작업에 disable-model-invocation 미설정

증상: Claude가 "준비된 것 같다"고 판단해서 자동으로 `/deploy` 실행.
해결: 배포, 커밋, 외부 알림 등에는 반드시 `disable-model-invocation: true` 설정.
