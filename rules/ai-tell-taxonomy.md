# Korean AI-Tell Taxonomy v1.2

Patterns that mark LLM-generated Korean text across 10 categories. Severity: **S1** (critical — remove always) | **S2** (high — remove at 3+ occurrences) | **S3** (low — rhythm adjust only).

**Permanently non-overridable:** A-8 (double passive), C-5 (emoji overuse), all D patterns.

---

## A. Translation-ese (번역투) — S1~S2

| ID | Bad pattern | Fix |
|----|------------|-----|
| A-1 [S1] | ~에 대해 논의할 필요가 있다 | ~를 논의해야 한다 |
| A-2 [S1] | ~을 통해 인사이트를 얻는다 | ~를 분석해 인사이트를 얻는다 |
| A-3 [S1] | 이 문제에 있어서 | 이 문제에서 |
| A-4 [S2] | 확장성이 뛰어나다는 점에서 | 확장성이 뛰어나서 |
| A-5 [S2] | 보안과 관련하여 주의해야 한다 | 보안에 주의해야 한다 |
| A-6 [S2] | 데이터에 기반하여 판단한다 | 데이터로 판단한다 |
| A-7 [S1] | 경쟁력을 가지고 있다 | 경쟁력이 강하다 |
| A-8 [S1] ⛔ | 판단되어진다 | 판단된다 / 판단한다 |
| A-9 [S2] | AI에 의해 생성된 이미지 | AI가 만든 이미지 |
| A-10 [S2] | 높일 수 있다. 줄일 수 있다. 단축할 수 있다. | 높인다. 줄인다. 단축한다. |
| A-11 [S2] | 고객 만족을 위해 노력한다 | 고객이 만족하도록 일한다 |
| A-12 [S2] | 합의가 이루어졌다 | 합의했다 |
| A-13 [S2] | AI 기술 발전 속도 가속화 | AI 기술의 발전 속도가 빨라진다 |
| A-14 [S2] | 그는 보고했다. 그리고 자리에 앉았다. | 그는 보고하고 자리에 앉았다. |
| A-15 [S2] | DeepSeek의 등장은 ~을 보여줍니다 | DeepSeek는 ~을 증명했다 |

---

## B. English Over-citation (영어 용어 과다) — S2

| ID | Bad pattern | Fix |
|----|------------|-----|
| B-1 [S2] | 인공지능(AI)은 거대언어모델(LLM)과 다르다 | 전문 독자 대상이면 1회만 병기, 이후 한국어만 |
| B-2 [S2] | 이 framework를 leverage하여 | 이 체계를 활용해 (단, API·SDK 등 고유명사 유지) |
| B-3 [S2] | 과도한 영어 인용구 | 한국어로 풀어쓰고 출처만 병기 |
| B-4 [S3] | 'AGI'라고 알려진 범용 인공지능 | 범용 인공지능(AGI) |

---

## C. Structural AI Patterns (구조적 패턴) — S1~S2

| ID | Bad pattern | Fix |
|----|------------|-----|
| C-1 [S1] | 첫째, ~. 둘째, ~. 셋째, ~. (전체 지배) | 1~2개만 서술문으로 녹이거나 "우선/다음으로"로 변주 |
| C-2 [S2] | 과도한 불릿 리스트 | 산문으로 녹이기 |
| C-3 [S2] | ## 도입 ## 본론 ## 결론 도식 | 산문형이면 헤딩 제거 |
| C-4 [S2] | 문단마다 첫 문장이 요약 공식 | 사례·장면·인용으로 순서 흐트러뜨림 |
| C-5 [S1] ⛔ | ✅ 🚀 💡 ⚠️ 📊 이모지 남발 | 에세이/리포트에서 전량 제거 |
| C-6 [S2] | 헤딩 아래 한 줄 요약 박스 | 삭제 |
| C-7 [S2] | 먼저 ~ 반면 ~ 결국 ~ 3단 공식 | 3개 중 2개 삭제 |
| C-8 [S2] | "A인가, B인가" 대구 반복 | 3개 중 2개 비대칭으로 재배치 |

---

## D. AI Signature Phrases (AI 관용구) — S1 ⛔ All non-overridable

**Remove on sight:**
- 결론적으로 / 요약하면 / 종합하면 / 정리하자면
- ~라고 할 수 있다 / ~라고 볼 수 있다 / ~라 하겠다
- 매우 중요하다 / 반드시 기억해야 한다 / 시사하는 바가 크다 / 주목할 만하다 / 간과할 수 없다
- 크게 세 가지로 나눌 수 있다 / 다음과 같은 특징을 가진다
- 혁신적인 / 획기적인 / 전례 없는 / ~의 새로운 장을 열다 / ~시대가 도래했다

**S2 variants:**
- D-5: 추상 주어 의인화 — "두 지능의 충돌이 질문을 던집니다" → 실제 행위자로 교체
- D-6: "~할 때입니다 / 시점입니다" → 구체 동사 단언으로

---

## E. Rhythm / Sentence-length Uniformity — S2

- E-1: All sentences same length → insert 10–15 char short sentences per paragraph
- E-2: Repeated endings (~이다. ~이다.) → mix ~다 / ~았다 / ~인 것 / noun-final
- E-3: All paragraphs 3–4 sentences → intentionally mix 1-sentence and 6-sentence paragraphs

---

## F. Over-modification (과도한 수식) — S2

| ID | Bad pattern | Fix |
|----|------------|-----|
| F-1 | 매우 / 정말 / 대단히 / 극히 | 90% delete; replace with specific data |
| F-2 | 중요하고 핵심적인 역할 | keep one modifier |
| F-3 | ~로서의 역할과 기능 | keep one |
| F-4 | ~적 측면 / ~성 / ~화 남발 | "근본적 변화" → "뿌리부터 바뀐다" |
| F-5 | 에이전트적 자율성 / 기술적 안정성 | verb form or concrete noun |

---

## G. Over-hedging (과도한 완곡) — S2

- G-1: ~할 수 있을 것으로 보인다 → ~로 보인다 → ~이다 (assert where possible)
- G-2: ~할 가능성이 있을 수 있다 → keep only one hedge layer

---

## H. Conjunction Overuse (접속사 남발) — S2

- H-1: 또한 / 따라서 / 즉 / 나아가 / 아울러 — remove 70%+
- H-2: 하지만 / 그러나 repeated — delete half, vary with 그런데
- H-3: 이는 ~을 의미한다 — merge into previous sentence
- H-4: 즉 남발 — max 2 per document; vary with 곧 / 말하자면

---

## I. Formal Noun Overuse (형식명사 과다) — S2

- I-1: ~한 것이다 → ~이다 / ~다
- I-2: 점 / 바 / 수 / 데 repeated → concrete noun or delete
- I-3: ~라는 것이다 → ~이다
- I-4: ~할 필요가 있다 → ~해야 한다
- I-5: ~이 필요하다 → specify who does what
- I-6: ~능력 chain (사고 능력, 추론 능력) → verb form; max 2 per doc

---

## J. Visual Decoration Overuse (시각 장식) — S2~S3

- J-1: **bold** overuse in body text → remove almost all
- J-2: quotation marks overuse → only real quotes or special usage
- J-3: dash (—) overuse → max 1–2 per doc; use comma/parenthesis/separate sentence
- J-4: parenthetical asides "(이는 ~을 의미한다)" repeated → merge into body or delete
