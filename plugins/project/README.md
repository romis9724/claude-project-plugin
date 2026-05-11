# project — AI Agent 기반 SI 프로젝트 산출물 플러그인

한국 SI 발주처 표준 산출물을 AI 에이전트 기반 개발 환경에서 자동 생성하는 플러그인입니다.

발주처는 SI 표준 형식 산출물을 요구하지만, 개발은 AI 에이전트와 자유롭게 진행. 본 플러그인은 **출력 형식 보장**(워크플로우 강제 ❌)을 목표로 합니다.

---

## 3개 스킬

### 1. `/project:init` — 프로젝트 초기화

빈 프로젝트 디렉토리에서 한 번 실행. AI 에이전트 기반 SI 개발에 필요한 초기 환경 일괄 설정.

생성물:
- `CLAUDE.md` (프로젝트 메타·코드 컨벤션·3개 스킬 안내 포함)
- `.claude/settings.json` (스택 기반 권한·훅)
- `docs/{00-kickoff ~ 07-delivery}/` 디렉토리
- 핵심 7개 문서 충실 생성 (requirements, software-architecture, security-definition, db-design, test-plan, product-backlog, definition-of-done)
- 핵심 15개 마크다운 템플릿 복사

### 2. `/project:doc <문서명>` — 개별 산출물 생성

발주처가 특정 문서를 요청했을 때 1건 생성.

```
/project:doc 인터페이스요구사항정의서
/project:doc interface-requirements
```

동작:
1. `reference/doc-catalog.md`에서 문서 식별 → 마일스톤·템플릿 결정
2. 템플릿 + `reference/methodology.md` 작성 가이드 로드
3. 현재 프로젝트 상태(`CLAUDE.md`, `docs/`, 소스 코드) 분석
4. 정보 채움 + 사람 입력 필요 항목은 `> ⚠️ TODO` 표기
5. `docs/{milestone}/{file}.md` 저장

### 3. `/project:bundle <마일스톤>` — 마일스톤 묶음 생성

마일스톤별 필수 산출물 일괄 생성 + 문서간 일관성 점검.

```
/project:bundle 01-requirements
/project:bundle 요구분석
```

동작:
1. 카탈로그에서 마일스톤의 `필수도=필수` 문서 추출
2. 누락 문서 일괄 생성 (공통 컨텍스트 캐시 사용)
3. 용어 통일·크로스 레퍼런스·6관점 점검
4. TODO 통합 리스트 보고

---

## 플러그인 구조

```
project/
├── .claude-plugin/plugin.json
├── reference/
│   ├── methodology.md           ★ 방법론 지식 (작성 가이드)
│   └── doc-catalog.md           ★ 106개 문서 카탈로그
├── skills/
│   ├── init/SKILL.md
│   ├── doc/SKILL.md
│   └── bundle/SKILL.md
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

## 8단계 매핑

| 단계 | 디렉토리 | 필수 산출물 |
|------|---------|------------|
| 착수 | 00-kickoff | 프로젝트정의서·수행계획서·위험분석서·개발환경정의서 |
| 요구분석 | 01-requirements | 요구사항기술서·시스템비전기술서·사용자정의서·용어집 |
| 아키텍처 | 02-architecture | 소프트웨어아키텍처기술서·시스템아키텍처정의서·보안정의서·데이터모형기술서 |
| 상세설계 | 03-design | 컴포넌트명세서·데이터베이스설계서·테이블정의서·화면정의서 |
| 구현 | 04-implementation | 제품백로그·완료기준(DoD) |
| 시험 | 05-testing | 테스트계획서·설계서·수행보고서·결과보고서 |
| 관리 | 06-management | 변경요청·품질관리계획·형상관리·진척보고 (지속) |
| 인도 | 07-delivery | 시스템설치보고서·데이터전환보고서·시스템전환계획서·사용자지침서·운영계획서·완료보고서 |

---

## 워크플로우 예시 (SI + AI 에이전트)

```
[Day 1] 프로젝트 시작
  $ /project:init
  → CLAUDE.md, 초기 docs/, settings 생성

[Day 2~30] AI 에이전트와 자유롭게 개발
  - Claude와 대화하며 코드 작성
  - docs/ 핵심 7개 문서를 점진적으로 보완
  - 단계 게이트 강제 없음

[Day 31] 발주처 요구분석 산출물 납품 요청
  $ /project:bundle 01-requirements
  → 필수 4개 + 권장 7개 문서 일괄 생성
  → 일관성 점검 보고
  → 사람이 TODO 채우고 검토

[Day 60] 발주처가 "인터페이스 정의서" 추가 요청
  $ /project:doc 인터페이스요구사항정의서
  → 현재 src/api/ 코드 + CLAUDE.md 기반 생성

[Day 120] 인도 단계 산출물 납품
  $ /project:bundle 07-delivery
  → 시스템설치보고서, 사용자지침서 등 16개 일괄 생성
```

---

## 팀 공유

플러그인 디렉토리가 자기완결 단위:
- 모든 스킬·템플릿·방법론 문서가 디렉토리 내부에 포함
- 별도 git 저장소로 분리해 팀원에게 배포 가능
- 팀원은 `~/.claude/plugins/marketplaces/<org>/plugins/project/`에 배치
