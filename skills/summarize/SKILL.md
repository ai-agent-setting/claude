---
name: summarize
description: 파일, 변경사항, 또는 코드베이스 일부를 요약한다
context: fork
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
---

$ARGUMENTS 를 요약하라.

다음 형식으로 작성하라:

## 개요
한 문단으로 전체 목적과 구조 설명.

## 핵심 내용
- 주요 기능이나 변경사항 (불릿 리스트)

## 의존관계
- 이 코드가 의존하는 것
- 이 코드에 의존하는 것

## 주목할 점
놓치기 쉬운 중요한 세부사항.
