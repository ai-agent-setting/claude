# Korean Rewriting Playbook

Substitution recipes for removing AI-tell patterns. Reference: ai-tell-taxonomy.md.

## 0. Core Principles

1. **Fidelity**: Never alter facts, numbers, proper nouns, quoted text, or causal relationships.
2. **Tone match**: Keep the genre register (formal → formal, essay → essay).
3. **Locality**: Edit only the AI-tell span, not the whole sentence.
4. **Naturalness**: Target the median rhythm of an everyday Korean writer — not literary.
5. **Over-editing warning**: If >50% of sentences changed, content may be distorted.

---

## 1. Substitution Recipes by Category

### A. Translation-ese

| Original | Rewritten |
|----------|----------|
| X에 대해 논의한다 | X를 논의한다 / X를 이야기한다 |
| X를 통해 Y한다 | X로 Y한다 / X해서 Y한다 |
| X에 있어서 | X에서 / X를 볼 때 |
| X라는 점에서 | X해서 / X이기 때문에 |
| X와 관련하여 | X에서 / X를 두고 |
| X에 기반하여 | X로 / X를 근거로 |
| 경쟁력을 가지고 있다 | 경쟁력이 있다 / 경쟁력이 강하다 |
| 판단되어진다 | 판단된다 / 판단한다 |
| AI에 의해 생성된 | AI가 만든 |
| 높일 수 있다 | 높인다 (fact) / 높일 여지가 있다 (possibility) |
| X을 위해 Y한다 | X하려고 Y한다 |
| 합의가 이루어졌다 | 합의했다 |
| 기술 발전 속도 가속화 | 기술의 발전 속도가 빨라진다 |
| 그리고 (sentence-initial) | delete / compress with "-고" |

### B. English term translation (common)

| English | Korean |
|---------|--------|
| framework | 체계 / 틀 / 구조 |
| leverage | 활용하다 / 끌어올리다 |
| seamless | 매끄러운 / 끊김 없는 |
| robust | 튼튼한 / 견고한 |
| insight | 통찰 / 시사점 |
| impact | 영향 / 파장 |
| holistic | 전체적 / 총체적 |

### C. Structure

- **첫째/둘째/셋째**: prose → "A다. B도 마찬가지다. 여기에 C가 더해진다."
- **Bullet → prose**: "속도는 빠르고 비용도 낮다. 무엇보다 확장 여지가 크다."
- **Emoji**: remove all in essay/report context.
- **Heading-summary box**: delete.

### D. Signature phrases (delete-first)

| Delete | Alternative |
|--------|------------|
| 결론적으로 | delete |
| 요약하면 / 정리하자면 | delete or "한 줄로 말하면" |
| ~라고 할 수 있다 | ~이다 (if assertable) / ~로 보인다 (if observational) |
| 매우 중요하다 | replace with specific evidence |
| 시사하는 바가 크다 | delete |
| 주목할 만하다 | delete |
| 혁신적인 / 획기적인 | delete or "처음 시도한" / "이전과 다른" |
| ~의 지평을 열다 | delete; describe actual change |

### E. Rhythm

- Insert 1–2 short sentences (10–15 chars) per paragraph: "맞다. 그게 핵심이다."
- Mix endings: ~다 / ~았다 / ~인 것 / noun-final.
- Ban 4–5 consecutive identical endings.

### F. Modifiers

- 매우 / 정말 / 대단히 → delete 90%; use specific data instead.
- Double synonyms → keep one.
- ~적 / ~성 / ~화 → verb or concrete noun: "근본적 변화" → "뿌리부터 바뀐다"

### G. Hedging

- Downgrade where assertable: "~할 수 있을 것으로 보인다" → "~로 보인다" → "~이다"
- Keep one hedge layer only when genuinely uncertain.

### H. Conjunctions

- Sentence-initial 또한 → delete most; vary with "여기에" / "더해".
- 하지만 / 그러나 repeated → alternate or replace with 그런데.

### I. Formal nouns

- ~것이다 → ~이다 / ~다
- ~할 필요가 있다 → ~해야 한다
- ~이 필요하다 → specify subject + verb

### J. Decoration

- **Bold** in body text → remove almost all.
- Quotation marks → real quotes and special usage only.
- Dash (—) → max 1–2 per doc; use comma/parenthesis/separate sentence.

---

## 2. Do-NOT alter

- Proper nouns, product names, model names
- Numbers, units, dates
- Directly quoted text (inside double quotation marks)
- Legal or regulatory citations
- Academic terms (확률적 앵무새, 창발, etc.)

---

## 3. Genre fine-tuning

| Genre | Allowed | Forbidden |
|-------|---------|-----------|
| Column / Essay | short sentences, personal voice, literary analogy | emoji, heavy headings, excess bullets |
| Report | 1-level headings, stats, citations | hype vocabulary |
| Blog post | friendly tone, rhetorical questions | mechanical 첫째/둘째 pattern |
| Formal speech | formal register, literary prose | colloquial, emoji, bullets |
