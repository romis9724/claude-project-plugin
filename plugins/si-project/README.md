# si-project — AI Agent 기반 SI 프로젝트 산출물 플러그인 (v2.3.1)

한국 SI 발주처 표준 산출물을 AI 에이전트 기반 개발 환경에서 자동 생성하는 플러그인입니다.

발주처는 SI 표준 형식 산출물을 요구하지만, 개발은 AI 에이전트와 자유롭게 진행. 본 플러그인은 **출력 형식 보장**(워크플로우 강제 ❌) + **명확성 게이트**(모호 요구사항 추측 구현 차단) + **도메인 검토 게이트**(작성 직후 검토자 subagent 호출) + **원격 작업 큐**(`.claude/inbox.md` + GitHub Issues 통합 표시)를 목표로 합니다.

> **v2.3.1 (2026-05)**: 비용 통제·중복 제거·검증 기반 보강 패치. (1) `REVIEWER_SUBAGENT` 플래그 신설(`off` / `critical-only` 기본 / `on`) — 마일스톤 일괄 생성 시 sonnet 7개 산출물의 호출 비용을 기본 차단, opus 3개 핵심 산출물만 호출. (2) 검토자 호출 결과를 산출물 파일 footer에 `<!-- review-history -->` HTML 주석으로 누적 — 사후 추세 확인·v2.4 효과 측정 근거. (3) project-inbox STEP 3 자동 도출을 `/si-project:project-check`로 위임 단순화(중복 제거). (4) 구현 단계 subagent 가이드를 CLAUDE.md에서 plugin reference로 분리(컨텍스트 절약). (5) `GH_CLI_PATH` 변수 신설(Windows PATH 미반영 환경 지원). v2.3.0과 하위 호환.
>
> **v2.3.0 (2026-05)**: 원격 작업 큐(`/si-project:project-inbox`) 신설. AI agent 비대기 상태에서 사용자가 모바일·웹·다른 디바이스에서 큐잉한 다음 task를 한 화면(30줄)에 표시. 로컬 `.claude/inbox.md` + GitHub Issues(label:`si-project-inbox`) + project-check 4축 자동 도출 통합. CLAUDE.md에 "원격 작업 큐" 가이드 섹션 추가(Anthropic 공식 도구 + 입력 채널). v2.2.0과 하위 호환.
>
> **v2.2.0 (2026-05)**: 도메인 검토 체크리스트 + 검토자 subagent 패턴 도입. 핵심 10개 산출물(project-charter / requirements / system-vision / software-architecture / security-definition / db-design / test-plan / system-architecture / risk-register / change-request)에 산출물 고유의 도메인 함정 5±2줄을 박고, 작성 직후 별도 subagent가 반영 여부를 판단해 1줄로 보고. CLAUDE.md에 구현 단계 subagent 가이드 섹션 추가. v2.1.0과 하위 호환.
>
> **v2.1.0 (2026-05)**: 명확성 게이트(`/si-project:project-check`) 신설. 요구사항 산출물에 `명확도` 등급 컬럼 의무화. CLAUDE.md에 `## 미설정 항목` 섹션 자리 박힘. v2.0.0과 하위 호환.
>
> **v2.0.0 (2026-05)**: 플러그인 이름 `project` → `si-project`, 스킬 이름 모두 변경 (breaking). 마이그레이션은 저장소 루트 [README.md](../../README.md#v1x--v200-마이그레이션) 참조.

---

## 7개 스킬

### 1. `/si-project:project-setup` — 신규 SI 프로젝트 셋업

빈 프로젝트 디렉토리에서 한 번 실행. AI 에이전트 기반 SI 개발에 필요한 초기 환경 일괄 설정.

생성물:
- `CLAUDE.md` (≤150줄, 프로젝트 메타·코드 컨벤션·5개 스킬 안내 포함)
- `.claude/settings.json` (스택 기반 권한·hooks. SessionStart hook 포함)
- `.claude/lessons.md` (팀 공유 lesson)
- `.pre-commit-config.yaml` (gitleaks + 컴포넌트별 lint)
- `.github/workflows/ci.yml` (HARNESS=GitHub Actions일 때)
- `.github/PULL_REQUEST_TEMPLATE.md`, `.github/CODEOWNERS`
- `.gitignore`, `.gitattributes`, `CHANGELOG.md`, `scripts/git-setup.md`
- `docs/{00-kickoff ~ 07-delivery}/` 디렉토리 + ADR 디렉토리
- 핵심 7개 문서 충실 생성 (requirements, software-architecture, security-definition, db-design, test-plan, product-backlog, definition-of-done)
- ADR-0001 (스택 선택 근거 자동 기록)
- 핵심 5~8개 마크다운 템플릿 조건부 복사
- git init + 첫 커밋 자동 실행

### 2. `/si-project:project-document <문서명>` — 개별 산출물 생성

발주처가 특정 문서를 요청했을 때 1건 생성.

```
/si-project:project-document 인터페이스요구사항정의서
/si-project:project-document interface-requirements
```

동작:
1. `reference/doc-catalog.md`에서 문서 식별 (Grep 기반 lazy 매칭) → 마일스톤·템플릿 결정
2. 템플릿 + `reference/methodology.md` 작성 가이드(마커 기반 부분 로드)
3. 현재 프로젝트 상태(`CLAUDE.md`, `docs/`, 소스 코드) 분석
4. `.claude/lessons.md` 컨텍스트 반영
5. 정보 채움 + 사람 입력 필요 항목은 `> ⚠️ TODO` 표기
6. `docs/{milestone}/{file}.md` 저장 + CHANGELOG.md 1줄 추가

### 3. `/si-project:project-milestone <마일스톤>` — 마일스톤 묶음 생성

마일스톤별 필수 산출물 일괄 생성 + 문서간 일관성 점검.

```
/si-project:project-milestone 01-requirements
/si-project:project-milestone 요구분석
```

동작:
1. 카탈로그에서 마일스톤의 `필수도=필수` 문서 추출 (섹션 단위 부분 로드)
2. 누락 문서 일괄 생성 (공통 컨텍스트 캐시 사용, **순차 처리** — subagent 의도적 미도입으로 일관성 보장)
3. 용어 통일·크로스 레퍼런스·6관점 점검
4. TODO 통합 리스트 보고 + CHANGELOG.md 1줄 추가

### 4. `/si-project:project-summary` — 현황·오늘 할 일 요약

새 세션 진입·일감 파악 시점에 호출. 30줄 이하 5섹션 요약 (읽기 전용).

소스: `CLAUDE.md` + `.claude/project-context.json` + `CHANGELOG.md` + `git log -10` + `docs/` 산출물 카운트.

출력 섹션: 프로젝트 / 현재 마일스톤 / 최근 변경 / 완료된 산출물 / 다음 권장 액션.

### 5. `/si-project:project-adr <결정 제목>` — 아키텍처 결정 기록

MADR(Markdown Architecture Decision Records) 형식 결정 기록 1건 생성. 자동 번호 부여.

```
/si-project:project-adr "PostgreSQL을 RDB로 채택"
```

저장 위치: `docs/02-architecture/adr/NNNN-{slug}.md`. `adr/README.md` 인덱스 자동 갱신 + CHANGELOG.md 1줄 추가.

### 6. `/si-project:project-check` — 구현 준비도(명확성 게이트) 점검 ★ v2.1.0 신설

본격 구현 시작 또는 마일스톤 진입 직전에 호출. **읽기 전용, 30줄 이하**.

```
/si-project:project-check
```

스캔 대상 4축:
1. 요구사항 산출물(`docs/01-requirements/`)의 `명확도: 명확/모호/미정` 등급 카운트 + `TBD` 패턴
2. `.env.example` / `.env`의 빈 슬롯
3. CLAUDE.md `## 미설정 항목` 체크리스트 미체크 행
4. `.claude/project-context.json` 누락 행정 필드(client·acceptance·delivery_date)

산출: 0~100점 준비도 점수 + 등급(통과 ≥80 / 보강 50~79 / 비권장 <50) + 권장 액션 3건.

> **목적**: AI 에이전트가 모호한 요구사항을 추측으로 구현하는 사고를 막는 자기 점검 게이트. 차단이 아니라 신호.

### 7. `/si-project:project-inbox` — 원격 작업 큐 ★ v2.3.0 신설

AI agent 작업 완료 후·새 세션 진입 시, 사용자가 모바일·웹·다른 디바이스에서 큐잉한 "다음 할 일"을 한 화면으로 표시. **읽기 전용, 30줄 이하**.

```
/si-project:project-inbox
```

통합 소스:
- 로컬 `.claude/inbox.md` (P0/P1/P2 우선순위 분류, 폐쇄망·오프라인 친화)
- GitHub Issues (label: `si-project-inbox`) — 모바일 GitHub 앱·웹에서 즉시 추가 가능. `gh` 미설치/폐쇄망 시 자동 스킵
- `/si-project:project-check` 4축 자동 도출 (요구사항 모호도, `.env` 빈 슬롯, 미설정 항목, TODO 카운트)

출력 말미에 "다음 추천: {최상위 P0 항목}" 1줄 제안.

> **위치 부재**: Anthropic 공식 도구(claude.ai/code, IDE 확장, PushNotification, CronCreate) 우선 활용 권장. 본 skill은 그 갭을 채우는 inbox 표시기. plugin이 외부 ITS API를 자동 호출해 issue 생성·close하지 않음 (사용자 명시 액션만).

---

## 플러그인 구조

```
si-project/
├── .claude-plugin/plugin.json     (v2.3.1)
├── reference/
│   ├── methodology.md           ★ 방법론 지식 + 명확도 등급(§0) + 도메인 체크리스트(v2.2, 10개)
│   ├── doc-catalog.md           ★ 106개 문서 카탈로그
│   └── subagent-patterns.md     ★ v2.3.1 — 구현 단계 Agent tool 활용 가이드(분리)
├── skills/
│   ├── project-setup/
│   │   ├── SKILL.md            (워크플로 로직만, CLAUDE.md ## 미설정 항목 자리 생성)
│   │   └── reference/          (questions, claude-md-template, settings, git-files[+inbox-init.md v2.3], project-files, adr, ci-workflows)
│   ├── project-document/SKILL.md   (v2.2 검토자 subagent 호출)
│   ├── project-milestone/SKILL.md  (v2.2 검토자 subagent 호출 + 합산)
│   ├── project-summary/SKILL.md    (v2.3 inbox 카운트 1줄 포함)
│   ├── project-adr/SKILL.md
│   ├── project-check/SKILL.md      ★ v2.1.0 (명확성 게이트)
│   └── project-inbox/SKILL.md      ★ v2.3.0 신설 (원격 작업 큐)
├── templates/                   (106개 마크다운 템플릿)
│   ├── 00-kickoff/      (12)
│   ├── 01-requirements/ (11)
│   ├── 02-architecture/ (13)
│   ├── 03-design/       (23)
│   ├── 04-implementation/ (8)
│   ├── 05-testing/      (4)
│   ├── 06-management/   (19)
│   └── 07-delivery/     (16)
└── README.md
```

---

## 6개 관점 커버리지

| 관점 | 핵심 점검 |
|------|----------|
| PMO | 일정·범위·예산·이해관계자 |
| PM | 인력·리스크·의사소통·이슈 |
| 아키텍처 | 기술 선택·구조·NFR·확장성 |
| DBA | 데이터 모델·성능·무결성·백업 |
| 하네스 엔지니어 | 빌드·배포·CI/CD·환경 분리 |
| 보안 전문가 | 위협 모델·인증·암호화·감사 |

각 산출물 생성 시 본 6관점을 체크리스트로 적용.

---

## 도메인 검토 체크리스트 (v2.2.0)

`reference/methodology.md`의 핵심 10개 산출물 DOC 블록에 산출물 고유의 **도메인 함정 체크리스트** 5±2줄을 박았습니다. `/si-project:project-document`·`/si-project:project-milestone`이 산출물 작성 직후 **검토자 subagent**를 호출해 각 항목 반영 여부를 1줄로 보고합니다.

대상 10개 산출물 (모델 정책: `opus` ★ / `sonnet` ☆):
- ★ requirements — 명확도 등급, NFR 정량화, ID 충돌
- ★ software-architecture — 책임 단일성, 통신 패턴, fault isolation
- ★ security-definition — 인증 매트릭스, 키 회전, 망분리, 규제 매핑
- ☆ project-charter — 스폰서, KPI, 범위 In/Out, DoD
- ☆ system-vision — As-Is/To-Be 격차, 가치 제안 압축, 측정 지표
- ☆ db-design — 인덱스 전략, 정규화 결정, RPO/RTO, 파티셔닝
- ☆ test-plan — 테스트 피라미드, 비기능 시나리오, 격리 전략
- ☆ system-architecture — 물리 토폴로지, 네트워크 가정, DR site, 비용
- ☆ risk-register — 확률×영향, 전략 분류, owner·트리거, residual
- ☆ change-request — 영향 범위, 거절 리스크, 회귀 테스트 범위

상세 항목은 `reference/methodology.md` 각 DOC 블록 참조. 나머지 10개 산출물은 v2.3에서 보강 예정.

---

## 8단계 매핑

| 단계 | 디렉토리 | 필수 산출물 |
|------|---------|------------|
| 착수 | 00-kickoff | 프로젝트정의서·수행계획서·위험분석서·개발환경정의서 |
| 요구분석 | 01-requirements | 요구사항기술서·시스템비전기술서·사용자정의서·용어집 |
| 아키텍처 | 02-architecture | 소프트웨어아키텍처기술서·시스템아키텍처정의서·보안정의서·데이터모형기술서 + ADR |
| 상세설계 | 03-design | 컴포넌트명세서·데이터베이스설계서·테이블정의서·화면정의서 |
| 구현 | 04-implementation | 제품백로그·완료기준(DoD) |
| 시험 | 05-testing | 테스트계획서·설계서·수행보고서·결과보고서 |
| 관리 | 06-management | 변경요청·품질관리계획·형상관리·진척보고 (지속) |
| 인도 | 07-delivery | 시스템설치보고서·데이터전환보고서·시스템전환계획서·사용자지침서·운영계획서·완료보고서 |

---

## 워크플로우 예시 (SI + AI 에이전트)

```
[Day 1] 프로젝트 시작
  $ /si-project:project-setup
  → CLAUDE.md, 초기 docs/, settings, pre-commit, CI, ADR-0001 생성 + git init + 첫 커밋
  $ pip install pre-commit && pre-commit install   # 1회 셋업

[Day 2] 본격 구현 시작 전 — 명확성 게이트
  $ /si-project:project-check
  → 0~100 준비도 점수. 모호·미정 요구사항·미설정 정보를 카운트해 노출
  → 점수 <50이면 요구사항 보강 후 진행 권장

[Day 2~30] AI 에이전트와 자유롭게 개발
  - Claude와 대화하며 코드 작성
  - docs/ 핵심 7개 문서를 점진적으로 보완
  - 단계 게이트 강제 없음
  - 결정 시점에 $ /si-project:project-adr "결정 제목"
  - 새 모호 요구사항 발견 시 CLAUDE.md ## 미설정 항목에 1줄 기록

[새 세션 진입]
  $ /si-project:project-summary
  → 30줄 요약 (현황·할 일)

[Day 31] 발주처 요구분석 산출물 납품 요청
  $ /si-project:project-milestone 01-requirements
  → 필수 4개 + 권장 7개 문서 일괄 생성
  → 일관성 점검 보고
  → 사람이 TODO 채우고 검토

[Day 60] 발주처가 "인터페이스 정의서" 추가 요청
  $ /si-project:project-document 인터페이스요구사항정의서
  → 현재 src/api/ 코드 + CLAUDE.md 기반 생성

[Day 120] 인도 단계 산출물 납품
  $ /si-project:project-milestone 07-delivery
  → 시스템설치보고서, 사용자지침서 등 16개 일괄 생성
```

---

## 설계 원칙

- **워크플로 강제 X, 출력 형식 보장 O**: 발주처에 납품할 때 표준 형식만 유지. 개발 단계 게이트는 강제하지 않음
- **명확성 게이트 (v2.1.0+)**: 모호한 요구사항·미설정 외부 정보 상태에서 AI가 추측 구현하는 사고를 신호화. `project-check`는 차단이 아닌 자기 점검 도구
- **Lazy loading**: methodology.md/doc-catalog.md/CLAUDE.md 모두 마커·부분 로드. 컨텍스트 절약
- **작성 subagent 미도입, 검토 subagent 도입 (v2.2)**: project-document·project-milestone은 산출물 *작성*을 순차로 (cross-reference 손실 회피). 작성 직후 *검토*만 별도 subagent로 분리 — 검토는 단일 산출물 대상이라 격리 가능. 페르소나 정신을 검토 단계에서만 실현
- **자동 기록 X**: `.claude/lessons.md`는 사용자가 직접 큐레이션. 자동 누적은 편향 위험
- **팀 공유 우선**: lessons.md는 gitignore에서 제외. 팀 공유 가치 있는 lesson만 기록

---

## 팀 공유

플러그인 디렉토리가 자기완결 단위:
- 모든 스킬·템플릿·방법론 문서가 디렉토리 내부에 포함
- GitHub 마켓플레이스 통해 배포 (`romis9724/claude-project-plugin`)
- 팀원은 `extraKnownMarketplaces` 설정 + `enabledPlugins.si-project@team-tools: true`로 자동 설치
