---
name: doc
description: >
  SI 표준 형식의 개별 산출물 문서를 현재 프로젝트 상태에서 생성한다.
  발주처가 특정 문서를 요청했을 때 즉시 1건 생성.
  입력: 한글 문서명(예: "인터페이스요구사항정의서") 또는 영문 파일명(예: "interface-requirements").
  트리거: "산출물 만들어줘", "doc generate", "인터페이스정의서 작성", "테스트설계서 만들어줘"
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

# /project:doc — 개별 산출물 문서 생성

발주처 요청 또는 내부 필요 시점에 SI 표준 형식의 문서 한 개를 현재 프로젝트 상태에서 생성한다.

---

## STEP 0 — 사전 확인 + lessons 참고

현재 디렉토리가 `/project:init`으로 초기화된 프로젝트인지 확인:
- `CLAUDE.md` 존재 여부 → 없으면 사용자에게 "init이 먼저 필요합니다" 안내 후 중단
- `docs/` 디렉토리 존재 여부

**lessons.md 참고** (v2.4): `.claude/lessons.md` 존재 시 Read해 컨텍스트로 반영. 자동 기록 없음 — 사용자가 명시 요청 시에만 본 스킬이 추가 가능.

---

## STEP 1 — 입력 파싱 (Grep 기반 lazy 매칭)

사용자가 제공한 문서명을 식별:
- 한글 입력(예: `인터페이스요구사항정의서`) → 카탈로그에서 영문 파일명 매칭
- 영문 입력(예: `interface-requirements`) → 직접 매칭
- 누락 시 AskUserQuestion으로 사용자에게 어떤 문서를 생성할지 확인

**카탈로그 접근 (v2.4 lazy 패턴)**: 전체 Read 금지. Grep으로 1줄만 추출.

```
# 한글 입력 매칭 예
Grep pattern="인터페이스요구사항" path="<plugin>/reference/doc-catalog.md" output_mode="content" -C=0
# 결과: | interface-requirements.md | 인터페이스요구사항정의서 | 권장 | ... |
# → 영문 파일명 + 마일스톤(헤더 추정) 확보

# 영문 입력 매칭 예
Grep pattern="^\| interface-requirements" path="<plugin>/reference/doc-catalog.md" output_mode="content"
```

매칭 후 해당 마일스톤을 결정. 마일스톤은 doc-catalog.md의 섹션 헤더(예: `## 01-requirements`)로 추적. Grep 1번 더로 헤더 위치 확인.

---

## STEP 2 — 템플릿·가이드 로드 (lazy, 마커 기반)

식별된 문서에 대해:
1. **템플릿 로드**: `{플러그인_경로}/templates/{milestone}/{file}.md` 읽기
   - 플러그인 경로 예: `~/.claude/plugins/marketplaces/team-tools/plugins/project/`
2. **작성 가이드 로드 (v2.4 lazy 패턴)**: `{플러그인_경로}/reference/methodology.md` **전체 읽지 말 것**.
   - `Grep pattern="<!-- DOC: {file_stem} -->" -n` → 시작 라인 N
   - `Grep pattern="<!-- /DOC: {file_stem} -->" -n` → 종료 라인 M
   - `Read offset=N, limit=(M-N+1)` → 해당 섹션만 (~20-40줄) 로드
   - 마커가 없는 문서(methodology.md 3.X에 없는 것)는 일반 작성 원칙(섹션 4 이후)만 참조
3. **카탈로그 매핑은 이미 STEP 1에서 확보됨** — 중복 Read 금지

---

## STEP 3 — 컨텍스트 파일 확인·수집 (`.claude/project-context.json`)

**행정 정보(발주처·계약·MM·검수·SLA·이해관계자 등)는 init에서 받지 않는다.** 산출물 생성 시점에 본 스킬·bundle 스킬이 모아 컨텍스트 파일에 저장·재사용한다.

### 3-1. 컨텍스트 파일 존재 여부 확인

`.claude/project-context.json` 파일 존재 여부:

**경우 A — 없음**: 본 호출이 첫 행정 정보 수집 시점
- 생성할 문서가 어떤 행정 정보를 필요로 하는지 카탈로그·methodology에서 식별
- 누락 정보를 AskUserQuestion으로 묶어 수집 (한 번에 4문항 이하)
- 수집 결과로 `.claude/project-context.json` 생성

**경우 B — 있음**: 기존 컨텍스트 활용
- 파일 로드 → 산출물에 필요한 정보 자동 채움
- 현재 문서가 필요로 하는데 컨텍스트에 누락된(`{{TBD}}` 또는 키 없음) 항목만 추가 질문
- 수집한 정보를 컨텍스트 파일에 병합 저장

### 3-2. 컨텍스트 파일 구조

```json
{
  "_version": "1.0",
  "_updated": "YYYY-MM-DD",

  "customer": {
    "type": "공공기관|금융|제조|통신|유통|의료|교육|기타",
    "name": "발주처명",
    "contact_person": "담당자",
    "contact_email": "이메일"
  },
  "contract": {
    "type": "도급|SI|SM|위탁|자체개발",
    "acceptance": "단계별|일괄|마일스톤|Agile",
    "scope_in": ["..."],
    "scope_out": ["..."]
  },
  "schedule": {
    "kickoff_date": "YYYY-MM-DD",
    "delivery_date": "YYYY-MM-DD",
    "milestones": [
      { "name": "단계명", "date": "YYYY-MM-DD" }
    ]
  },
  "resource": {
    "total_mm": 0,
    "budget": "...",
    "partners": ["..."],
    "customer_involvement": "없음|검토만|상주|PMO상주"
  },
  "stakeholders": [
    { "role": "역할", "name": "이름" }
  ],
  "operations": {
    "owner": "자체|발주처|SI사 SM|별도 SM",
    "runtime": "24/7|평일주간|평일24h",
    "availability": "99.9%|99.99%|99.999%",
    "rto_rpo": "RTO/RPO 명시"
  },
  "methodology": {
    "name": "폭포수|애자일(스크럼)|하이브리드|RUP|SI 자체",
    "standards": ["전자정부프레임워크|사내표준|..."]
  },
  "delivery_format": {
    "format": "Markdown|HWP|DOCX|PDF|혼합",
    "stages": "4단계|6단계|Agile|일괄",
    "user_training": "있음(매뉴얼+오프라인)|있음(매뉴얼만)|없음"
  }
}
```

### 3-3. 산출물 ↔ 컨텍스트 필드 매핑

생성 대상 문서별 필요 컨텍스트 (없으면 수집):

| 산출물 | 필요 컨텍스트 |
|--------|------------|
| project-charter | customer.*, contract.*, schedule.kickoff_date/delivery_date |
| project-plan | customer.name, schedule.*, resource.*, methodology.* |
| project-overview | customer.name, contract.type, resource.budget |
| project-budget | resource.total_mm, resource.budget |
| project-progress-report | resource.*, schedule.milestones |
| change-request | customer.name, contract.scope_in/out |
| project-closure-report | customer.*, schedule.*, resource.* |
| system-transition-plan | customer.name, operations.owner |
| operations-plan | operations.availability, operations.runtime, operations.owner |
| contingency-plan | operations.rto_rpo, operations.availability |
| phase-evaluation | contract.acceptance |
| user-training-plan/report | delivery_format.user_training |

### 3-4. 누락 정보 수집 정책

- 첫 호출 시: 카탈로그상 해당 문서 + 같은 마일스톤의 필수 문서가 공통으로 필요로 하는 정보를 묶어 수집
- 사용자 미답 항목: 컨텍스트 파일에 `{{TBD}}` 저장. 산출물 본문에도 `> ⚠️ TODO: {{설명}}` 표기
- 추후 다른 문서 생성 시 `{{TBD}}` 발견하면 다시 질문 (강제는 아님 — 사용자가 스킵 가능)

---

## STEP 4 — 프로젝트 상태 분석

기술 정보는 현재 프로젝트에서 추출 (컨텍스트 파일 외):

| 소스 | 추출 정보 |
|------|----------|
| `CLAUDE.md` | 프로젝트명·목적·스택·DB·인프라·인증·LLM·민감도·규제 |
| `.claude/project-context.json` | **행정 정보 (발주처·계약·일정·자원·이해관계자·운영·방법론·납품)** |
| `docs/00-kickoff/project-charter.md` | 범위 In/Out |
| `docs/01-requirements/requirements.md` | 기능/비기능 요구사항 |
| `docs/02-architecture/*.md` | 아키텍처 결정·컴포넌트·보안 모델 |
| `docs/03-design/*.md` | DB 설계·API 설계·컴포넌트 명세 |
| 소스 코드 (Glob/Grep) | 실제 구현된 모듈·클래스·API 엔드포인트 |

---

## STEP 5 — 문서 생성

다음 규칙으로 생성:

1. **YAML 프론트매터**: 템플릿 그대로 사용, 변수 치환
   - `{{PROJECT_NAME}}` → CLAUDE.md의 프로젝트명
   - `{{DATE}}` → 오늘 날짜
   - `{{OWNER}}` → CLAUDE.md에서 추출 또는 `TBD`

2. **섹션 채움 우선순위**:
   - **확신 정보**: 프로젝트 상태에서 직접 추출 → 그대로 작성
   - **추론 가능 정보**: 기존 문서·코드에서 합리적으로 도출 → 작성 + `> ⓘ {{source}} 기반 추론` 주석
   - **미입력 정보**: 사람만 알 수 있음 → `> ⚠️ TODO: {{설명}}` 표기

3. **표·다이어그램 처리**:
   - 표 컬럼 구조는 템플릿 유지
   - 행은 프로젝트 상태에서 추출하거나 `TBD` 행 1~2개로 채움
   - Mermaid 다이어그램은 추출된 컴포넌트·엔티티로 초안 생성

4. **6관점 점검**: methodology.md에 명시된 해당 문서의 6관점 체크포인트 적용

---

## STEP 6 — 저장 및 보고

저장 위치: `docs/{milestone}/{file}.md`
- 이미 존재 시 AskUserQuestion으로 사용자 확인 (덮어쓰기/스킵/이름 변경/병합)

생성 후 컨텍스트 파일 업데이트 (STEP 3에서 수집된 정보 + 본 문서 작성 중 알게 된 새 정보).

**CHANGELOG.md 1줄 append (v2.4)**:
- 파일 존재 시 `## [Unreleased]` 아래 `### Added` 섹션에 `- {YYYY-MM-DD} doc: {한글 문서명} 생성 (docs/{milestone}/{file}.md)` 1줄 추가
- 덮어쓰기·병합 케이스는 `### Changed`에 기록
- 스킵 케이스는 기록 안 함

생성 보고 출력:
```
✅ 생성 완료: docs/{milestone}/{file}.md

📊 정보 출처:
- 프로젝트 메타: CLAUDE.md
- 행정 정보: .claude/project-context.json (필드 N개 사용)
- 기능 요구사항: docs/01-requirements/requirements.md (5건 인용)
- 아키텍처 컴포넌트: docs/02-architecture/software-architecture.md (3건 인용)
- 실제 구현: src/api/ 디렉토리 (2건 추출)

📝 컨텍스트 파일 업데이트:
- {{새로 추가된 필드 목록 또는 "변경 없음"}}

⚠️ 사람 입력 필요 (TODO):
- {{TODO 항목 목록 — 보통 3~7개}}
```

---

## 사용 예시

```
사용자: /project:doc 인터페이스요구사항정의서
Claude:
  → reference/doc-catalog.md에서 매칭: interface-requirements.md (01-requirements)
  → templates/01-requirements/interface-requirements.md 로드
  → CLAUDE.md + 기존 docs/ + src/ 분석
  → docs/01-requirements/interface-requirements.md 생성
  → 보고
```

```
사용자: "외부 API 연동 문서 만들어줘"
Claude:
  → 의도 파악: 인터페이스 관련 문서
  → AskUserQuestion: "인터페이스요구사항정의서 / 인터페이스명세서 중 어떤 것?"
  → 사용자 선택 후 위와 동일하게 진행
```
