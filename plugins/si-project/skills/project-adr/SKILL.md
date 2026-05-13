---
name: project-adr
description: >
  SI 프로젝트 아키텍처 결정 기록(MADR 형식) 1건을 생성한다. docs/02-architecture/adr/ 디렉토리에 다음 번호로 자동 저장.
  코드·DB·인프라·보안 등 주요 결정 시점에 호출. 결정 근거를 영구 보존해 향후 reverting·재검토 시 컨텍스트 제공.
  입력: 결정 제목(한글 또는 영문 자유). 미입력 시 AskUserQuestion.
  트리거: "ADR 작성", "아키텍처 결정 기록", "기술 결정 ADR", "project adr"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(ls *)
  - Bash(Get-ChildItem *)
  - AskUserQuestion
---

# /si-project:project-adr — 아키텍처 결정 기록 (ADR) 생성

MADR(Markdown Architecture Decision Records) 형식의 결정 기록 1건을 생성한다. 결정 시점에 호출해 근거를 영구 보존.

설계 원칙:
- 매우 짧게 (≤ 80줄/ADR). 길면 후대가 안 읽음.
- 결정 자체보다 **근거(Why)** 보존이 본질
- "Accepted"가 된 ADR은 수정 금지. 변경 시 Superseding ADR 신규 작성

---

## STEP 0 — 사전 확인 + lessons 참고

1. `CLAUDE.md` 존재 → 없으면 "`/si-project:project-setup`이 먼저 필요합니다" 중단
2. `docs/02-architecture/adr/` 디렉토리 존재 → 없으면 생성 (project-setup이 만들었어야 하지만 누락 케이스 대비)
3. `.claude/lessons.md` 존재 시 Read → 과거 lesson에 동일 결정 관련 항목 있으면 컨텍스트로 활용

---

## STEP 1 — 결정 제목 수집

사용자가 제공한 제목 사용. 미입력 시 AskUserQuestion으로 수집.

**제목 가이드**:
- 능동·구체적: "PostgreSQL을 RDB로 채택", "Redis 캐시 도입"
- 추상·모호 금지: "DB 결정", "캐시 관련"
- 영문 가능: "Adopt FastAPI for REST API layer"

---

## STEP 2 — ADR 번호 자동 할당

`docs/02-architecture/adr/` 디렉토리에서 가장 큰 번호 + 1 자동 부여.

```
Glob pattern="docs/02-architecture/adr/[0-9][0-9][0-9][0-9]-*.md"
→ 마지막 파일 번호 추출 (예: 0007 → 다음은 0008)
→ 디렉토리 비어있으면 0001 시작
```

파일명: `{NNNN}-{slug}.md`
- slug: 제목을 소문자-하이픈으로 변환 (예: "PostgreSQL을 RDB로 채택" → `0002-postgresql-rdb.md`)
- 한글이 길면 영문 키워드로 축약 권장

---

## STEP 3 — 컨텍스트 수집

다음 소스에서 결정 근거 자동 추출:

| 소스 | 추출 정보 |
|------|----------|
| `CLAUDE.md` | 프로젝트 스택·제약·보안 요구사항 |
| `.claude/project-context.json` | 발주처·계약 acceptance·납기 |
| 사용자 추가 입력 (AskUserQuestion 필요 시) | 검토 대안, 잔존 위험 |

**AskUserQuestion 발동 조건**:
- "Alternatives Considered" 섹션을 자동 채울 수 없으면 사용자에게 검토 대안 1~3건 질문
- 단, 사용자가 명시적으로 충분한 컨텍스트를 제공하면 스킵

---

## STEP 4 — ADR 생성 (MADR 형식)

```markdown
---
title: "ADR-{NNNN}: {결정 제목}"
status: Accepted
date: {YYYY-MM-DD}
deciders: [{owner from CLAUDE.md or TBD}]
tags: [{관련 컴포넌트: backend/user-fe/.../db/security}]
---

# ADR-{NNNN}: {결정 제목}

## Status

**Accepted** (YYYY-MM-DD)

> 변경 시 신규 ADR로 Supersede. 본 ADR은 수정 금지.

## Context

{결정이 필요한 배경. CLAUDE.md·project-context에서 추출:
- 어떤 제약·요구사항이 있는가? (DATA_SENSITIVITY, COMPLIANCE, SLA 등)
- 왜 지금 결정해야 하는가?
- 누가 영향받는가? (컴포넌트·이해관계자)}

## Decision

{채택한 안. 명확하고 짧게. 1~3문장 + 필요 시 핵심 수치/버전.}

## Consequences

**긍정**:
- {장점 1~3건}

**부정**:
- {trade-off 1~3건. 정직하게 적을 것 — "단점 없음"은 금지}

**잔존 위험**:
- {모니터링·재검토 필요 항목}

## Alternatives Considered

| 대안 | 기각 사유 |
|------|---------|
| {대안 A} | {짧은 이유} |
| {대안 B} | {짧은 이유} |

(사용자 제공 없으면 STEP 3에서 수집)

## References

- 관련 문서: [software-architecture.md](../software-architecture.md), [security-definition.md](../security-definition.md)
- 외부 자료: {URL 또는 "없음"}
```

---

## STEP 5 — 저장 + 인덱스 갱신

1. 파일 저장: `docs/02-architecture/adr/{NNNN}-{slug}.md`
2. `docs/02-architecture/adr/README.md`의 인덱스 표에 1행 추가:
   ```
   | NNNN | {제목} | Accepted | YYYY-MM-DD |
   ```
   - README의 인덱스 표가 없으면 생성

3. **CHANGELOG.md 1줄 append** (v2.4 패턴 일관):
   - `### Added` 섹션에 `- {YYYY-MM-DD} adr: ADR-{NNNN} {제목} 채택`

---

## STEP 6 — 완료 보고

```
✅ ADR-{NNNN} 생성: docs/02-architecture/adr/{NNNN}-{slug}.md

📋 결정 요약:
- 제목: {제목}
- 상태: Accepted
- 영향 컴포넌트: {tags}

📝 다음 권장:
- 본 결정이 software-architecture.md / security-definition.md 갱신 필요 여부 검토
- 큰 결정이면 팀 리뷰 후 status 확정 (Proposed → Accepted)

⚠️ 수정 정책:
- 본 ADR은 Accepted 후 수정 금지
- 변경 시 신규 ADR 작성, 본 ADR의 status를 "Superseded by ADR-MMMM"로 변경
```

---

## 사용 예시

```
사용자: /si-project:project-adr "PostgreSQL을 RDB로 채택"
Claude:
  → ADR 번호: 0002 (기존 0001-stack-selection.md 다음)
  → CLAUDE.md에서 DATA_SENSITIVITY=금융정보 확인
  → AskUserQuestion: "검토한 대안은? (MySQL, Oracle, MariaDB 등 중 선택)"
  → MADR 형식으로 0002-postgresql-rdb.md 생성
  → adr/README.md 인덱스 추가, CHANGELOG 1줄 append
```

```
사용자: ADR 작성
Claude:
  → 제목 미입력 → AskUserQuestion: "어떤 결정의 ADR을 작성할까요?"
  → 이하 동일 흐름
```

---

## 본 스킬의 비채택 사항 (명시적 경계)

- **자동 결정 감지 안 함**: 코드 변경에서 "이건 ADR감"이라고 자동 판단하지 않음. 사용자가 호출해야만 생성. 자동화하면 ADR이 노이즈로 가득.
- **이력 자동 추적 안 함**: ADR 간 supersede 그래프를 자동 그리지 않음 (단순 참조만).
- **외부 도구 통합 안 함**: log4brains·adr-tools 등 외부 CLI 호출 안 함. 본 스킬은 markdown만 다룸.
