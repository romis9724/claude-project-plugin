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
  - Bash(git init*)
  - Bash(git add *)
  - Bash(git commit *)
  - Bash(git branch *)
  - Bash(git symbolic-ref *)
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

AskUserQuestion으로 아래 **5개 질문 (Q1, Q2-A/B, Q3, Q4, Q5-A/B — 총 7회 호출)**을 순차적으로 수집한다. AI 에이전트가 코드를 작성하고 검증하는 데 직접 필요한 정보만 받는다.

발주처·계약·MM·검수·운영 SLA 등 행정 정보는 본 단계에서 받지 않는다 — 첫 산출물 생성 시점에 `/project:doc` 또는 `/project:bundle` 스킬이 `.claude/project-context.json`에 모아 수집·재사용한다.

---

**Q1. 프로젝트 기본 정보**
- 프로젝트 한글명
- 프로젝트 영문명 (소문자-하이픈, 예: `next-gen-bank-core`)
- 프로젝트 목적 (2~3문장)
- 예상 기간 (예: `2026-03 ~ 2027-02`)

**Q2. 시스템 유형 + 컴포넌트별 기술 스택 + 인프라·하네스**

> Q2는 정보량이 많으므로 AskUserQuestion을 2회 분할해 받는다.
> - **Q2-A**: 시스템 유형 + Backend / User Frontend / Admin Frontend / Mobile 4슬롯 스택
> - **Q2-B**: DB 4축 + 인프라·망환경 + CI/CD

---

**Q2-A. 시스템 유형 + 컴포넌트별 스택**

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

SI 프로젝트는 거의 항상 백엔드·사용자 화면·관리자 화면을 별도 컴포넌트로 구분하므로, 각 슬롯을 분리해 수집한다. (각 슬롯이 다른 팀·다른 배포 파이프라인을 가지는 게 일반적)

- **Backend / API 서버**:
  - 사용 여부: `있음` / `없음 (프론트 전용·정적 사이트)`
  - 있을 때 스택 (자유 입력, 예: `Java 17 + Spring Boot 3`, `Node.js + NestJS`, `Python + FastAPI`, `Kotlin + Spring`, `.NET 8`)

- **사용자 Frontend** (대고객·외부 사용자용 화면):
  - 사용 여부: `있음` / `없음 (API only)`
  - 있을 때 스택 (자유 입력, 예: `React + Next.js`, `Vue 3 + Nuxt`, `Angular`, `SvelteKit`)

- **관리자 Frontend** (운영자·내부 관리자용 화면):
  - 사용 형태:
    - `별도 화면 (Admin이 사용자 FE와 완전히 분리)`
    - `사용자 FE에 통합 (역할 기반으로 사용자 화면 안에서 관리자 메뉴 노출)`
    - `없음 (관리는 DB/스크립트로 직접)`
  - 별도 또는 통합인 경우 스택 (자유 입력, 예: `React Admin`, `Ant Design Pro`, `Vue + Element Plus`, `Refine`, `사용자 FE와 동일 스택`)

- **Mobile**:
  - 유형:
    - `없음`
    - `네이티브 (iOS + Android 별도)`
    - `하이브리드 (React Native / Flutter)`
    - `PWA (웹앱)`
  - 있을 때 스택 (자유 입력, 예: `Flutter 3.19`, `React Native + Expo`, `Swift 5 + Kotlin`, `PWA(사용자 FE와 동일)`)

---

**Q2-B. DB + 인프라·망환경 + CI/CD**

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

**Q5. 코드 품질·검증 (하네스 관점)**

> Q5는 정보량이 많으므로 AskUserQuestion을 2회 분할해 받는다.
> - **Q5-A**: 린터·포매터 + 테스트 프레임워크 (모두 컴포넌트별 분리)
> - **Q5-B**: 정적/보안 분석 도구 + 커버리지·Quality Gate
> 사용 중인 컴포넌트(STACK_*_USE)만 슬롯을 활성화한다 — Q2에서 "없음"이라 한 슬롯은 묻지 않는다.

---

**Q5-A. 린터·포매터 + 테스트 프레임워크 (컴포넌트별)**

- **린터·포매터** (해당 컴포넌트별 자유 입력 또는 `없음`):
  - Backend (STACK_BACKEND_USE=있음일 때): 예 `Checkstyle + Spotless`, `Black + Ruff + isort`, `golangci-lint + gofmt`, `dotnet format + StyleCop`
  - 사용자 Frontend (STACK_USER_FE_USE=있음일 때): 예 `ESLint + Prettier`, `Biome`, `Stylelint + Prettier`
  - 관리자 Frontend (STACK_ADMIN_FE_MODE ≠ none·integrated일 때): 예 `사용자 FE와 동일`, `ESLint + Prettier`
  - Mobile (STACK_MOBILE_TYPE ≠ none일 때): 예 `SwiftLint + ktlint`, `Flutter analyze + dart format`, `ESLint(RN)`

- **단위 테스트 프레임워크** (해당 컴포넌트별 자유 입력 또는 `없음`):
  - Backend: 예 `JUnit5 + Mockito`, `PyTest + pytest-mock`, `Jest(NestJS)`, `Go test + testify`, `xUnit`
  - 사용자 Frontend: 예 `Vitest + Testing Library`, `Jest + Testing Library`
  - 관리자 Frontend: 예 `사용자 FE와 동일`
  - Mobile: 예 `XCTest + Quick`, `JUnit + Espresso(Android)`, `flutter_test`, `Jest(RN)`

- **E2E·UI 테스트** (다중 선택, 사용자 FE/Admin FE 공용): `Cypress` / `Playwright` / `Selenium` / `TestCafe` / `Detox(RN)` / `Maestro` / `없음`

- **성능·부하 테스트** (다중): `JMeter` / `k6` / `Gatling` / `Locust` / `nGrinder` / `없음`

---

**Q5-B. 정적/보안 분석 + 커버리지·Quality Gate**

- **SAST (정적 분석)** (다중): `SonarQube` / `CodeQL` / `Semgrep` / `Snyk Code` / `Fortify SCA` / `Checkmarx` / `없음`
- **SCA (의존성 취약점)** (다중): `Snyk Open Source` / `OWASP Dependency-Check` / `Dependabot` / `Renovate` / `Trivy(deps)` / `없음`
- **컨테이너·이미지 스캔** (다중): `Trivy` / `Grype` / `Anchore` / `Clair` / `없음` (INFRA가 컨테이너 기반일 때 권장)
- **DAST (동적 분석)** (다중): `OWASP ZAP` / `Burp Suite` / `Nikto` / `없음`
- **단위 커버리지 목표**: `70%` / `80%` / `90%` / `미정`
- **Quality Gate**:
  - `Pass 필수 — CI 차단 (Critical/High 결함 0건, 커버리지 미달 시 머지 불가)`
  - `리포팅만 — 머지는 허용`
  - `미정`
- **코드 리뷰 정책**: `PR 필수 + 1 reviewer 승인` / `PR 필수 + 2 reviewer 승인` / `PR 권장 (자율)` / `미정`

---

### 수집 변수 (CLAUDE.md·산출물에서 사용)

```
[기본]            KR_NAME, EN_NAME, PURPOSE, PERIOD

[시스템 유형]      SYSTEM_TYPES (배열)

[컴포넌트별 스택]   STACK_BACKEND_USE (있음/없음), STACK_BACKEND (자유텍스트)
                  STACK_USER_FE_USE (있음/없음), STACK_USER_FE
                  STACK_ADMIN_FE_MODE (separate/integrated/none), STACK_ADMIN_FE
                  STACK_MOBILE_TYPE (none/native/hybrid/pwa), STACK_MOBILE

[데이터·인프라]    DB_RDB, DB_NOSQL, DB_SEARCH, DB_ANALYTICS, INFRA, HARNESS

[보안]            AUTH, DATA_SENSITIVITY, COMPLIANCE (배열)

[AI·연동]         LLM, AI_USAGE (배열), EXT_API_LIST, PG, MESSAGING (배열)

[코드 품질·검증]   LINTER_BACKEND, LINTER_USER_FE, LINTER_ADMIN_FE, LINTER_MOBILE
                  TEST_UNIT_BACKEND, TEST_UNIT_USER_FE, TEST_UNIT_ADMIN_FE, TEST_UNIT_MOBILE
                  TEST_E2E (배열), TEST_PERF (배열)
                  STATIC_SAST (배열), STATIC_SCA (배열), STATIC_IMAGE (배열), STATIC_DAST (배열)
                  COVERAGE_TARGET, QUALITY_GATE, CODE_REVIEW_POLICY

[Git·구조]        GIT_FLOW_STYLE (trunk-based | git-flow — 자동 결정)
                  MONOREPO_LAYOUT (사용 컴포넌트 디렉토리 합집합 — 자동 계산)
                  CREATE_INFRA_DIR (INFRA에 클라우드 포함 시 true)
```

> 산출물·CLAUDE.md에서 통합 표기가 필요할 때는 4개 STACK_* 변수를 콤마로 합쳐 `STACK_COMBINED` 형태로 사용할 수 있다. 다만 아키텍처·CI/CD·배포 산출물은 컴포넌트별로 분리해 작성한다.

**누락 처리**: 사용자가 답하지 않거나 "없음"을 선택한 항목은 CLAUDE.md·산출물에서 자연스럽게 생략. 행정 정보(발주처·계약·MM·검수·SLA 등)는 첫 doc/bundle 호출 시점에 `.claude/project-context.json`으로 수집됨을 CLAUDE.md에 명시.

---

## STEP 2 — 디렉토리 구조 생성 (컴포넌트 기반 mono-repo)

OS에 맞게 아래 디렉토리를 생성한다 (Windows: `New-Item -ItemType Directory -Force`, Unix: `mkdir -p`).

> ⚠️ v1의 `src/`·`tests/` 단일 디렉토리는 **만들지 않는다**. 각 컴포넌트(Backend/User FE/Admin FE/Mobile)는 자체 framework 관례로 내부 구조를 가지므로, init은 컴포넌트 루트 폴더 + README 안내까지만 생성한다.

**항상 생성**:
```
.claude/
docs/00-kickoff/
docs/01-requirements/
docs/02-architecture/
docs/03-design/
docs/04-implementation/
docs/05-testing/
docs/06-management/
scripts/                    ← 통합 deploy/build orchestration, DB migration runner 등 컴포넌트 무관 스크립트
```

**컴포넌트별 조건부 생성** (Q2 응답 기반):

| 컴포넌트 | 생성 조건 | 디렉토리 | README 안내 |
|---------|---------|---------|------------|
| Backend | `STACK_BACKEND_USE == 있음` | `backend/` | "Spring init / FastAPI 생성 / NestJS CLI 등으로 내부 구조 초기화" |
| 사용자 Frontend | `STACK_USER_FE_USE == 있음` | `user-fe/` | "`npx create-next-app` / `pnpm create vite` / Angular CLI 등" |
| 관리자 Frontend | `STACK_ADMIN_FE_MODE == separate` | `admin-fe/` | "Admin 프레임워크 또는 사용자 FE와 동일 stack" |
| Mobile | `STACK_MOBILE_TYPE ≠ none` | `mobile/` | "`flutter create` / `npx react-native init` / Xcode·Android Studio 신규 프로젝트" |
| Infra (IaC) | `INFRA`에 클라우드(AWS/Azure/GCP/NCP/KT/G클라우드/하이브리드) 중 하나 이상 포함 | `infra/` | "Terraform/CDK/Pulumi/Helm 등 IaC 도구 후보 안내" |

> `STACK_ADMIN_FE_MODE == integrated`이면 `admin-fe/` 루트 폴더를 만들지 않는다. 사용자 FE의 권한 분기/모듈로 처리되므로, `user-fe/admin/` 같은 하위 경로는 framework CLI 초기화 후 사용자가 직접 생성한다.
> `INFRA`가 `사내 IDC/온프레미스/폐쇄망` 단독이면 `infra/` 생성하지 않는다.

각 컴포넌트 폴더 안 README.md에는 다음을 작성:

1. **사용 stack**: 해당 STACK_* 값 그대로 인용
2. **초기화 가이드**: framework CLI 예시 명령 (예: `npx create-next-app@latest .`)
3. **내부 디렉토리 관례 안내**: 각 framework 표준 — 강요 아닌 참고
4. **빌드·테스트 진입점**: TODO 마커 (사용자가 framework CLI 실행 후 채움)

---

## STEP 2.5 — Git 셋팅·전략 자동 적용 (mono-repo 표준)

SI 시스템 단위 mono-repo로 시작한다. 다음 파일을 자동 생성한다 (사용자 추가 질의 없이).

### 2.5-1. Branch 전략 자동 선택 → `GIT_FLOW_STYLE` 변수 설정

`.claude/project-context.json`이 존재하고 `contract.acceptance`가 "단계별" 또는 "마일스톤"이면 `GIT_FLOW_STYLE = "git-flow"`, 그 외 `"trunk-based"` (default). init 시점에는 보통 컨텍스트가 없으므로 **trunk-based**로 시작하고, 추후 doc/bundle 첫 호출 시 컨텍스트 발견되면 CLAUDE.md를 자동 갱신.

### 2.5-2. `.gitignore` — 컴포넌트별 표준 패턴 합집합

공통:
```
# OS
.DS_Store
Thumbs.db
desktop.ini

# Editor
.vscode/
.idea/
*.swp
*~

# Env / Secret
.env
.env.*
*.secret
*.pem
*.key

# Logs / Coverage
*.log
coverage/
.nyc_output/

# Claude state (산출물 commit 안 함)
.claude/projects/
.claude/sessions/
```

Backend (`STACK_BACKEND` detect):
- Java/Maven: `target/`, `*.class`, `*.jar`, `.mvn/`
- Java/Gradle: `build/`, `.gradle/`, `*.jar`
- Python: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `.pytest_cache/`, `.ruff_cache/`, `*.egg-info/`
- Node.js: `node_modules/`, `dist/`
- Go: `bin/`, `*.test`, `vendor/`
- .NET: `bin/`, `obj/`, `*.user`

User FE / Admin FE (separate):
- `node_modules/`, `dist/`, `.next/`, `.nuxt/`, `.svelte-kit/`, `out/`, `.cache/`, `.parcel-cache/`

Mobile:
- Flutter: `.dart_tool/`, `build/`, `.flutter-plugins`, `.flutter-plugins-dependencies`, `ios/Pods/`, `ios/Flutter/Generated.xcconfig`
- React Native: `node_modules/`, `ios/build/`, `android/build/`, `*.keystore`, `ios/Pods/`
- iOS 네이티브: `Pods/`, `*.xcworkspace/xcuserdata/`, `DerivedData/`, `*.xcuserstate`
- Android 네이티브: `*.iml`, `local.properties`, `.gradle/`, `build/`, `captures/`

→ 사용 컴포넌트(Q2)에 해당하는 섹션만 `.gitignore`에 포함.

### 2.5-3. `.gitattributes`

```
# 줄바꿈 통일
* text=auto eol=lf
*.bat text eol=crlf
*.cmd text eol=crlf

# 바이너리
*.png binary
*.jpg binary
*.jpeg binary
*.gif binary
*.ico binary
*.pdf binary
*.zip binary

# Mobile binaries → Git LFS (Mobile 사용 시 추가)
# *.keystore filter=lfs diff=lfs merge=lfs -text
# *.aab filter=lfs diff=lfs merge=lfs -text
# *.ipa filter=lfs diff=lfs merge=lfs -text
```

Mobile이 있으면 LFS 라인의 `#` 주석을 해제하고 README에 `git lfs install` 안내.

### 2.5-4. `.github/PULL_REQUEST_TEMPLATE.md`

```markdown
## 변경 컴포넌트 (해당 항목 체크)
- [ ] Backend
- [ ] User Frontend
- [ ] Admin Frontend
- [ ] Mobile
- [ ] Infra / CI
- [ ] Docs (docs/, CLAUDE.md)

> 두 개 이상의 컴포넌트가 동시 변경되면 본 PR이 **atomic**하게 묶여야 합니다.
> API 계약 변경(backend+frontend), 데이터 모델 변경(backend+mobile) 등이 해당.

## 요약

## 변경 이유

## 테스트
- [ ] 단위 테스트 추가/갱신 — 커버리지 {COVERAGE_TARGET} 유지
- [ ] E2E 영향 확인 ({TEST_E2E})
- [ ] Quality Gate 통과 확인 ({QUALITY_GATE})

## 보안 체크
- [ ] 외부 입력 검증 (OWASP)
- [ ] 비밀키·토큰 미노출
- [ ] 새 의존성 SCA 통과 ({STATIC_SCA})

## 산출물 영향
- 변경 시 갱신해야 할 docs/ 항목:
```

### 2.5-5. `.github/CODEOWNERS`

```
# 컴포넌트별 코드 소유자 — 팀 구성 후 채움
# 형식: <path>  @user1 @user2 @team-name

# 산출물·메타
/docs/         @TODO
/CLAUDE.md     @TODO
/.claude/      @TODO

# 컴포넌트 (사용 중인 것만 자동 포함)
# /backend/      @TODO
# /user-fe/      @TODO
# /admin-fe/     @TODO
# /mobile/       @TODO
# /infra/        @TODO
# /scripts/      @TODO
```

Q2에서 사용한 컴포넌트만 활성 라인으로(주석 해제) 출력.

### 2.5-6. `CHANGELOG.md` (Keep a Changelog 형식 초안)

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- 프로젝트 초기 구조 (project plugin v2.3 init)

### Changed

### Fixed

### Security
```

### 2.5-7. `scripts/git-setup.md` — 사람이 1회 실행할 원격 정책 가이드

```markdown
# Git 원격 정책 셋업 가이드

본 문서는 init이 자동 적용할 수 없는 원격(GitHub/GitLab) 정책을 사람이 1회 실행하기 위한 체크리스트.

## 1. Remote 등록

\`\`\`bash
git remote add origin <REPO_URL>
git push -u origin main
\`\`\`

## 2. Branch protection (main, develop이 있으면 develop도)

GitHub: Settings → Branches → Add branch protection rule
- Branch name pattern: `main` (그리고 `develop`)
- ✅ Require a pull request before merging
- ✅ Require approvals: {CODE_REVIEW_POLICY 기반 — 1 또는 2}
- ✅ Dismiss stale pull request approvals
- ✅ Require status checks to pass before merging
  - 필수 CI workflow 추가 (예: `ci / lint`, `ci / test`, `ci / sast`)
- ✅ Require conversation resolution
- ✅ Restrict force push
- ✅ Restrict deletions
- (선택) Require linear history

## 3. CODEOWNERS 활성

`.github/CODEOWNERS`의 TODO 부분을 팀원·팀 ID로 교체. branch protection에서 ✅ Require review from Code Owners 체크.

## 4. Required status checks

Q5에서 선택한 도구들의 CI job 이름:
- 린터: {LINTER_*}
- 테스트: {TEST_UNIT_*}, {TEST_E2E}
- SAST: {STATIC_SAST}
- SCA: {STATIC_SCA}
- 이미지: {STATIC_IMAGE}

## 5. Issue / PR 라벨

- type: feat / fix / docs / refactor / test / chore / security / perf / build / ci
- scope: backend / user-fe / admin-fe / mobile / infra / docs / meta
- priority: P0 / P1 / P2

## 6. (Mobile 있을 때) Git LFS

\`\`\`bash
git lfs install
git lfs track "*.keystore" "*.aab" "*.ipa"
git add .gitattributes
git commit -m "chore(meta): enable Git LFS for mobile binaries"
\`\`\`
```

### 2.5-8. `GIT_FLOW_STYLE`을 CLAUDE.md에 반영

CLAUDE.md "Git·협업 규약" 섹션(STEP 4에서 생성)에 `GIT_FLOW_STYLE` 값 박힘.

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
- **아키텍처 스타일**: SYSTEM_TYPE + STACK_* 기반으로 레이어드/이벤트 기반/마이크로서비스 중 제안
- **컴포넌트 표** (명칭·역할·기술): 사용 중인 슬롯별로 컴포넌트 분리 작성
  - Backend (있으면): API 서버·도메인 서비스·배치 등
  - 사용자 Frontend (있으면): SPA·SSR·BFF 등
  - 관리자 Frontend (별도면 별도 컴포넌트, 통합이면 사용자 FE의 권한 분기로 기술)
  - Mobile (있으면): 클라이언트 앱 + 필요 시 PUSH/원격구성 백엔드
  - 최소 4~6개 컴포넌트 (사용 슬롯 수에 따라 조정)
- **Mermaid 컴포넌트 다이어그램**: 위 컴포넌트로 관계 다이어그램 작성 — Backend / User FE / Admin FE / Mobile / DB / 외부 API를 노드로
- **데이터 흐름**: 주요 시나리오 2개 텍스트로 기술 (사용자 FE → Backend → DB 1건, Admin FE → Backend → DB 1건 권장)
- **AI 통합 아키텍처** (LLM ≠ 사용안함이면): LLM 호출 흐름, 프롬프트 관리, 응답 검증 기술
- **기술 결정 근거 표**: 채택 기술, 대안, 선택 이유 — 컴포넌트별로 그룹화
- **품질속성 시나리오**: 성능·가용성·보안 각 1개

### 3-3. `docs/02-architecture/security-definition.md` — 보안 정의서

필수 섹션:
- **STRIDE 위협 모델 표** (위협·대상·대응): SYSTEM_TYPE + AUTH + EXT_API 기반으로 최소 6개 위협 식별
- **인증/인가**: AUTH 방식 상세 구현 방향 (토큰 만료·갱신·저장 방식 포함)
- **데이터 보안**: DATA_SENSITIVITY 기반 분류(공개/내부/기밀)·암호화·마스킹 정책
- **API 보안**: Rate Limiting·입력검증·OWASP Top 10 대응 방침
- **AI 보안** (LLM ≠ 사용안함이면): 프롬프트 인젝션 방어·데이터 유출 방지·모델 남용 방어
- **감사 로그**: 기록 대상 이벤트·보관 기간·접근 통제
- **취약점 관리 SLA**: 심각도별 패치 기한
- **보안 점검 도구 (Q5-B 반영)**:
  - SAST: `{STATIC_SAST}` — CI 통합 위치·실행 시점·실패 처리
  - SCA: `{STATIC_SCA}` — 차단 라이센스·심각도 정책
  - 이미지 스캔: `{STATIC_IMAGE}` — 베이스 이미지 정책·취약점 임계치
  - DAST: `{STATIC_DAST}` — 실행 주기·대상 URL·통과 기준
  - Quality Gate: `{QUALITY_GATE}`

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

필수 섹션 (Q5-A/B 값을 우선 반영):
- **테스트 전략**: 단위(목표 = COVERAGE_TARGET)·통합·E2E·성능·보안(SAST/DAST)·AI 모델 평가 전략
- **커버리지 목표 표**: 레이어별 — 단위 = `{COVERAGE_TARGET}`, 통합 ≥ 60%, E2E 주요 시나리오 100%
- **환경 표**: dev/staging 테스트 환경 구성
- **도구 표** (Q5 응답 기반, 컴포넌트별):
  - Backend 단위: `{TEST_UNIT_BACKEND}` (없으면 STACK_BACKEND에서 표준 도구 추천)
  - 사용자 FE 단위: `{TEST_UNIT_USER_FE}`
  - 관리자 FE 단위: `{TEST_UNIT_ADMIN_FE}`
  - Mobile 단위: `{TEST_UNIT_MOBILE}`
  - E2E·UI: `{TEST_E2E}`
  - 성능·부하: `{TEST_PERF}`
- **수용 기준**: 각 테스트 유형별 통과 기준 — Quality Gate = `{QUALITY_GATE}`에 맞춰 정의
- **AI 모델 평가 기준** (LLM ≠ 사용안함이면): 정확도·응답 일관성·지연시간·할루시네이션율 기준

### 3-6. `docs/04-implementation/product-backlog.md` — 제품 백로그

필수 섹션:
- **에픽 목록 표** (ID·에픽명·설명·우선순위): PURPOSE + SYSTEM_TYPE에서 최소 4~6개 에픽 도출
- **유저 스토리 표** (ID·에픽ID·스토리·수용기준·추정포인트): 각 에픽당 2~3개 스토리
- **우선순위 기준**: MoSCoW 또는 WSJF 방식 명시

### 3-7. `docs/04-implementation/definition-of-done.md` — 완료 기준

필수 섹션 (Q5 값을 명시적으로 인용):
- **코드 기준**:
  - 리뷰: `{CODE_REVIEW_POLICY}` 충족
  - 린터·포매터 무위반: 컴포넌트별 LINTER_* 도구 통과
  - SAST/SCA/이미지/DAST 결과 Quality Gate(`{QUALITY_GATE}`) 통과
- **테스트 기준**:
  - 단위 커버리지: `{COVERAGE_TARGET}` 이상
  - E2E: 사용자 핵심 시나리오 통과 (`{TEST_E2E}`)
  - 성능: SLA 임계 통과 (`{TEST_PERF}` 도구 측정)
- **문서 기준**: 변경 시 업데이트해야 할 문서 목록 (요구사항·아키텍처·API 명세·README 등)
- **배포 기준**: CI 파이프라인 통과(린터·테스트·SAST·SCA·이미지 스캔 모두 그린) + 스테이징 검증
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

> 발주처·계약·MM·검수·SLA 등 행정 정보는 `.claude/project-context.json`에 별도 저장됨 (첫 `/project:doc` 또는 `/project:bundle` 호출 시 자동 수집).

## 디렉토리 구조

\`\`\`
{MONOREPO_LAYOUT 기반 — 사용 컴포넌트만 출력}
backend/        — Backend / API 서버 ({STACK_BACKEND})
user-fe/        — 사용자 Frontend ({STACK_USER_FE})
admin-fe/       — 관리자 Frontend ({STACK_ADMIN_FE})  [separate 모드일 때만]
mobile/         — Mobile 클라이언트 ({STACK_MOBILE})
infra/          — IaC (Terraform/CDK/Helm 등)        [INFRA 클라우드일 때만]
scripts/        — 통합 deploy / build orchestration
docs/           — 프로젝트 문서 (SI 표준 8단계)
.claude/        — Claude Code 설정 (project-context.json 포함)
\`\`\`

각 컴포넌트 내부 구조는 해당 framework 관례를 따른다 (`backend/src/main/java`, `user-fe/src` 등). framework CLI(`spring init`, `create-next-app`, `flutter create`)로 초기화 후 사용.

## 핵심 명령어

\`\`\`bash
# 각 컴포넌트는 자체 빌드·테스트 명령을 가짐. framework CLI 초기화 후 README에 채움.

# 예: Backend
# cd backend && {STACK_BACKEND 기반 명령}

# 예: User FE
# cd user-fe && npm run dev / npm run build / npm test

# 통합 (선택, scripts/에서 orchestration)
# ./scripts/deploy-all.sh
\`\`\`

## 코드 컨벤션

- **언어**: {모든 STACK_*에서 주요 언어 추출 — Backend / User FE / Admin FE / Mobile 각각}
- **커밋**: \`type(scope): 한글 메시지\` (feat/fix/docs/refactor/test/chore/security)
  - scope는 컴포넌트 prefix 권장: `feat(backend): ...`, `fix(user-fe): ...`, `chore(admin-fe): ...`, `feat(mobile): ...`
- **브랜치**: \`feature/{ticket}-{desc}\`, \`fix/{ticket}-{desc}\`
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

1. **문서 우선**: 코드 변경 전 관련 \`docs/\` 문서를 먼저 확인한다
2. **단계 준수**: 요구사항 → 설계 → 구현 → 테스트 순서를 지킨다
3. **보안 체크**: 매 변경마다 인증·인가·입력검증·비밀키 노출 여부를 확인한다
4. **테스트 의무**: 신규 기능은 반드시 테스트 코드를 함께 작성한다
5. **문서 동기화**: 설계 변경 시 관련 \`docs/\` 문서도 업데이트한다

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

## Git·협업 규약 (init 자동 설정)

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
    "CLAUDE_CONTEXT_STACK_BACKEND": "{STACK_BACKEND}",
    "CLAUDE_CONTEXT_STACK_USER_FE": "{STACK_USER_FE}",
    "CLAUDE_CONTEXT_STACK_ADMIN_FE": "{STACK_ADMIN_FE}",
    "CLAUDE_CONTEXT_STACK_MOBILE": "{STACK_MOBILE}",
    "CLAUDE_CONTEXT_QUALITY_GATE": "{QUALITY_GATE}",
    "CLAUDE_CONTEXT_COVERAGE_TARGET": "{COVERAGE_TARGET}"
  }
}
```

STACK_BACKEND / STACK_USER_FE / STACK_ADMIN_FE / STACK_MOBILE 모두에서 매칭되는 도구를 합집합으로 allow에 추가한다:

- Python → `"Bash(python *)"`, `"Bash(pip *)"`, `"Bash(pytest *)"`, `"Bash(uvicorn *)"`, `"Bash(poetry *)"`, `"Bash(uv *)"`
- Node.js / React / Next / Vue / Angular / SvelteKit / NestJS → `"Bash(npm *)"`, `"Bash(npx *)"`, `"Bash(yarn *)"`, `"Bash(pnpm *)"`
- Java/Maven → `"Bash(mvn *)"`, `"Bash(java *)"`
- Java/Gradle / Kotlin → `"Bash(./gradlew *)"`, `"Bash(java *)"`
- Go → `"Bash(go *)"`, `"Bash(gofmt *)"`, `"Bash(golangci-lint *)"`
- .NET → `"Bash(dotnet *)"`
- Flutter → `"Bash(flutter *)"`, `"Bash(dart *)"`
- React Native / Expo → `"Bash(expo *)"`, `"Bash(npx react-native *)"`
- iOS 네이티브 → `"Bash(xcodebuild *)"`, `"Bash(pod *)"`
- Android 네이티브 → `"Bash(./gradlew *)"`, `"Bash(adb *)"`
- Docker → `"Bash(docker *)"`, `"Bash(docker-compose *)"`, `"Bash(docker compose *)"`
- GitHub Actions → `"Bash(gh *)"`
- GitLab CI → `"Bash(glab *)"`
- Jenkins → `"Bash(jenkins-cli *)"`

품질·검증 도구 기반 추가 allow (Q5에서 선택된 도구만):

- 린터·포매터 (Q5-A LINTER_*):
  - ESLint/Prettier → `"Bash(eslint *)"`, `"Bash(prettier *)"`, `"Bash(npx eslint *)"`
  - Biome → `"Bash(biome *)"`, `"Bash(npx biome *)"`
  - Black/Ruff/isort → `"Bash(black *)"`, `"Bash(ruff *)"`, `"Bash(isort *)"`
  - Checkstyle/Spotless → `"Bash(mvn checkstyle:check)"`, `"Bash(./gradlew spotlessApply)"`, `"Bash(./gradlew check)"`
  - golangci-lint → `"Bash(golangci-lint *)"`
  - SwiftLint → `"Bash(swiftlint *)"`
  - ktlint → `"Bash(ktlint *)"`
  - dart format / flutter analyze → `"Bash(dart *)"`, `"Bash(flutter analyze)"`
- 테스트 (Q5-A TEST_UNIT_* / TEST_E2E / TEST_PERF):
  - PyTest → `"Bash(pytest *)"`
  - Jest/Vitest → `"Bash(jest *)"`, `"Bash(vitest *)"`, `"Bash(npx jest *)"`, `"Bash(npx vitest *)"`
  - JUnit/Mockito → 이미 Maven/Gradle allow에 포함
  - Go test → `"Bash(go test *)"`
  - Cypress/Playwright → `"Bash(cypress *)"`, `"Bash(playwright *)"`, `"Bash(npx cypress *)"`, `"Bash(npx playwright *)"`
  - JMeter/k6/Gatling → `"Bash(jmeter *)"`, `"Bash(k6 *)"`, `"Bash(gatling *)"`
  - flutter_test → `"Bash(flutter test *)"`
- 정적·보안 분석 (Q5-B STATIC_*):
  - SonarQube → `"Bash(sonar-scanner *)"`, `"Bash(mvn sonar:sonar *)"`
  - Semgrep → `"Bash(semgrep *)"`
  - Snyk → `"Bash(snyk *)"`
  - OWASP Dependency-Check → `"Bash(dependency-check *)"`
  - Trivy → `"Bash(trivy *)"`
  - Grype → `"Bash(grype *)"`
  - OWASP ZAP → `"Bash(zap-cli *)"`, `"Bash(zap.sh *)"`

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

## 컴포넌트 구성
{MONOREPO_LAYOUT 기반 표 — 사용 컴포넌트만}

| 컴포넌트 | 디렉토리 | 스택 |
|---------|---------|------|
| Backend | backend/ | {STACK_BACKEND} |
| 사용자 Frontend | user-fe/ | {STACK_USER_FE} |
| 관리자 Frontend | admin-fe/ (또는 user-fe/admin) | {STACK_ADMIN_FE} |
| Mobile | mobile/ | {STACK_MOBILE} |
| Infra | infra/ | Terraform/CDK/Helm 등 |

- **DB**: {DB_RDB} / {DB_NOSQL} / {DB_SEARCH} / {DB_ANALYTICS}
- **인프라·망환경**: {INFRA}
- **CI/CD**: {HARNESS}

## 빠른 시작

\`\`\`bash
# 각 컴포넌트는 자체 framework CLI로 초기화 (mono-repo)
# Backend 예시
cd backend && {STACK_BACKEND 기반 초기화·실행 명령}

# User FE 예시
cd user-fe && npx create-next-app@latest .  # 또는 사용 framework CLI

# Mobile 예시
cd mobile && flutter create .                # 또는 npx react-native init
\`\`\`

## Git 협업

- **Repo 모델**: mono-repo
- **Branch 전략**: {GIT_FLOW_STYLE}
- **Commit**: `type(scope): 한글 메시지` — scope ∈ {backend, user-fe, admin-fe, mobile, infra, docs, meta}
- **원격 정책 1회 셋업**: [scripts/git-setup.md](scripts/git-setup.md)

> GitHub 외 SCM(GitLab/Bitbucket) 사용 시 `.github/PULL_REQUEST_TEMPLATE.md`와 `.github/CODEOWNERS`를 각 SCM의 등가 위치로 옮기세요:
> - GitLab: `.gitlab/merge_request_templates/Default.md`, `CODEOWNERS` (루트)
> - Bitbucket: `pull_request_template.md` (루트), CODEOWNERS는 외부 서비스 활용

## 문서
- [프로젝트 정의서](docs/00-kickoff/project-charter.md)
- [요구사항 기술서](docs/01-requirements/requirements.md)
- [SW 아키텍처](docs/02-architecture/software-architecture.md)
- [보안 정의서](docs/02-architecture/security-definition.md)
- [CHANGELOG](CHANGELOG.md)
```

### `.gitignore`, `.gitattributes`, `.github/PULL_REQUEST_TEMPLATE.md`, `.github/CODEOWNERS`, `CHANGELOG.md`, `scripts/git-setup.md`

STEP 2.5에서 정의한 내용대로 생성. `.gitignore`는 사용 컴포넌트(Q2 STACK_*_USE)에 해당하는 섹션만 포함.

---

## STEP 7.5 — git init + 첫 commit (자동)

STEP 0~7의 모든 파일 생성이 끝난 직후 실행:

```bash
git init -b main                              # 또는 git init && git symbolic-ref HEAD refs/heads/main
git add -A
git commit -m "chore(meta): 프로젝트 초기화 (project plugin v2.3)

- 사용 컴포넌트: {MONOREPO_LAYOUT}
- Branch 전략: {GIT_FLOW_STYLE}
- Quality Gate: {QUALITY_GATE}
- 커버리지 목표: {COVERAGE_TARGET}
"
```

GIT_FLOW_STYLE이 `git-flow`면 develop 브랜치도 생성:
```bash
git branch develop
```

> Mobile이 있으면 `git lfs install` 후 LFS track 추가 안내 (실행은 하지 않음 — 시스템에 lfs 미설치 가능성).
> 사용자는 다음으로 `git remote add origin <URL>` + `git push -u origin main` (그리고 git-flow면 develop도) 만 하면 됨.
> `scripts/git-setup.md`의 원격 정책 1회 셋업은 사용자가 GitHub UI에서 수동 적용.

---

## STEP 8 — 완료 보고

### 생성된 파일 트리 출력
`ls -R docs/ .claude/` (또는 `Get-ChildItem -Recurse docs/, .claude/`)로 생성 결과를 출력한다.

### 즉시 작성 권장 문서 (Top 3)
1. `docs/01-requirements/requirements.md` — 기능/비기능 요구사항을 팀과 함께 구체화
2. `docs/02-architecture/software-architecture.md` — 아키텍처 결정 사항 상세화
3. `docs/02-architecture/security-definition.md` — 보안 위협 모델 구체화

### 스프린트 1 준비 체크리스트
- [ ] 각 컴포넌트 framework CLI 초기화 완료 (backend/, user-fe/, admin-fe/, mobile/)
- [ ] 요구사항 기술서 1차 리뷰 완료
- [ ] 아키텍처 기술서 팀 공유·승인
- [ ] 보안 정의서 보안 전문가 검토
- [ ] 제품 백로그 우선순위 확정
- [ ] Definition of Done 팀 합의
- [ ] `.claude/settings.json` 권한 조정 완료
- [x] git 초기화 + 첫 커밋 (init이 자동 처리)
- [ ] `git remote add origin <URL>` + 최초 push
- [ ] `scripts/git-setup.md` 따라 branch protection / CODEOWNERS / Required checks 적용
- [ ] CI 파이프라인 초기 설정 (린터·테스트·SAST·SCA·이미지 스캔)
- [ ] (Mobile 있으면) `git lfs install` + LFS track 활성화
