---
name: project-setup
description: >
  AI 에이전트 기반 신규 SI 프로젝트 초기 환경 일괄 설정.
  PMO·PM·아키텍처·DBA·하네스·보안 6개 관점 통합.
  생성물: CLAUDE.md(≤150줄, ## 미설정 항목 섹션 포함), .claude/settings.json, .pre-commit-config.yaml,
  .github/workflows/ci.yml(GHA일 때), 핵심 문서 7개, ADR-0001, 산출물 템플릿 5~8개(조건부).
  추가 산출물은 /si-project:project-document 또는 /si-project:project-milestone으로 필요 시점에 생성.
  구현 본격 시작 전에는 /si-project:project-check로 명확성 게이트 점검 권장 (v2.1.0+).
  트리거: "SI 프로젝트 셋업", "신규 SI 프로젝트 초기화", "project setup", "SI 프로젝트 환경 구축"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
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

# /si-project:project-setup — AI Agent 기반 SI 프로젝트 초기화

SI 프로젝트 산출물 표준 방법론 기반으로 신규 프로젝트의 초기 환경을 자동 설정합니다. 초기화 이후 추가 산출물은 `/si-project:project-document`(개별) 또는 `/si-project:project-milestone`(마일스톤 번들)로 생성합니다.

> **v2.5.1 슬림화**: 본 SKILL.md는 워크플로·결정 로직만 보유. 인라인 템플릿·옵션 풀은 `reference/` 디렉토리로 분리. 모든 STEP에서 "Read → placeholder 치환 → Write" 패턴 사용.

**reference 디렉토리 구조**:
```
skills/project-setup/reference/
├── questions.md                   ← Q1~Q5 옵션 풀
├── claude-md-template.md          ← CLAUDE.md 골격
├── settings-template.json         ← .claude/settings.json 골격
├── stack-tools.md                 ← 스택→Bash 권한 매핑
├── core-docs-guides.md            ← STEP 3 7개 문서 작성 가이드 (마커 기반)
├── git-files/                     ← .gitignore/.gitattributes/PR/CODEOWNERS/CHANGELOG/git-setup/lessons/pre-commit
├── project-files/                 ← README/test-design/change-request
├── adr/                           ← ADR README + 0001 템플릿
└── ci-workflows/github-actions.yml.template
```

---

## STEP 0 — 사전 확인

`CLAUDE.md` 존재 시 AskUserQuestion으로 덮어쓰기 확인. 없으면 STEP 1 진행.

---

## STEP 1 — 프로젝트 정보 수집

본 STEP은 AskUserQuestion을 **총 7회** 호출 (Q1, Q2-A, Q2-B, Q3, Q4, Q5-A, Q5-B). 각 호출의 옵션·자유 입력 형식은 `reference/questions.md` 참조.

**진행 순서**:
1. `Read reference/questions.md`로 질문 정의 확인
2. 각 Q에 대해 AskUserQuestion 호출 (Q2-A/B와 Q5-A/B는 정보량 분할)
3. 응답을 변수로 보관 (KR_NAME, EN_NAME, PURPOSE, PERIOD, SYSTEM_TYPES, STACK_*, DB_*, INFRA, HARNESS, AUTH, DATA_SENSITIVITY, COMPLIANCE, LLM, AI_USAGE, EXT_API_LIST, PG, MESSAGING, LINTER_*, TEST_UNIT_*, TEST_E2E, TEST_PERF, STATIC_*, COVERAGE_TARGET, QUALITY_GATE, CODE_REVIEW_POLICY)
4. 자동 파생 변수:
   - `MONOREPO_LAYOUT`: 사용 컴포넌트(STACK_*_USE/MODE/TYPE)의 디렉토리 합집합
   - `GIT_FLOW_STYLE`: 기본 `trunk-based`. project-context.json이 사후 등장하면 갱신
   - `CREATE_INFRA_DIR`: INFRA에 클라우드 포함 시 true

발주처·계약·MM·검수·운영 SLA 등 행정 정보는 본 단계에서 받지 않는다 — 첫 `/si-project:project-document` 또는 `/si-project:project-milestone` 호출 시점에 `.claude/project-context.json`에 수집.

**누락 처리**: 사용자가 답하지 않거나 "없음" 선택 항목은 산출물에서 자연스럽게 생략.

---

## STEP 2 — 디렉토리 구조 생성 (컴포넌트 기반 mono-repo)

OS 분기 (Windows: `New-Item -ItemType Directory -Force`, Unix: `mkdir -p`).

**항상 생성**:
```
.claude/
docs/00-kickoff/  docs/01-requirements/  docs/02-architecture/  docs/02-architecture/adr/
docs/03-design/  docs/04-implementation/  docs/05-testing/  docs/06-management/
scripts/
```

**컴포넌트별 조건부**:

| 컴포넌트 | 생성 조건 | 디렉토리 |
|---------|---------|---------|
| Backend | `STACK_BACKEND_USE == 있음` | `backend/` |
| 사용자 Frontend | `STACK_USER_FE_USE == 있음` | `user-fe/` |
| 관리자 Frontend | `STACK_ADMIN_FE_MODE == separate` | `admin-fe/` |
| Mobile | `STACK_MOBILE_TYPE ≠ none` | `mobile/` |
| Infra | `INFRA`에 클라우드 포함 | `infra/` |

각 컴포넌트 폴더 안 `README.md`에 ① 사용 stack ② framework CLI 초기화 예시 ③ 내부 디렉토리 관례 ④ 빌드·테스트 진입점 TODO 작성.

> `STACK_ADMIN_FE_MODE == integrated`이면 `admin-fe/` 생성 안 함. `INFRA`가 온프레미스 단독이면 `infra/` 생성 안 함.

---

## STEP 2.5 — Git 셋팅·정책 파일 생성

다음 파일을 `reference/git-files/`에서 Read → 치환 → Write. 사용자 추가 질의 없음.

| 파일 | 소스 | 동작 |
|------|------|------|
| `.gitignore` | `reference/git-files/gitignore-patterns.md` | 마커 기반 Grep — `common` + 사용 컴포넌트 BLOCK만 추출 합쳐 작성 |
| `.gitattributes` | `reference/git-files/gitattributes` | 그대로 복사. Mobile 있으면 LFS 라인 주석 해제 |
| `.github/PULL_REQUEST_TEMPLATE.md` | `reference/git-files/pr-template.md` | `{COVERAGE_TARGET}`/`{TEST_E2E}`/`{QUALITY_GATE}`/`{STATIC_SCA}` 치환 |
| `.github/CODEOWNERS` | `reference/git-files/codeowners` | 사용 컴포넌트 라인만 주석 해제 |
| `CHANGELOG.md` | `reference/git-files/changelog-init.md` | `{YYYY-MM-DD}`를 오늘 날짜로 치환 |
| `.claude/lessons.md` | `reference/git-files/lessons-init.md` | 그대로 복사 (gitignore 미포함 — 팀 공유) |
| `scripts/git-setup.md` | `reference/git-files/git-setup.md` | `{CODE_REVIEW_POLICY}`/`{LINTER_*}`/`{STATIC_*}`/`{TEST_*}` 치환 |
| `.pre-commit-config.yaml` | `reference/git-files/pre-commit-config.yaml` | gitleaks 항상 포함. Q5-A LINTER_*에 따라 ruff/eslint/spotless 블록 주석 해제 |

**Branch 전략**: project-context.json이 있고 `contract.acceptance ∈ {단계별, 마일스톤}`이면 `GIT_FLOW_STYLE = "git-flow"`, 그 외 `"trunk-based"`. project-setup 시점에는 보통 컨텍스트가 없어 기본 trunk-based.

---

## STEP 3 — 핵심 문서 7개 생성

각 문서는 `reference/core-docs-guides.md`의 마커(`<!-- DOC: name -->` ~ `<!-- /DOC: name -->`)로 감싸진 가이드 섹션을 Grep + Read offset/limit로 lazy load하여 작성.

대상 7건:
- `docs/01-requirements/requirements.md` (마커: `requirements`)
- `docs/02-architecture/software-architecture.md` (마커: `software-architecture`)
- `docs/02-architecture/security-definition.md` (마커: `security-definition`)
- `docs/03-design/db-design.md` (마커: `db-design`. DB가 모두 없음이면 스킵)
- `docs/05-testing/test-plan.md` (마커: `test-plan`)
- `docs/04-implementation/product-backlog.md` (마커: `product-backlog`)
- `docs/04-implementation/definition-of-done.md` (마커: `definition-of-done`)

각 문서 공통 생성 규칙은 `core-docs-guides.md` 헤더 참조. YAML 프론트매터 + 섹션 채움 + 불확실 항목 `> ⚠️ TODO` 표기.

---

## STEP 4 — CLAUDE.md 생성 (루트, 슬림 모드 ≤150줄)

1. `Read reference/claude-md-template.md`
2. STEP 1 응답으로 placeholder 치환:
   - `{KR_NAME}` `{EN_NAME}` `{PURPOSE}` `{PERIOD}`
   - `{SYSTEM_TYPES}` `{STACK_BACKEND_USE}` `{STACK_BACKEND}` `{STACK_USER_FE_USE}` `{STACK_USER_FE}` `{STACK_ADMIN_FE_MODE}` `{STACK_ADMIN_FE}` `{STACK_MOBILE_TYPE}` `{STACK_MOBILE}`
   - `{DB_RDB}` `{DB_NOSQL}` `{DB_SEARCH}` `{DB_ANALYTICS}` `{INFRA}` `{HARNESS}`
   - `{AUTH}` `{DATA_SENSITIVITY}` `{COMPLIANCE}`
   - `{LINTER_*}` `{TEST_UNIT_*}` `{TEST_E2E}` `{TEST_PERF}` `{COVERAGE_TARGET}` `{QUALITY_GATE}` `{CODE_REVIEW_POLICY}`
   - `{STATIC_SAST}` `{STATIC_SCA}` `{STATIC_IMAGE}` `{STATIC_DAST}`
   - `{LLM}` `{EXT_API_LIST}` `{PG}` `{MESSAGING}` `{GIT_FLOW_STYLE}` `{MONOREPO_LAYOUT}`
3. 조건부 섹션 처리:
   - `LLM == 사용안함`이면 "LLM 관련 지침" 섹션 통째로 제거
   - 사용 안 하는 컴포넌트 행은 디렉토리 구조·표에서 제거
4. 루트 디렉토리에 `CLAUDE.md`로 Write
5. 길이 검증: 150줄 초과 시 placeholder 미치환·중복 행 확인 후 재정리

**원칙**: 상세는 `docs/` 위임. CLAUDE.md는 "인덱스 + 핵심 규약"만.

---

## STEP 5 — .claude/settings.json 생성

1. `Read reference/settings-template.json`
2. `{KR_NAME}` `{EN_NAME}` `{STACK_*}` `{QUALITY_GATE}` `{COVERAGE_TARGET}` 치환
3. `Read reference/stack-tools.md`로 매핑 확인
4. STACK_BACKEND / STACK_USER_FE / STACK_ADMIN_FE / STACK_MOBILE / HARNESS / LINTER_* / TEST_* / STATIC_* 응답과 일치하는 Bash 권한들을 `permissions.allow` 배열에 합집합으로 추가
5. `.claude/settings.json`으로 Write

---

## STEP 6 — 핵심 템플릿 조건부 복사 (5~8개)

**플러그인 템플릿 루트**: `~/.claude/plugins/marketplaces/team-tools/plugins/si-project/templates/`

### 6-1. 항상 복사 (5개)

| 템플릿 파일 | 원본 | 복사 대상 |
|------------|------|----------|
| project-charter.md | `templates/00-kickoff/` | `docs/00-kickoff/` |
| project-plan.md | `templates/00-kickoff/` | `docs/00-kickoff/` |
| risk-register.md | `templates/00-kickoff/` | `docs/00-kickoff/` |
| system-vision.md | `templates/01-requirements/` | `docs/01-requirements/` |
| quality-plan.md | `templates/06-management/` | `docs/06-management/` |

### 6-2. 조건부 복사

| 조건 | 추가 템플릿 |
|------|----------|
| `STACK_BACKEND_USE == 있음` OR `STACK_USER_FE_USE == 있음` | `api-design.md`, `component-spec.md` |
| 애자일 (project-context에 명시 시) | `sprint-template.md`, `sprint-retrospective.md` |
| 폭포수 OR 미정 | `config-management.md` |
| UI 있음 (User FE / Admin FE / Mobile) | `actors.md` |
| INFRA에 클라우드 포함 | `system-architecture.md` |

복사 후 `{{PROJECT_NAME_KR}}` `{{PROJECT_NAME_EN}}` `{{DATE}}` `{{OWNER}}` 치환.

**최소·최대**: 5(정적 사이트) ~ 9(애자일+풀스택+클라우드).

### 6-3. ADR 디렉토리 초기화

- `Read reference/adr/readme.md` → `{DATE}` 치환 → `docs/02-architecture/adr/README.md` Write
- `Read reference/adr/0001-stack-selection.md` → 모든 placeholder({DATE}, {OWNER}, {SYSTEM_TYPES}, {STACK_*}, {DB_*}, {INFRA}, {AUTH}, {DATA_SENSITIVITY}, {COMPLIANCE}, {PERIOD}, {HARNESS}) 치환 → `docs/02-architecture/adr/0001-stack-selection.md` Write

> 추가 90+ 템플릿은 매 프로젝트 복사하지 않음. `/si-project:project-document <문서명>` 또는 `/si-project:project-milestone <마일스톤>`로 필요 시점 생성.

---

## STEP 7 — 추가 기본 파일 생성

| 파일 | 소스 | 동작 |
|------|------|------|
| `docs/06-management/change-request-template.md` | `reference/project-files/change-request-template.md` | 그대로 복사 |
| `docs/05-testing/test-design.md` | `reference/project-files/test-design.md` | `{KR_NAME}` 치환 |
| `README.md` (프로젝트 루트, ≤30줄 슬림) | `reference/project-files/readme-slim.md` | `{KR_NAME}` `{PURPOSE}` `{MONOREPO_LAYOUT}` `{PERIOD}` `{INFRA}` `{STACK_BACKEND}` `{STACK_USER_FE}` 치환 |

### `.github/workflows/ci.yml` (HARNESS == GitHub Actions일 때만)

1. `Read reference/ci-workflows/github-actions.yml.template`
2. 사용 안 하는 컴포넌트 job(backend/user-fe/admin-fe/mobile) 삭제 — 주석 포함 모두 제거
3. 각 job의 setup·lint·test step을 STACK_*·LINTER_*·TEST_UNIT_* 응답에 맞춰 자동 선택 (Python/Java/Node/Go/.NET 분기는 템플릿 주석 예시 참조)
4. security job의 SAST/SCA/이미지 스캔 step을 `STATIC_*` 응답에 따라 활성화
5. `QUALITY_GATE == Pass 필수`이면 quality-gate job 활성화, 그 외 제거
6. `.github/workflows/ci.yml`로 Write

다른 HARNESS(GitLab CI / Jenkins / Argo CD)는 본 릴리스 범위 외. `scripts/git-setup.md`에 안내됨.

---

## STEP 7.5 — git init + 첫 commit (자동)

STEP 0~7 완료 직후 실행 (OS에 맞게 분기):

```bash
git init -b main                              # 또는 git init && git symbolic-ref HEAD refs/heads/main
git add -A
git commit -m "chore(meta): 프로젝트 초기화 (si-project plugin v2.1.0)

- 사용 컴포넌트: {MONOREPO_LAYOUT}
- Branch 전략: {GIT_FLOW_STYLE}
- Quality Gate: {QUALITY_GATE}
- 커버리지 목표: {COVERAGE_TARGET}
"
```

`GIT_FLOW_STYLE == git-flow`면 `git branch develop` 추가.

> Mobile이 있으면 README에 `git lfs install` 안내 (실행은 사용자가 직접).
> `git remote add origin <URL>` + `git push -u origin main`은 사용자가 수동 수행.
> `scripts/git-setup.md`의 원격 정책 1회 셋업도 사용자가 GitHub UI에서 적용.

---

## STEP 8 — 완료 보고

### 생성된 파일 트리 출력
`ls -R docs/ .claude/ .github/` (또는 `Get-ChildItem -Recurse`).

### 즉시 작성 권장 문서 (Top 3)
1. `docs/01-requirements/requirements.md` — 기능/비기능 요구사항 구체화 (**모든 행에 명확도 등급 표기**)
2. `docs/02-architecture/software-architecture.md` — 아키텍처 결정 상세화
3. `docs/02-architecture/security-definition.md` — 보안 위협 모델 구체화

### 다음 단계 안내 (사용자에게 한 줄로 출력)

```
다음 권장:
  1) 요구사항 정의 → `/si-project:project-milestone 01-requirements`
  2) 구현 시작 전 명확성 게이트 → `/si-project:project-check`
```

### 스프린트 1 준비 체크리스트
- [ ] 각 컴포넌트 framework CLI 초기화 완료
- [ ] 요구사항 기술서 1차 리뷰 완료
- [ ] 아키텍처 기술서 팀 공유·승인
- [ ] 보안 정의서 보안 전문가 검토
- [ ] 제품 백로그 우선순위 확정
- [ ] Definition of Done 팀 합의
- [ ] `.claude/settings.json` 권한 조정 완료
- [x] git 초기화 + 첫 커밋 (project-setup이 자동 처리)
- [ ] `git remote add origin <URL>` + 최초 push
- [ ] `scripts/git-setup.md` 따라 branch protection / CODEOWNERS / Required checks 적용
- [ ] CI 파이프라인 초기 설정 (린터·테스트·SAST·SCA·이미지 스캔)
- [ ] (Mobile 있으면) `git lfs install` + LFS track 활성화
- [ ] `/si-project:project-summary` 시범 실행 — 30줄 요약 정상 출력 확인
- [ ] (선택) `.claude/lessons.md`에 팀 lesson 1건 추가
- [ ] `pip install pre-commit && pre-commit install` — commit 가드 활성화 (1회)
- [ ] (HARNESS=GitHub Actions) `.github/workflows/ci.yml` 잡 검토 + 도구 버전 핀 조정
- [ ] `docs/02-architecture/adr/0001-stack-selection.md` 읽고 첫 ADR 내용 검토
