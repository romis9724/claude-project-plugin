# project-setup STEP 3 — 핵심 문서 7개 작성 가이드

본 파일은 `/si-project:project-setup` STEP 3에서 7개 핵심 문서를 생성할 때 사용하는 섹션 구조·표 컬럼·다이어그램 지침. 각 문서는 마커로 감싸 있어 Grep으로 위치 잡고 Read offset/limit로 부분 로드 가능.

**공통 생성 규칙**:
- YAML 프론트매터 포함 (`title`, `project`, `version: 0.1`, `status: 초안`, `date`, `owner`)
- 모든 알려진 항목은 STEP 1 답변으로 채움
- 불확실한 항목만 `> ⚠️ TODO: [설명]` 표기
- 구조 데이터는 마크다운 표 사용
- 다이어그램은 Mermaid 코드블록으로 초안 작성

---

<!-- DOC: requirements -->
## 3-1. `docs/01-requirements/requirements.md` — 요구사항 기술서

필수 섹션 및 채움 지침:
- **기능 요구사항 표** (ID·요구사항·우선순위·출처): PURPOSE + SYSTEM_TYPE에서 최소 5~8개 도출
- **비기능 요구사항 표**: 성능(응답시간·TPS)·가용성·확장성 + AI 관련(LLM 지연허용·할루시네이션 대응·프롬프트 인젝션 방어) 항목 반드시 포함
- **제약사항**: 기술(STACK 기반)·법규(SECURITY_REQ 기반)·환경(INFRA 기반) 항목 채움
- **수용 기준**: 주요 FR마다 검증 가능한 조건 작성
<!-- /DOC: requirements -->

<!-- DOC: software-architecture -->
## 3-2. `docs/02-architecture/software-architecture.md` — SW 아키텍처 기술서

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
<!-- /DOC: software-architecture -->

<!-- DOC: security-definition -->
## 3-3. `docs/02-architecture/security-definition.md` — 보안 정의서

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
<!-- /DOC: security-definition -->

<!-- DOC: db-design -->
## 3-4. `docs/03-design/db-design.md` — DB 설계서 (DB ≠ 없음인 경우)

필수 섹션:
- **DB 스택 표**: DBMS·버전·ORM·마이그레이션 툴
- **Mermaid ERD**: PURPOSE + SYSTEM_TYPE에서 핵심 엔티티 최소 3~5개 도출, 관계 포함
- **테이블 정의 표**: 각 테이블별 컬럼명·타입·제약·설명 (nullable 여부 포함)
- **인덱스 전략**: 자주 조회되는 컬럼 식별, 복합 인덱스 필요성 기술
- **마이그레이션 규칙**: 툴(Flyway/Liquibase/Alembic 등)·명명 규칙·롤백 절차
- **민감 정보 암호화**: SECURITY_REQ 기반 암호화 대상 컬럼 식별

DB가 없음이면 이 파일 생성 건너뜀.
<!-- /DOC: db-design -->

<!-- DOC: test-plan -->
## 3-5. `docs/05-testing/test-plan.md` — 테스트 계획서

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
<!-- /DOC: test-plan -->

<!-- DOC: product-backlog -->
## 3-6. `docs/04-implementation/product-backlog.md` — 제품 백로그

필수 섹션:
- **에픽 목록 표** (ID·에픽명·설명·우선순위): PURPOSE + SYSTEM_TYPE에서 최소 4~6개 에픽 도출
- **유저 스토리 표** (ID·에픽ID·스토리·수용기준·추정포인트): 각 에픽당 2~3개 스토리
- **우선순위 기준**: MoSCoW 또는 WSJF 방식 명시
<!-- /DOC: product-backlog -->

<!-- DOC: definition-of-done -->
## 3-7. `docs/04-implementation/definition-of-done.md` — 완료 기준

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
<!-- /DOC: definition-of-done -->
