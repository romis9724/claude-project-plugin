---
name: project-milestone
description: >
  SI 마일스톤(단계)의 필수 산출물을 일괄 생성하고 일관성을 점검한다.
  발주처 단계별 납품 시점에 호출.
  입력: 마일스톤 식별자(00-kickoff ~ 07-delivery) 또는 한글(요구분석, 아키텍처 등).
  트리거: "마일스톤 산출물 일괄", "단계 납품물 생성", "요구분석 산출물 일괄", "아키텍처 번들 생성", "project milestone"
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
  - TodoWrite
  - Agent
---

# /si-project:project-milestone — 마일스톤 산출물 번들 생성

특정 마일스톤에 발주처가 요구하는 필수 문서들을 일괄 생성하고 문서간 일관성을 보장한다.

---

## STEP 0 — 사전 확인 + lessons 참고

- `CLAUDE.md` 존재 여부 → 없으면 `/si-project:project-setup` 안내 후 중단
- 사용자 입력 마일스톤 식별
- **lessons.md 참고 (v2.4)**: `.claude/lessons.md` 존재 시 Read해 컨텍스트로 반영. 일관성 점검 시 lesson에 기록된 과거 불일치 패턴이 있으면 우선 검증.

### 설계 결정: 작성은 순차, 검토는 subagent (v2.2 갱신)

산출물 *작성*은 의도적으로 **단일 conversation 순차 생성**을 유지한다.
- 문서간 일관성 점검(STEP 5)이 본 스킬의 핵심 가치
- 작성용 subagent 병렬 실행 시 각 agent가 다른 문서의 출력을 보지 못해 cross-reference(용어·요구사항 ID·이해관계자 명칭) 손실
- 속도 < 정합성 — SI 산출물 표준에 맞는 선택

산출물 *작성 후 도메인 검토*는 v2.2에서 subagent로 분리한다.
- 검토는 단일 산출물 대상이므로 cross-reference 문제 없음
- 작성·검토 컨텍스트 분리로 자기검증 한계 완화
- 좁은 도메인 페르소나가 검토 한정에서는 효과 발휘
- 부모 컨텍스트 보호: 검토자 응답 1줄만 보존

**마일스톤 매핑**:
| 입력 | 정규화 |
|------|--------|
| `00-kickoff`, `착수`, `kickoff` | 00-kickoff |
| `01-requirements`, `요구분석`, `requirements` | 01-requirements |
| `02-architecture`, `아키텍처`, `architecture` | 02-architecture |
| `03-design`, `설계`, `design` | 03-design |
| `04-implementation`, `구현`, `implementation` | 04-implementation |
| `05-testing`, `테스트`, `testing` | 05-testing |
| `06-management`, `관리`, `management` | 06-management |
| `07-delivery`, `인도`, `이관`, `delivery` | 07-delivery |

---

## STEP 1 — 필수 문서 목록 추출 (lazy, 섹션 단위)

`{플러그인_경로}/reference/doc-catalog.md` **전체 Read 금지** (v2.4).

```
# 1단계: 섹션 헤더 위치 확인
Grep pattern="^## {milestone}" -n
# → 시작 라인 N 확보 (예: ## 01-requirements → line 39)

# 2단계: 다음 섹션 헤더로 끝 위치 확보
Grep pattern="^## [0-9]" -n     # 모든 섹션 헤더 위치
# → milestone 다음 헤더 라인 M, 또는 "## 합계" 라인

# 3단계: 부분 Read
Read offset=N, limit=(M-N)
```

- 해당 마일스톤의 모든 문서 추출 (~10-25줄)
- `필수도 = 필수` 항목 우선 (필요 시 `권장` 포함 여부 사용자 확인)

TodoWrite로 각 문서를 task로 등록 (진행 추적).

---

## STEP 2 — 컨텍스트 일괄 수집 (`.claude/project-context.json`)

마일스톤 단위 일괄 생성이므로 project-document 스킬과 달리 **한 번에 모든 필요 행정 정보를 수집**해 효율을 높인다.

### 2-1. 컨텍스트 파일 확인
- 없으면 신규 생성
- 있으면 기존 내용 로드

### 2-2. 마일스톤별 필요 정보 합산
이번 마일스톤의 필수 문서들이 공통으로 요구하는 행정 정보를 사전 정의에서 추출:

| 마일스톤 | 일괄 수집 항목 |
|---------|-------------|
| 00-kickoff | customer.*, contract.*, schedule.kickoff_date/delivery_date, resource.total_mm/partners/customer_involvement, stakeholders[], methodology.* |
| 01-requirements | customer.name, contract.acceptance, stakeholders[] (현업 검토자) |
| 02-architecture | (대부분 CLAUDE.md에서 추출) |
| 03-design | (대부분 CLAUDE.md·소스에서 추출) |
| 04-implementation | resource.total_mm, methodology.name(스프린트 여부) |
| 05-testing | contract.acceptance, delivery_format.stages |
| 06-management | resource.partners, methodology.standards |
| 07-delivery | operations.owner/runtime/availability/rto_rpo, delivery_format.user_training, customer.contact_person |

### 2-3. 누락 정보 일괄 질문
컨텍스트 파일에서 누락된 항목들을 묶어 AskUserQuestion 1~3회로 일괄 수집 (개별 project-document 호출보다 효율적).

수집 결과로 `.claude/project-context.json` 업데이트.

> 컨텍스트 파일의 상세 구조는 `/si-project:project-document` 스킬의 STEP 3-2 참조.

---

## STEP 3 — 기존 문서 점검

각 필수 문서에 대해:
- `docs/{milestone}/{file}.md` 존재 여부 확인
- 기존 문서 발견 시 AskUserQuestion으로 일괄 정책 결정:
  - 모두 덮어쓰기
  - 모두 스킵 (기존 유지)
  - 모두 병합 (기존 + 신규 정보)
  - 개별 확인 (각 파일마다 물음)

---

## STEP 4 — 일괄 생성 (순차, lazy 로드)

각 누락 문서에 대해 `project-document` 스킬과 동일한 로직으로 생성:
1. 템플릿 로드
2. **methodology.md에서 작성 가이드 (v2.4 마커 기반 lazy 로드)**:
   - `Grep pattern="<!-- DOC: {file_stem} -->" -n` → 시작 라인 N
   - `Grep pattern="<!-- /DOC: {file_stem} -->" -n` → 종료 라인 M
   - `Read offset=N, limit=(M-N+1)` → 해당 섹션만 로드
   - 마커 없는 문서는 생략
3. `.claude/project-context.json` + CLAUDE.md + 소스 분석
4. 생성 (YAML 프론트매터 + 섹션 채움 + TODO 표기)
5. **도메인 체크리스트 검토자 subagent 호출 (v2.2 / v2.3.1 플래그 반영)** — methodology.md DOC 블록에 `**도메인 검토 체크리스트**`가 있는 산출물만:
   - **플래그 확인 (v2.3.1)**: 마일스톤 시작 시 한 번 `Grep pattern="^REVIEWER_SUBAGENT:" path="CLAUDE.md"`로 정책 확인 (캐싱)
     - `off`: 마일스톤 전체 검토자 호출 스킵
     - `critical-only` (기본): opus 정책 산출물만 호출, sonnet 정책은 스킵
     - `on`: 체크리스트 보유 산출물 모두 호출
   - Agent 호출: `subagent_type=general-purpose`, `model=opus` (requirements/software-architecture/security-definition) 또는 `model=sonnet` (그 외)
   - 프롬프트 형식은 `/si-project:project-document` SKILL의 STEP 5-5 참조
   - 결과 1줄을 산출물별로 누적 보관 (STEP 5에서 합산)
   - 실제 호출되어 결과를 받은 산출물은 파일 footer에 `<!-- review-history ... -->` 블록 append/update (v2.3.1, 형식은 project-document STEP 6 참조)
   - subagent 실패 시 해당 산출물은 `검토 실패`로 마킹 (footer 누적 안 함)
6. 저장 + TodoWrite 완료 표시

> 산출물 *작성* 자체는 **순차** 처리. 작성용 subagent는 띄우지 않는다 (STEP 0 설계 결정 참조).
> 검토자 subagent만 산출물별로 호출 — cross-reference 없는 단독 검토라 격리 가능.

**일관성을 위한 공통 컨텍스트**:
- STEP 2에서 수집한 컨텍스트 파일을 모든 문서가 공유 → 발주처명·이해관계자·일정 등이 자동으로 모든 문서에 동일하게 박힘
- 새로 도출된 정보(예: 새 이해관계자)는 다음 문서 생성 전 컨텍스트 파일에 병합

---

## STEP 5 — 일관성 점검

전체 생성 완료 후 다음을 검증:

### 5-1. 용어 통일
- 모든 신규 문서에서 사용된 주요 명사 추출
- 동의어·이형 검출 (예: `사용자` vs `이용자`, `시스템` vs `서비스`)
- 불일치 시 보고

### 5-2. 크로스 레퍼런스
- 문서간 참조 링크 (`[XXX](../YY-zz/file.md)`) 유효성 확인
- 깨진 링크 보고

### 5-3. 6관점 점검
methodology.md의 해당 마일스톤 6관점 체크리스트 적용:

| 관점 | 점검 항목 (예시 — 01-requirements) |
|------|----------------------------------|
| PMO | 범위·일정·예산 영향이 모든 문서에 반영 |
| PM | 이해관계자 요구가 추적 가능하게 기록 |
| 아키텍처 | NFR이 아키텍처 결정에 영향 줄 수 있도록 명확 |
| DBA | 데이터 관련 요구가 식별됨 |
| 하네스 | CI/CD·환경 관련 NFR 명시 |
| 보안 | 보안 요건·인증·개인정보 처리 명시 |

### 5-4. 도메인 체크리스트 합산 (v2.2 / v2.3.1 플래그 반영)

STEP 4-5에서 산출물별로 받은 검토자 subagent 결과를 합산:
- 플래그 `off`: 본 섹션 전체 스킵, STEP 6에 비활성 1줄만 출력
- 플래그 `critical-only` (기본): opus 정책 산출물만 합산, sonnet 정책은 "정책 스킵"으로 카운트
- 플래그 `on`: 체크리스트 보유 산출물 모두 합산
- 합산 형식: 호출 K개 / 정책 스킵 S개 / 실패 F개. 호출 K개에 대해 총 항목 X중 반영 Y, 미반영 Z
- 미반영 ≥ 2건 산출물 목록 별도 표시 (STEP 6 보고에서 보강 권장)

---

## STEP 6 — 완료 보고 + CHANGELOG 기록

**CHANGELOG.md 1줄 append (v2.4)**:
- `## [Unreleased]` 아래 `### Added` 섹션에 `- {YYYY-MM-DD} project-milestone: {milestone-id} N건 생성, 일관성 점검 통과/이슈 M건` 1줄 추가
- 모든 문서가 스킵이면 기록 안 함

### 보고 형식

```
✅ {{milestone}} 마일스톤 산출물 생성 완료

📁 생성된 문서 ({{N}}건):
- docs/{{milestone}}/{{file1}}.md ← 신규
- docs/{{milestone}}/{{file2}}.md ← 덮어쓰기
- docs/{{milestone}}/{{file3}}.md ← 스킵 (기존 유지)
...

⚠️ 사람 입력 필요 (TODO 통합 리스트):
- {{file1}}.md: 이해관계자 연락처
- {{file2}}.md: 발주처 SLA 수치
- {{file3}}.md: 인수기준 문구
...

🔍 일관성 점검:
- 용어 통일: ✅ 통과 / ⚠️ 불일치 N건
- 크로스 레퍼런스: ✅ 통과 / ❌ 깨진 링크 N건
- 6관점 점검: ⚠️ 보안 관점 보강 필요 (X.md, Y.md)

🔎 도메인 검토 합산 (검토자 subagent, v2.2, REVIEWER_SUBAGENT={{flag-value}}):
- 호출 K개 / 정책 스킵 S개 / 실패 F개
- 호출 K개에 대해 총 항목 X중 반영 Y, 미반영 Z
- 미반영 ≥ 2건 산출물: {{file1.md, file2.md}} — ⚠️ 보강 권장
```

---

## 사용 예시

```
사용자: /si-project:project-milestone 01-requirements
Claude:
  → reference/doc-catalog.md에서 01-requirements 필수 문서 5개 추출
  → 기존 문서 점검: requirements.md 있음, 나머지 4개 없음
  → AskUserQuestion: 기존 requirements.md 정책?
  → 사용자: "병합"
  → 4개 신규 생성 + 1개 병합
  → 일관성 점검
  → 완료 보고
```
