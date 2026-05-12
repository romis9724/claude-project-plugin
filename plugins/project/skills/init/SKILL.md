---
name: init
description: >
  AI 에이전트 기반 신규 프로젝트 초기 환경 일괄 설정.
  PMO·PM·아키텍처·DBA·하네스·보안 6개 관점 통합.
  생성물: CLAUDE.md, .claude/settings.json, 핵심 문서 7개(프로젝트 정보로 충실히), 산출물 템플릿 15개 복사.
  추가 산출물은 /project:doc 또는 /project:bundle로 필요 시점에 생성.
  트리거: "프로젝트 초기화", "프로젝트 환경 설정", "init project", "새 프로젝트 셋업"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Bash(mkdir -p *)
  - Bash(New-Item *)
  - Bash(cp *)
  - Bash(Copy-Item *)
  - Bash(ls *)
  - Bash(Get-ChildItem *)
  - TodoWrite
  - AskUserQuestion
---

# /project:init — AI Agent 프로젝트 초기화

SI 프로젝트 산출물 표준 방법론 기반으로 신규 프로젝트의 초기 환경을 자동 설정합니다. 초기화 이후 추가 산출물은 `/project:doc`(개별) 또는 `/project:bundle`(마일스톤 번들)로 생성합니다.

---

## STEP 0 — 사전 확인

현재 디렉토리에 `CLAUDE.md`가 이미 존재하는지 확인한다.
- 존재하면: "이미 초기화된 프로젝트입니다. 덮어쓸까요?" AskUserQuestion으로 확인 후 진행
- 없으면: 바로 STEP 1 진행

---

## STEP 1 — 프로젝트 정보 수집

AskUserQuestion으로 아래 **4개 질문**을 순차적으로 수집한다. AI 에이전트가 코드를 작성하는 데 직접 필요한 정보만 받는다.

발주처·계약·MM·검수·운영 SLA 등 행정 정보는 본 단계에서 받지 않는다 — 첫 산출물 생성 시점에 `/project:doc` 또는 `/project:bundle` 스킬이 `.claude/project-context.json`에 모아 수집·재사용한다.

---

**Q1. 프로젝트 기본 정보**
- 프로젝트 한글명
- 프로젝트 영문명 (소문자-하이픈, 예: `next-gen-bank-core`)
- 프로젝트 목적 (2~3문장)
- 예상 기간 (예: `2026-03 ~ 2027-02`)

**Q2. 기술·인프라·하네스**
- **시스템 유형** (다중 선택):
  - `차세대 코어 시스템(레거시 마이그레이션)`
  - `대고객 웹 서비스(B2C)`
  - `사내 업무 시스템(B2E)`
  - `B2B 포털·API`
  - `모바일 앱`
  - `데이터 플랫폼/DW/BI`
  - `AI/ML 시스템`
  - `IoT/임베디드`
  - `메시징·스트리밍 처리`
  - `행정·민원·전자정부 시스템`
  - `결제·금융 트랜잭션`
- **언어 + 프레임워크** (자유 입력, 다중 가능. 예: `Java 17 + Spring Boot 3, React + Next.js, Kotlin + Android`)
- **DB** (4축 다중 선택):
  - RDB: `Oracle` / `PostgreSQL` / `MySQL` / `MS SQL Server` / `Tibero` / `DB2` / `없음`
  - NoSQL: `MongoDB` / `Cassandra` / `Redis` / `DynamoDB` / `없음`
  - 검색: `Elasticsearch` / `OpenSearch` / `Solr` / `없음`
  - 분석: `BigQuery` / `Snowflake` / `Hadoop` / `Spark` / `없음`
- **인프라·망환경** (코드 작성에 직접 영향 — 폐쇄망이면 패키지 미러·외부 API 차단·CDN 불가 코드 필요):
  - 퍼블릭: `AWS` / `Azure` / `GCP` / `NCP(네이버)` / `KT 클라우드` / `G클라우드(공공)`
  - 내부: `사내 IDC(온프레미스)` / `프라이빗 클라우드(VMware/OpenStack)`
  - 혼합·격리: `하이브리드(온프레+클라우드)` / `망분리(내부+DMZ+외부)` / `폐쇄망(인터넷 차단)` / `이중망(개발/운영 분리)`
- **CI/CD 도구**: `Jenkins` / `GitHub Actions` / `GitLab CI` / `Argo CD` / `Tekton` / `AWS CodePipeline` / `없음`

**Q3. 보안·인증·규제**
- **인증 방식**: `자체 ID/PW` / `SSO(SAML)` / `OAuth2/OIDC` / `JWT` / `공인인증서/PKI` / `간편인증(카카오/PASS)` / `다중요소(MFA)` / `없음`
- **데이터 민감도**: `일반` / `개인정보 포함` / `금융 정보` / `의료 정보` / `정부 기밀`
- **적용 규제** (다중 선택, 해당 없으면 `해당 없음`):
  - `개인정보보호법` / `정보통신망법` / `전자금융감독규정` / `신용정보법`
  - `의료법(전자의무기록)` / `공공기관 개인정보보호 지침`
  - `GDPR` / `CCPA` / `ISO 27001` / `ISO 27701`
  - `ISMS-P` / `K-ISMS` / `PCI-DSS`
  - `해당 없음`

**Q4. AI·외부 연동**
- **LLM 사용**: `Claude` / `GPT-4` / `Gemini` / `Llama(자체 호스팅)` / `폐쇄망 자체 LLM` / `사용 안 함`
- **AI 적용 영역** (LLM 사용 시 다중): `코드 생성/리뷰` / `문서 자동 생성` / `사용자 챗봇` / `데이터 분석/예측` / `RAG/검색` / `의사결정 지원`
- **외부 API 목록** (자유 입력 또는 `없음`. 예: `Slack API, 카카오톡 비즈, 토스 PG`)
- **결제 PG**: `국내 PG (KG이니시스/토스/카카오페이/네이버페이 등)` / `해외 PG (Stripe/PayPal)` / `없음`
- **메시징·알림** (다중): `SMS` / `카카오 알림톡` / `이메일` / `Push` / `없음`

---

### 수집 변수 (CLAUDE.md·산출물에서 사용)

```
[기본]            KR_NAME, EN_NAME, PURPOSE, PERIOD

[기술]            SYSTEM_TYPES (배열), STACK, DB_RDB, DB_NOSQL, DB_SEARCH,
                  DB_ANALYTICS, INFRA, HARNESS

[보안]            AUTH, DATA_SENSITIVITY, COMPLIANCE (배열)

[AI·연동]         LLM, AI_USAGE (배열), EXT_API_LIST, PG, MESSAGING (배열)
```

**누락 처리**: 사용자가 답하지 않거나 "없음"을 선택한 항목은 CLAUDE.md·산출물에서 자연스럽게 생략. 행정 정보(발주처·계약·MM·검수·SLA 등)는 첫 doc/bundle 호출 시점에 `.claude/project-context.json`으로 수집됨을 CLAUDE.md에 명시.

---

## STEP 2 — 디렉토리 구조 생성

OS에 맞게 아래 디렉토리를 생성한다 (Windows: `New-Item -ItemType Directory -Force`, Unix: `mkdir -p`):

```
.claude/
docs/00-kickoff/
docs/01-requirements/
docs/02-architecture/
docs/03-design/
docs/04-implementation/
docs/05-testing/
docs/06-management/
src/
tests/
scripts/
```

---

## STEP 3 — 핵심 문서 6개 생성 (프로젝트 정보로 충실히 작성)

**생성 규칙**:
- YAML 프론트매터 포함 (`title`, `project`, `version: 0.1`, `status: 초안`, `date`, `owner`)
- 모든 알려진 항목은 STEP 1 답변으로 채움
- 불확실한 항목만 `> ⚠️ TODO: [설명]` 표기
- 구조 데이터는 마크다운 표 사용
- 다이어그램은 Mermaid 코드블록으로 초안 작성

### 3-1. `docs/01-requirements/requirements.md` — 요구사항 기술서

필수 섹션 및 채움 지침:
- **기능 요구사항 표** (ID·요구사항·우선순위·출처): PURPOSE + SYSTEM_TYPE에서 최소 5~8개 도출
- **비기능 요구사항 표**: 성능(응답시간·TPS)·가용성·확장성 + AI 관련(LLM 지연허용·할루시네이션 대응·프롬프트 인젝션 방어) 항목 반드시 포함
- **제약사항**: 기술(STACK 기반)·법규(SECURITY_REQ 기반)·환경(INFRA 기반) 항목 채움
- **수용 기준**: 주요 FR마다 검증 가능한 조건 작성

### 3-2. `docs/02-architecture/software-architecture.md` — SW 아키텍처 기술서

필수 섹션:
- **아키텍처 스타일**: SYSTEM_TYPE + STACK 기반으로 레이어드/이벤트 기반/마이크로서비스 중 제안
- **컴포넌트 표** (명칭·역할·기술): STACK에서 최소 4~6개 컴포넌트 도출
- **Mermaid 컴포넌트 다이어그램**: 위 컴포넌트로 관계 다이어그램 작성
- **데이터 흐름**: 주요 시나리오 2개 텍스트로 기술
- **AI 통합 아키텍처** (LLM ≠ 사용안함이면): LLM 호출 흐름, 프롬프트 관리, 응답 검증 기술
- **기술 결정 근거 표**: 채택 기술, 대안, 선택 이유
- **품질속성 시나리오**: 성능·가용성·보안 각 1개

### 3-3. `docs/02-architecture/security-definition.md` — 보안 정의서

필수 섹션:
- **STRIDE 위협 모델 표** (위협·대상·대응): SYSTEM_TYPE + AUTH + EXT_API 기반으로 최소 6개 위협 식별
- **인증/인가**: AUTH 방식 상세 구현 방향 (토큰 만료·갱신·저장 방식 포함)
- **데이터 보안**: SECURITY_REQ 기반 분류(공개/내부/기밀)·암호화·마스킹 정책
- **API 보안**: Rate Limiting·입력검증·OWASP Top 10 대응 방침
- **AI 보안** (LLM ≠ 사용안함이면): 프롬프트 인젝션 방어·데이터 유출 방지·모델 남용 방어
- **감사 로그**: 기록 대상 이벤트·보관 기간·접근 통제
- **취약점 관리 SLA**: 심각도별 패치 기한

### 3-4. `docs/03-design/db-design.md` — DB 설계서 (DB ≠ 없음인 경우)

필수 섹션:
- **DB 스택 표**: DBMS·버전·ORM·마이그레이션 툴
- **Mermaid ERD**: PURPOSE + SYSTEM_TYPE에서 핵심 엔티티 최소 3~5개 도출, 관계 포함
- **테이블 정의 표**: 각 테이블별 컬럼명·타입·제약·설명 (nullable 여부 포함)
- **인덱스 전략**: 자주 조회되는 컬럼 식별, 복합 인덱스 필요성 기술
- **마이그레이션 규칙**: 툴(Flyway/Liquibase/Alembic 등)·명명 규칙·롤백 절차
- **민감 정보 암호화**: SECURITY_REQ 기반 암호화 대상 컬럼 식별

DB가 없음이면 이 파일 생성 건너뜀.

### 3-5. `docs/05-testing/test-plan.md` — 테스트 계획서

필수 섹션:
- **테스트 전략**: 단위(≥80%)·통합(≥60%)·E2E·성능·보안(OWASP)·AI 모델 평가 전략
- **커버리지 목표 표**: 레이어별 목표 수치
- **환경 표**: dev/staging 테스트 환경 구성
- **도구 표**: STACK 기반 테스트 프레임워크·모킹·커버리지 도구 선정
- **수용 기준**: 각 테스트 유형별 통과 기준
- **AI 모델 평가 기준** (LLM ≠ 사용안함이면): 정확도·응답 일관성·지연시간·할루시네이션율 기준

### 3-6. `docs/04-implementation/product-backlog.md` — 제품 백로그

필수 섹션:
- **에픽 목록 표** (ID·에픽명·설명·우선순위): PURPOSE + SYSTEM_TYPE에서 최소 4~6개 에픽 도출
- **유저 스토리 표** (ID·에픽ID·스토리·수용기준·추정포인트): 각 에픽당 2~3개 스토리
- **우선순위 기준**: MoSCoW 또는 WSJF 방식 명시

### 3-7. `docs/04-implementation/definition-of-done.md` — 완료 기준

필수 섹션:
- **코드 기준**: 리뷰어 조건·린터·포매터·보안 스캔 통과 조건
- **테스트 기준**: 커버리지 목표치·통합 테스트 통과
- **문서 기준**: 변경 시 업데이트해야 할 문서 목록
- **배포 기준**: CI 파이프라인 통과 + 스테이징 검증
- **AI 관련 기준** (LLM ≠ 사용안함이면): 프롬프트 변경 시 회귀 테스트 통과

---

## STEP 4 — CLAUDE.md 생성 (루트 디렉토리)

아래 구조를 STEP 1 답변으로 채워 생성한다:

```markdown
# {KR_NAME} ({EN_NAME})

> {PURPOSE}

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 시스템 유형 | {SYSTEM_TYPES} |
| 기술 스택 | {STACK} |
| RDB | {DB_RDB} |
| NoSQL | {DB_NOSQL} |
| 검색 | {DB_SEARCH} |
| 분석 | {DB_ANALYTICS} |
| 인프라·망환경 | {INFRA} |
| CI/CD | {HARNESS} |
| 기간 | {PERIOD} |
| 인증 | {AUTH} |
| 데이터 민감도 | {DATA_SENSITIVITY} |
| 적용 규제 | {COMPLIANCE} |

> 발주처·계약·MM·검수·SLA 등 행정 정보는 `.claude/project-context.json`에 별도 저장됨 (첫 `/project:doc` 또는 `/project:bundle` 호출 시 자동 수집).

## 디렉토리 구조

\`\`\`
src/        — 소스 코드
tests/      — 테스트 코드
docs/       — 프로젝트 문서 (SI 표준 8단계)
scripts/    — 빌드·배포 스크립트
.claude/    — Claude Code 설정
\`\`\`

## 핵심 명령어

\`\`\`bash
# 개발 서버 실행
# TODO: {STACK} 기반으로 작성

# 테스트 실행
# TODO

# 빌드
# TODO
\`\`\`

## 코드 컨벤션

- **언어**: {STACK에서 주요 언어 추출}
- **커밋**: \`type(scope): 한글 메시지\` (feat/fix/docs/refactor/test/chore/security)
- **브랜치**: \`feature/{ticket}-{desc}\`, \`fix/{ticket}-{desc}\`
- **린터/포매터**: TODO

## AI 에이전트 작업 원칙

1. **문서 우선**: 코드 변경 전 관련 \`docs/\` 문서를 먼저 확인한다
2. **단계 준수**: 요구사항 → 설계 → 구현 → 테스트 순서를 지킨다
3. **보안 체크**: 매 변경마다 인증·인가·입력검증·비밀키 노출 여부를 확인한다
4. **테스트 의무**: 신규 기능은 반드시 테스트 코드를 함께 작성한다
5. **문서 동기화**: 설계 변경 시 관련 \`docs/\` 문서도 업데이트한다

## LLM 관련 지침 (해당 시)

{LLM이 사용안함이면 이 섹션 제거}
- **모델**: {LLM}
- **프롬프트 위치**: \`src/prompts/\`
- **사용자 입력 처리**: sanitize 후 LLM 전달 (프롬프트 인젝션 방지)
- **응답 처리**: 스키마 검증 후 사용, 신뢰하지 않음
- **프롬프트 변경 시**: 회귀 테스트 실행 필수

## 보안 규칙

- **인증**: {AUTH}
- **비밀키**: 코드 하드코딩 금지 → 환경변수 또는 Secret Manager
- **데이터 민감도**: {DATA_SENSITIVITY} → 그에 맞는 암호화·마스킹·접근 통제 적용
- **적용 규제**: {COMPLIANCE} → 규제별 요구사항 준수 (예: 개인정보보호법이면 수집·이용·보관 명시)
- **외부 입력**: 항상 검증·이스케이프 후 처리
- **외부 API**: {EXT_API_LIST}
- **결제**: {PG}
- **메시징**: {MESSAGING}

## 절대 금지

- `main`/`master` 브랜치 직접 push
- `.env` 파일 git commit
- 테스트 없는 기능 PR
- 비밀키·패스워드 코드 내 하드코딩
- TODO 주석만 남기고 구현 없는 PR 병합

## 주요 문서 링크

| 단계 | 문서 | 경로 |
|------|------|------|
| 착수 | 프로젝트 정의서 | [project-charter.md](docs/00-kickoff/project-charter.md) |
| 착수 | 위험 분석서 | [risk-register.md](docs/00-kickoff/risk-register.md) |
| 요구분석 | 요구사항 기술서 | [requirements.md](docs/01-requirements/requirements.md) |
| 아키텍처 | SW 아키텍처 | [software-architecture.md](docs/02-architecture/software-architecture.md) |
| 아키텍처 | 보안 정의서 | [security-definition.md](docs/02-architecture/security-definition.md) |
| 설계 | DB 설계서 | [db-design.md](docs/03-design/db-design.md) |
| 설계 | API 설계서 | [api-design.md](docs/03-design/api-design.md) |
| 구현 | 제품 백로그 | [product-backlog.md](docs/04-implementation/product-backlog.md) |
| 테스트 | 테스트 계획서 | [test-plan.md](docs/05-testing/test-plan.md) |

## 산출물 생성

발주처 요구·내부 마일스톤 시점에 다음 스킬로 산출물을 생성한다.

- **개별 문서 생성**: `/project:doc <문서명>`
  - 예: `/project:doc 인터페이스요구사항정의서`
  - 한글 정식 명칭 또는 영문 파일명(`-` 없는 형태) 모두 사용 가능

- **마일스톤 번들 생성**: `/project:bundle <마일스톤>`
  - 예: `/project:bundle 01-requirements`
  - 해당 마일스톤 필수 문서 일괄 생성 + 문서간 일관성 점검

- **사용 가능 문서 카탈로그**: 플러그인의 `reference/doc-catalog.md` (총 106개)

## AI 에이전트 자동 추천

사용자 발화 패턴에 따라 적절한 스킬을 우선 추천한다:

| 사용자 발화 예시 | 추천 스킬 |
|----------------|----------|
| "X 문서 만들어줘", "X 정의서 작성" | `/project:doc X` |
| "요구분석 산출물 정리", "X 단계 납품물" | `/project:bundle X` |
| "전체 다시 만들어줘", "처음부터" | `/project:init` |
```

---

## STEP 5 — .claude/settings.json 생성

기본 설정으로 생성 후, HARNESS·STACK 기반으로 추가 권한을 포함한다:

```json
{
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(ls *)",
      "Bash(cat *)",
      "Bash(find *)",
      "Bash(grep *)",
      "Bash(mkdir *)",
      "Bash(cp *)",
      "Bash(mv *)",
      "Bash(echo *)"
    ],
    "deny": [
      "Bash(rm -rf /*)",
      "Bash(sudo rm *)",
      "Bash(chmod 777 *)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[$(date '+%H:%M:%S')] CMD: $CLAUDE_TOOL_INPUT_COMMAND\""
          }
        ]
      }
    ]
  },
  "env": {
    "CLAUDE_CONTEXT_PROJECT": "{EN_NAME}",
    "CLAUDE_CONTEXT_STACK": "{STACK}"
  }
}
```

STACK 기반 추가 allow:
- Python → `"Bash(python *)"`, `"Bash(pip *)"`, `"Bash(pytest *)"`, `"Bash(uvicorn *)"`, `"Bash(poetry *)"`, `"Bash(uv *)"`
- Node.js → `"Bash(npm *)"`, `"Bash(npx *)"`, `"Bash(yarn *)"`, `"Bash(pnpm *)"`
- Java/Maven → `"Bash(mvn *)"`, `"Bash(java *)"`
- Java/Gradle → `"Bash(./gradlew *)"`, `"Bash(java *)"`
- Go → `"Bash(go *)"`, `"Bash(gofmt *)"`, `"Bash(golangci-lint *)"`
- Docker → `"Bash(docker *)"`, `"Bash(docker-compose *)"`, `"Bash(docker compose *)"`
- GitHub Actions → `"Bash(gh *)"`
- GitLab CI → `"Bash(glab *)"`
- Jenkins → `"Bash(jenkins-cli *)"`

---

## STEP 6 — 핵심 템플릿 15개 복사 및 배치

플러그인 내부 템플릿 디렉토리에서 해당 `docs/` 경로로 복사한다.

**플러그인 템플릿 루트**: `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/project/templates/`

| 템플릿 파일 | 원본 경로 | 복사 대상 |
|------------|----------|----------|
| project-charter.md | `templates/00-kickoff/` | `docs/00-kickoff/` |
| project-plan.md | `templates/00-kickoff/` | `docs/00-kickoff/` |
| risk-register.md | `templates/00-kickoff/` | `docs/00-kickoff/` |
| system-vision.md | `templates/01-requirements/` | `docs/01-requirements/` |
| actors.md | `templates/01-requirements/` | `docs/01-requirements/` |
| process-model.md | `templates/01-requirements/` | `docs/01-requirements/` |
| system-architecture.md | `templates/02-architecture/` | `docs/02-architecture/` |
| design-patterns.md | `templates/02-architecture/` | `docs/02-architecture/` |
| component-spec.md | `templates/03-design/` | `docs/03-design/` |
| api-design.md | `templates/03-design/` | `docs/03-design/` |
| service-definition.md | `templates/03-design/` | `docs/03-design/` |
| sprint-template.md | `templates/04-implementation/` | `docs/04-implementation/` |
| sprint-retrospective.md | `templates/04-implementation/` | `docs/04-implementation/` |
| quality-plan.md | `templates/06-management/` | `docs/06-management/` |
| config-management.md | `templates/06-management/` | `docs/06-management/` |

복사 후 각 파일의 `{{PROJECT_NAME_KR}}`·`{{PROJECT_NAME_EN}}`·`{{DATE}}`·`{{OWNER}}`를 실제 값으로 Edit 대체한다.

> **추가 90+ 템플릿**: 플러그인의 `templates/` 하위에 모두 있음. 매 프로젝트에 복사하지 않고, 필요 시점에 `/project:doc <문서명>` 또는 `/project:bundle <마일스톤>` 호출로 생성한다.

---

## STEP 7 — 추가 기본 파일 생성

### `docs/06-management/change-request-template.md`
```markdown
---
title: 변경 요청 기술서 (템플릿)
usage: "매 변경 요청 시 파일명을 CR-{ID}-{title}.md 로 복사하여 사용"
---

# CR-{{ID}}: {{제목}}

| 항목 | 내용 |
|------|------|
| 요청자 | |
| 요청일 | |
| 우선순위 | High / Medium / Low |
| 영향 범위 | |

## 변경 내용
-

## 변경 근거
-

## 영향 분석
- **코드**: 
- **문서**: 
- **테스트**: 
- **일정**: 

## 승인
- [ ] PM 승인
- [ ] 아키텍트 확인
- [ ] DBA 확인 (DB 변경 시)
- [ ] 보안 검토 (보안 관련 변경 시)
```

### `docs/05-testing/test-design.md`
비어있는 테스트 설계서 템플릿 생성:
```markdown
---
title: 테스트 설계서
project: {KR_NAME}
version: 0.1
status: 초안
---

# 테스트 설계서: {KR_NAME}

## 테스트 케이스 목록

| ID | 대상 | 시나리오 | 입력 | 기대 결과 | 실행 결과 |
|----|------|---------|------|----------|----------|
| TC-001 | | | | | |

## 경계값·예외 시나리오
- TODO

## 보안 테스트 케이스 (OWASP Top 10 기반)
| OWASP | 항목 | 테스트 방법 | 통과 기준 |
|-------|------|-----------|---------|
| A01 | Broken Access Control | 권한 없는 리소스 접근 시도 | 403 반환 |
| A02 | Cryptographic Failures | 민감정보 암호화 검증 | 평문 저장 없음 |
| A03 | Injection | SQL/Command Injection 시도 | 차단 및 로그 기록 |
```

### `README.md` (프로젝트 루트)
```markdown
# {KR_NAME}

> {PURPOSE}

## 기술 스택
- **언어/프레임워크**: {STACK}
- **DB**: {DB}
- **인프라**: {INFRA}

## 빠른 시작

\`\`\`bash
# 환경 설정
cp .env.example .env
# TODO: 개발 서버 실행 명령어

# 테스트
# TODO
\`\`\`

## 문서
- [프로젝트 정의서](docs/00-kickoff/project-charter.md)
- [요구사항 기술서](docs/01-requirements/requirements.md)
- [SW 아키텍처](docs/02-architecture/software-architecture.md)
- [보안 정의서](docs/02-architecture/security-definition.md)
```

### `.gitignore`
STACK에 맞는 .gitignore 생성 (Python이면 `__pycache__/`, `.venv/`, `*.pyc`, Node면 `node_modules/`, `.next/` 등). 공통 항목 반드시 포함:
```
.env
.env.local
.env.*.local
*.secret
.DS_Store
```

---

## STEP 8 — 완료 보고

### 생성된 파일 트리 출력
`ls -R docs/ .claude/` (또는 `Get-ChildItem -Recurse docs/, .claude/`)로 생성 결과를 출력한다.

### 즉시 작성 권장 문서 (Top 3)
1. `docs/01-requirements/requirements.md` — 기능/비기능 요구사항을 팀과 함께 구체화
2. `docs/02-architecture/software-architecture.md` — 아키텍처 결정 사항 상세화
3. `docs/02-architecture/security-definition.md` — 보안 위협 모델 구체화

### 스프린트 1 준비 체크리스트
- [ ] 요구사항 기술서 1차 리뷰 완료
- [ ] 아키텍처 기술서 팀 공유·승인
- [ ] 보안 정의서 보안 전문가 검토
- [ ] 제품 백로그 우선순위 확정
- [ ] Definition of Done 팀 합의
- [ ] `.claude/settings.json` 권한 조정 완료
- [ ] git 초기화 + 첫 커밋 (`chore: 프로젝트 초기화`)
- [ ] CI 파이프라인 초기 설정
