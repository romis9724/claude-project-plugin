# project-setup Q1~Q5 질문 옵션 풀

본 파일은 `/si-project:project-setup` STEP 1에서 `AskUserQuestion`을 호출할 때 사용하는 옵션 목록이다. SKILL.md가 본 파일을 Read해 각 Q의 옵션으로 사용.

총 7회 AskUserQuestion: Q1 / Q2-A / Q2-B / Q3 / Q4 / Q5-A / Q5-B.

---

## Q1. 프로젝트 기본 정보

자유 입력 4개:
- 프로젝트 한글명
- 프로젝트 영문명 (소문자-하이픈, 예: `next-gen-bank-core`)
- 프로젝트 목적 (2~3문장)
- 예상 기간 (예: `2026-03 ~ 2027-02`)

---

## Q2-A. 시스템 유형 + 컴포넌트별 스택

**시스템 유형** (다중 선택):
- 차세대 코어 시스템(레거시 마이그레이션)
- 대고객 웹 서비스(B2C)
- 사내 업무 시스템(B2E)
- B2B 포털·API
- 모바일 앱
- 데이터 플랫폼/DW/BI
- AI/ML 시스템
- IoT/임베디드
- 메시징·스트리밍 처리
- 행정·민원·전자정부 시스템
- 결제·금융 트랜잭션

> SI 프로젝트는 거의 항상 백엔드·사용자 화면·관리자 화면을 별도 컴포넌트로 구분하므로, 각 슬롯을 분리해 수집한다.

**Backend / API 서버**:
- 사용 여부: `있음` / `없음 (프론트 전용·정적 사이트)`
- 있을 때 스택 (자유 입력 예: `Java 17 + Spring Boot 3`, `Node.js + NestJS`, `Python + FastAPI`, `Kotlin + Spring`, `.NET 8`)

**사용자 Frontend** (대고객·외부 사용자용 화면):
- 사용 여부: `있음` / `없음 (API only)`
- 있을 때 스택 (자유 입력 예: `React + Next.js`, `Vue 3 + Nuxt`, `Angular`, `SvelteKit`)

**관리자 Frontend** (운영자·내부 관리자용 화면):
- 사용 형태:
  - `별도 화면 (Admin이 사용자 FE와 완전히 분리)`
  - `사용자 FE에 통합 (역할 기반으로 사용자 화면 안에서 관리자 메뉴 노출)`
  - `없음 (관리는 DB/스크립트로 직접)`
- 별도 또는 통합인 경우 스택 (자유 입력 예: `React Admin`, `Ant Design Pro`, `Vue + Element Plus`, `Refine`, `사용자 FE와 동일 스택`)

**Mobile**:
- 유형:
  - `없음`
  - `네이티브 (iOS + Android 별도)`
  - `하이브리드 (React Native / Flutter)`
  - `PWA (웹앱)`
- 있을 때 스택 (자유 입력 예: `Flutter 3.19`, `React Native + Expo`, `Swift 5 + Kotlin`, `PWA(사용자 FE와 동일)`)

---

## Q2-B. DB + 인프라·망환경 + CI/CD

**DB** (4축 다중 선택):
- RDB: `Oracle` / `PostgreSQL` / `MySQL` / `MS SQL Server` / `Tibero` / `DB2` / `없음`
- NoSQL: `MongoDB` / `Cassandra` / `Redis` / `DynamoDB` / `없음`
- 검색: `Elasticsearch` / `OpenSearch` / `Solr` / `없음`
- 분석: `BigQuery` / `Snowflake` / `Hadoop` / `Spark` / `없음`

**인프라·망환경** (코드 작성에 직접 영향 — 폐쇄망이면 패키지 미러·외부 API 차단·CDN 불가 코드 필요):
- 퍼블릭: `AWS` / `Azure` / `GCP` / `NCP(네이버)` / `KT 클라우드` / `G클라우드(공공)`
- 내부: `사내 IDC(온프레미스)` / `프라이빗 클라우드(VMware/OpenStack)`
- 혼합·격리: `하이브리드(온프레+클라우드)` / `망분리(내부+DMZ+외부)` / `폐쇄망(인터넷 차단)` / `이중망(개발/운영 분리)`

**CI/CD 도구**: `Jenkins` / `GitHub Actions` / `GitLab CI` / `Argo CD` / `Tekton` / `AWS CodePipeline` / `없음`

---

## Q3. 보안·인증·규제

**인증 방식**: `자체 ID/PW` / `SSO(SAML)` / `OAuth2/OIDC` / `JWT` / `공인인증서/PKI` / `간편인증(카카오/PASS)` / `다중요소(MFA)` / `없음`

**데이터 민감도**: `일반` / `개인정보 포함` / `금융 정보` / `의료 정보` / `정부 기밀`

**적용 규제** (다중 선택, 해당 없으면 `해당 없음`):
- `개인정보보호법` / `정보통신망법` / `전자금융감독규정` / `신용정보법`
- `의료법(전자의무기록)` / `공공기관 개인정보보호 지침`
- `GDPR` / `CCPA` / `ISO 27001` / `ISO 27701`
- `ISMS-P` / `K-ISMS` / `PCI-DSS`
- `해당 없음`

---

## Q4. AI·외부 연동

**LLM 사용**: `Claude` / `GPT-4` / `Gemini` / `Llama(자체 호스팅)` / `폐쇄망 자체 LLM` / `사용 안 함`

**AI 적용 영역** (LLM 사용 시 다중): `코드 생성/리뷰` / `문서 자동 생성` / `사용자 챗봇` / `데이터 분석/예측` / `RAG/검색` / `의사결정 지원`

**외부 API 목록** (자유 입력 또는 `없음`. 예: `Slack API, 카카오톡 비즈, 토스 PG`)

**결제 PG**: `국내 PG (KG이니시스/토스/카카오페이/네이버페이 등)` / `해외 PG (Stripe/PayPal)` / `없음`

**메시징·알림** (다중): `SMS` / `카카오 알림톡` / `이메일` / `Push` / `없음`

---

## Q5-A. 린터·포매터 + 테스트 프레임워크

사용 중인 컴포넌트(`STACK_*_USE`)만 슬롯을 활성화 — Q2에서 "없음"이라 한 슬롯은 묻지 않는다.

**린터·포매터** (해당 컴포넌트별 자유 입력 또는 `없음`):
- Backend (STACK_BACKEND_USE=있음일 때): 예 `Checkstyle + Spotless`, `Black + Ruff + isort`, `golangci-lint + gofmt`, `dotnet format + StyleCop`
- 사용자 Frontend (STACK_USER_FE_USE=있음일 때): 예 `ESLint + Prettier`, `Biome`, `Stylelint + Prettier`
- 관리자 Frontend (STACK_ADMIN_FE_MODE ≠ none·integrated일 때): 예 `사용자 FE와 동일`, `ESLint + Prettier`
- Mobile (STACK_MOBILE_TYPE ≠ none일 때): 예 `SwiftLint + ktlint`, `Flutter analyze + dart format`, `ESLint(RN)`

**단위 테스트 프레임워크** (해당 컴포넌트별 자유 입력 또는 `없음`):
- Backend: 예 `JUnit5 + Mockito`, `PyTest + pytest-mock`, `Jest(NestJS)`, `Go test + testify`, `xUnit`
- 사용자 Frontend: 예 `Vitest + Testing Library`, `Jest + Testing Library`
- 관리자 Frontend: 예 `사용자 FE와 동일`
- Mobile: 예 `XCTest + Quick`, `JUnit + Espresso(Android)`, `flutter_test`, `Jest(RN)`

**E2E·UI 테스트** (다중 선택, 사용자 FE/Admin FE 공용): `Cypress` / `Playwright` / `Selenium` / `TestCafe` / `Detox(RN)` / `Maestro` / `없음`

**성능·부하 테스트** (다중): `JMeter` / `k6` / `Gatling` / `Locust` / `nGrinder` / `없음`

---

## Q5-B. 정적/보안 분석 + 커버리지·Quality Gate

**SAST (정적 분석)** (다중): `SonarQube` / `CodeQL` / `Semgrep` / `Snyk Code` / `Fortify SCA` / `Checkmarx` / `없음`

**SCA (의존성 취약점)** (다중): `Snyk Open Source` / `OWASP Dependency-Check` / `Dependabot` / `Renovate` / `Trivy(deps)` / `없음`

**컨테이너·이미지 스캔** (다중): `Trivy` / `Grype` / `Anchore` / `Clair` / `없음` (INFRA가 컨테이너 기반일 때 권장)

**DAST (동적 분석)** (다중): `OWASP ZAP` / `Burp Suite` / `Nikto` / `없음`

**단위 커버리지 목표**: `70%` / `80%` / `90%` / `미정`

**Quality Gate**:
- `Pass 필수 — CI 차단 (Critical/High 결함 0건, 커버리지 미달 시 머지 불가)`
- `리포팅만 — 머지는 허용`
- `미정`

**코드 리뷰 정책**: `PR 필수 + 1 reviewer 승인` / `PR 필수 + 2 reviewer 승인` / `PR 권장 (자율)` / `미정`

---

## 수집 변수 (CLAUDE.md·산출물에서 사용)

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
