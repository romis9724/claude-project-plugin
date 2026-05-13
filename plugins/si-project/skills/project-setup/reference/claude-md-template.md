# {KR_NAME} ({EN_NAME})

> {PURPOSE}

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 시스템 유형 | {SYSTEM_TYPES} |
| Backend / API | {STACK_BACKEND_USE: "있음/없음"} — {STACK_BACKEND} |
| 사용자 Frontend | {STACK_USER_FE_USE: "있음/없음"} — {STACK_USER_FE} |
| 관리자 Frontend | {STACK_ADMIN_FE_MODE: "별도/통합/없음"} — {STACK_ADMIN_FE} |
| Mobile | {STACK_MOBILE_TYPE: "없음/네이티브/하이브리드/PWA"} — {STACK_MOBILE} |
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

> 발주처·계약·MM·검수·SLA 등 행정 정보는 `.claude/project-context.json`에 별도 저장됨 (첫 `/si-project:project-document` 또는 `/si-project:project-milestone` 호출 시 자동 수집).

## 미설정 항목 (구현 시작 전 채워야 함)

> 본 섹션은 사용자가 채우는 체크리스트다. 항목이 비어 있으면 해당 영역 구현 금지(절대 금지 항목 참조). 채우면 줄을 지운다.
> `/si-project:project-check` 호출 시 본 섹션 + `.env` + 요구사항 산출물 모호도가 함께 점수화된다.

- [ ] 외부 API 키 / 시크릿 (`.env` 빈 슬롯 채우기 — `.env.example`에서 복사)
- [ ] 발주처·계약·검수 기준 (`.claude/project-context.json`)
- [ ] 요구사항 모호 항목 (`docs/01-requirements/*.md`의 `명확도: 모호/미정` 행)
- [ ] (필요 시) Git 원격·branch protection 적용

## 디렉토리 구조

```
{MONOREPO_LAYOUT 기반 — 사용 컴포넌트만 출력}
backend/        — Backend / API 서버 ({STACK_BACKEND})
user-fe/        — 사용자 Frontend ({STACK_USER_FE})
admin-fe/       — 관리자 Frontend ({STACK_ADMIN_FE})  [separate 모드일 때만]
mobile/         — Mobile 클라이언트 ({STACK_MOBILE})
infra/          — IaC (Terraform/CDK/Helm 등)        [INFRA 클라우드일 때만]
scripts/        — 통합 deploy / build orchestration
docs/           — 프로젝트 문서 (SI 표준 8단계)
.claude/        — Claude Code 설정 (project-context.json 포함)
```

각 컴포넌트 내부 구조는 해당 framework 관례를 따른다 (`backend/src/main/java`, `user-fe/src` 등). framework CLI(`spring init`, `create-next-app`, `flutter create`)로 초기화 후 사용.

## 핵심 명령어

```bash
# 각 컴포넌트는 자체 빌드·테스트 명령을 가짐. framework CLI 초기화 후 README에 채움.

# 예: Backend
# cd backend && {STACK_BACKEND 기반 명령}

# 예: User FE
# cd user-fe && npm run dev / npm run build / npm test

# 통합 (선택, scripts/에서 orchestration)
# ./scripts/deploy-all.sh
```

## 코드 컨벤션

- **언어**: {모든 STACK_*에서 주요 언어 추출 — Backend / User FE / Admin FE / Mobile 각각}
- **커밋**: `type(scope): 한글 메시지` (feat/fix/docs/refactor/test/chore/security)
  - scope는 컴포넌트 prefix 권장: `feat(backend): ...`, `fix(user-fe): ...`, `chore(admin-fe): ...`, `feat(mobile): ...`
- **브랜치**: `feature/{ticket}-{desc}`, `fix/{ticket}-{desc}`
- **린터·포매터** (컴포넌트별, AI 코드 작성 시 반드시 준수):
  - Backend: {LINTER_BACKEND}
  - 사용자 Frontend: {LINTER_USER_FE}
  - 관리자 Frontend: {LINTER_ADMIN_FE} (`사용자 FE와 동일`인 경우 동일 규칙 적용)
  - Mobile: {LINTER_MOBILE}

## 테스트 정책 (AI는 신규 기능 시 반드시 테스트 동반 생성)

- **단위 테스트 프레임워크** (컴포넌트별):
  - Backend: {TEST_UNIT_BACKEND}
  - 사용자 Frontend: {TEST_UNIT_USER_FE}
  - 관리자 Frontend: {TEST_UNIT_ADMIN_FE}
  - Mobile: {TEST_UNIT_MOBILE}
- **E2E·UI**: {TEST_E2E}
- **성능·부하**: {TEST_PERF}
- **단위 커버리지 목표**: {COVERAGE_TARGET}
- **Quality Gate**: {QUALITY_GATE}
- **코드 리뷰**: {CODE_REVIEW_POLICY}

## 정적·보안 분석 (CI 파이프라인 통합 필수)

- **SAST**: {STATIC_SAST}
- **SCA(의존성)**: {STATIC_SCA}
- **컨테이너·이미지 스캔**: {STATIC_IMAGE}
- **DAST**: {STATIC_DAST}
- AI 작업 지침:
  - 외부 입력 처리 시 위 도구의 룰 위반(SQLi/XSS/Path traversal 등) 회피 패턴 사용
  - 의존성 추가 전 SCA 도구의 차단 라이브러리 여부 확인
  - 비밀키·토큰을 코드 또는 설정에 박지 않음 (SAST가 즉시 잡음)

## AI 에이전트 작업 원칙

0. **명확성 우선 (Clarity Gate)**: 요구사항·외부 정보가 모호하면 **구현 시작 금지**. 추측 금지. 다음 중 하나로 처리:
   - 사용자에게 `AskUserQuestion`으로 확인
   - `## 미설정 항목`에 한 줄 추가하고 해당 영역만 건너뜀 (다른 부분은 진행 가능)
   - 산출물의 명확도가 `모호`/`미정`인 항목은 코드로 옮기지 않음
   - 구현 직전 자기 점검: `/si-project:project-check` 호출 권장
1. **문서 우선**: 코드 변경 전 관련 `docs/` 문서를 먼저 확인한다
2. **단계 준수**: 요구사항 → 설계 → 구현 → 테스트 순서를 지킨다
3. **보안 체크**: 매 변경마다 인증·인가·입력검증·비밀키 노출 여부를 확인한다
4. **테스트 의무**: 신규 기능은 반드시 테스트 코드를 함께 작성한다
5. **문서 동기화**: 설계 변경 시 관련 `docs/` 문서도 업데이트한다

## LLM 관련 지침 (해당 시)

{LLM이 사용안함이면 이 섹션 제거}
- **모델**: {LLM}
- **프롬프트 위치**: `{backend|user-fe|… 중 LLM 호출 컴포넌트}/prompts/` (보통 backend, BFF 사용 시 user-fe)
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

## Git·협업 규약 (project-setup 자동 설정)

- **Repo 모델**: mono-repo — 컴포넌트별 루트 폴더 (`{MONOREPO_LAYOUT}`)
- **Branch 전략**: `{GIT_FLOW_STYLE}`
  - `trunk-based`: `main` + `feature/{ticket}-{desc}` (짧은 lifespan, 자주 머지)
  - `git-flow`: `main` ← `release/{ver}` ← `develop` ← `feature/*` (단계별 검수 발주처용, `hotfix/{ver}` 직접 main)
- **Commit message**: `type(scope): 한글 메시지`
  - type: `feat` / `fix` / `docs` / `refactor` / `test` / `chore` / `security` / `perf` / `build` / `ci`
  - scope: `backend` / `user-fe` / `admin-fe` / `mobile` / `infra` / `docs` / `meta` (여러 컴포넌트)
  - 예: `feat(backend): 주문 API 추가`, `fix(user-fe): 결제 화면 검증 오류 수정`
- **PR 정책**: main/develop **직접 push 금지** (branch protection). PR 통해서만 머지.
  - 리뷰어: `{CODE_REVIEW_POLICY}`
  - CI 통과 필수: `{QUALITY_GATE}` (린터·테스트·SAST·SCA·이미지 스캔 그린)
- **Atomic PR**: API 계약·데이터 모델 변경은 영향 컴포넌트 모두 같은 PR에 묶기 — `.github/PULL_REQUEST_TEMPLATE.md`의 "변경 컴포넌트" 체크박스로 가드
- **자주 머지·작은 단위**: long-lived feature branch 회피, 매일 또는 격일 머지
- **원격 정책 1회 셋업**: `scripts/git-setup.md` 안내대로 GitHub branch protection / CODEOWNERS / Required checks 1회 적용

## 절대 금지

- `main`/`master`/`develop` 브랜치 직접 push
- `.env`·`*.secret`·`*.keystore` 파일 git commit
- 테스트 없는 기능 PR
- 비밀키·패스워드 코드 내 하드코딩
- TODO 주석만 남기고 구현 없는 PR 병합
- 여러 컴포넌트 변경을 별도 PR로 쪼개 atomicity 깨뜨리기 (API 계약·데이터 모델 변경 시)
- **모호·미정 요구사항을 추측으로 구현** (산출물에 `명확도: 모호/미정` 또는 `{{TBD}}` 표기된 항목)
- **`.env` 빈 슬롯·미설정 외부 API 키 상태에서 해당 통합 코드 작성** (`## 미설정 항목` 체크 후 진행)

## 주요 문서 링크

> **참고**: 아래 문서는 일반 markdown 링크다. AI에게 항상 함께 컨텍스트로 주입하려면 `@`-prefix import 사용 (Claude Code memory hierarchy):
> - 예: `@docs/02-architecture/software-architecture.md` 한 줄로 작성하면 본 CLAUDE.md 로드 시 자동 포함.
> - 단, 모든 문서를 import하면 CLAUDE.md가 비대해진다. **요구사항·아키텍처·보안 3건 정도만** import 고려, 나머지는 일반 링크로.

<!-- AI 컨텍스트 자동 주입 (필요 시 주석 해제 — 신중하게)
@docs/01-requirements/requirements.md
@docs/02-architecture/software-architecture.md
@docs/02-architecture/security-definition.md
-->

| 단계 | 문서 | 경로 |
|------|------|------|
| 착수 | 프로젝트 정의서 | [project-charter.md](docs/00-kickoff/project-charter.md) |
| 착수 | 위험 분석서 | [risk-register.md](docs/00-kickoff/risk-register.md) |
| 요구분석 | 요구사항 기술서 | [requirements.md](docs/01-requirements/requirements.md) |
| 아키텍처 | SW 아키텍처 | [software-architecture.md](docs/02-architecture/software-architecture.md) |
| 아키텍처 | 보안 정의서 | [security-definition.md](docs/02-architecture/security-definition.md) |
| 아키텍처 | ADR (결정 기록) | [adr/](docs/02-architecture/adr/) |
| 설계 | DB 설계서 | [db-design.md](docs/03-design/db-design.md) |
| 설계 | API 설계서 | [api-design.md](docs/03-design/api-design.md) |
| 구현 | 제품 백로그 | [product-backlog.md](docs/04-implementation/product-backlog.md) |
| 테스트 | 테스트 계획서 | [test-plan.md](docs/05-testing/test-plan.md) |

## 산출물 생성

발주처 요구·내부 마일스톤 시점에 다음 스킬로 산출물을 생성한다.

- **개별 문서 생성**: `/si-project:project-document <문서명>`
  - 예: `/si-project:project-document 인터페이스요구사항정의서`
  - 한글 정식 명칭 또는 영문 파일명(`-` 없는 형태) 모두 사용 가능

- **마일스톤 번들 생성**: `/si-project:project-milestone <마일스톤>`
  - 예: `/si-project:project-milestone 01-requirements`
  - 해당 마일스톤 필수 문서 일괄 생성 + 문서간 일관성 점검

- **프로젝트 현황 요약**: `/si-project:project-summary`
  - 30줄 이하 5섹션 요약 (프로젝트·마일스톤·최근 변경·완료 산출물·권장 액션)

- **아키텍처 결정 기록**: `/si-project:project-adr <결정 제목>`
  - MADR 형식. `docs/02-architecture/adr/NNNN-{slug}.md` 자동 저장

- **구현 준비도 점검 (명확성 게이트)**: `/si-project:project-check`
  - 읽기 전용, 30줄 이하. 요구사항 산출물의 모호도(`TBD`/`?`/`모호`/`미정`) + 빈 `.env` 슬롯 + `## 미설정 항목` 카운트 → "준비도 점수" 출력
  - 구현 본격 시작 전, 또는 마일스톤 진입 직전에 호출 권장

- **원격 작업 큐 (Inbox)**: `/si-project:project-inbox`
  - 읽기 전용, 30줄 이하. `.claude/inbox.md` + GitHub Issues(label:si-project-inbox) + project-check 4축 자동 도출을 통합 표시
  - 새 세션 진입·작업 완료 시점에 호출 권장
  - 입력 채널: 어디서든 git push로 `.claude/inbox.md`에 추가 또는 GitHub Issues에 라벨 부여

- **사용 가능 문서 카탈로그**: 플러그인의 `reference/doc-catalog.md` (총 106개)

## 원격 작업 큐 (Remote Inbox)

AI agent 작업이 끝났을 때 사용자가 콘솔 앞에 없어도 **다음 할 일을 어디서든 큐잉**할 수 있게 한다.

### 우선 활용 — Anthropic 공식 도구
- **claude.ai/code (웹)** — 데스크탑·모바일 브라우저에서 Claude Code 세션 직접 실행
- **Claude Code 데스크탑 앱** (Mac/Windows)
- **VS Code / JetBrains IDE 확장**
- **PushNotification** — 작업 완료 시 모바일 푸시 (Claude Code 내장)
- **CronCreate / Scheduled Tasks** — 주기적 자동 실행 (Claude Code 내장)

### 입력 채널 — 2가지 (`/si-project:project-inbox`로 통합 표시)

**(1) 로컬 `.claude/inbox.md`** — 폐쇄망·오프라인 친화
- 어디서든 git push로 추가 (모바일 GitHub 앱에서 직접 편집 가능)
- 형식: `## P0|P1|P2` 헤더 아래 `- [ ] 항목`
- 처리 완료: `- [ ]` → `- [x]` 마킹 또는 삭제

**(2) GitHub Issues** (label: `si-project-inbox`) — 팀·모바일 친화
- 모바일 GitHub 앱·웹에서 새 issue 생성, 라벨 부여
- 처리 완료: `gh issue close <N>` 또는 모바일에서 close
- gh CLI 미설치 / 폐쇄망 환경에서는 자동 스킵

### 출력 — `/si-project:project-inbox`
- 새 세션 진입·작업 완료 시점에 호출
- 로컬 + GitHub Issues + `/si-project:project-check` 자동 도출 항목을 30줄 이하로 통합 표시
- 다음 추천 액션 1줄 제안

### 절대 금지
- 본 plugin이 외부 ITS API를 자동 호출해 issue 자동 생성·자동 close 안 함 (사용자 명시 액션만)
- `.claude/inbox.md`의 `[x]` 항목은 plugin이 자동 삭제 안 함 (사용자 큐레이션)

## 구현 단계 subagent 권장 패턴

산출물 작성이 끝나고 코드 구현 단계에 진입했을 때, Claude Code의 Agent tool로 다음 패턴을 활용 가능. **자율 사용 — 강제 아님.**

### ✅ 권장 (cross-reference 위험 낮음)
- **단위 테스트 생성**: 구현 코드 입력 → 테스트 작성 subagent
- **코드 리뷰어**: 보안 / 성능 / 스타일 리뷰 (병렬 3개 호출 가능)
- **코드베이스 탐색**: 의존성·호출처 추적 (격리 컨텍스트, 결과 요약만 부모로)
- **문서/주석 생성**: docstring, README 보강

### ⚠️ 신중 (인터페이스 합의 선행)
- **다중 모듈 동시 구현**: 공통 DTO·인터페이스·에러 코드를 부모가 먼저 확정해 사전 패키지로 전달한 뒤에만 병렬

### ❌ 위험 (subagent 분리 금지)
- DB 스키마 변경·마이그레이션
- 인증/인가 로직
- 트랜잭션 경계 코드
- 공통 미들웨어·인터셉터

### 모델 선택 가이드
- haiku/sonnet: 단순 CRUD·보일러플레이트·테스트·문서
- opus: 핵심 알고리즘·보안 로직·아키텍처 결정 코드

> 산출물 작성·검토 단계의 subagent는 `/si-project:project-document` / `/si-project:project-milestone`이 자동 호출 (검토자 패턴, v2.2). 본 가이드는 그 후속인 코드 구현 단계용.

## AI 에이전트 자동 추천

사용자 발화 패턴에 따라 적절한 스킬을 우선 추천한다:

| 사용자 발화 예시 | 추천 스킬 |
|----------------|----------|
| "X 문서 만들어줘", "X 정의서 작성" | `/si-project:project-document X` |
| "요구분석 산출물 정리", "X 단계 납품물" | `/si-project:project-milestone X` |
| "현황", "오늘 할 일", "어디까지 했지" | `/si-project:project-summary` |
| "X 결정 ADR로 남겨줘", "아키텍처 결정 기록" | `/si-project:project-adr X` |
| "구현 시작해도 되나", "준비도 점검", "명확도 체크", "모호한 부분 확인" | `/si-project:project-check` |
| "다음 뭐 할까", "받은 일감", "원격 지시 확인", "inbox" | `/si-project:project-inbox` |
| "전체 다시 만들어줘", "처음부터" | `/si-project:project-setup` |
