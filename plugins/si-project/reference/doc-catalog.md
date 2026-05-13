---
title: SI 산출물 문서 카탈로그
description: 106개 SI 표준 산출 문서 카탈로그. project-document·project-milestone 스킬이 참조하여 템플릿·마일스톤·필수도를 결정한다.
version: 1.0
---

# SI 산출물 문서 카탈로그 (106개)

## 사용법

- `project-document` 스킬: 사용자 입력 한글명/영문명을 본 카탈로그에서 매칭 → 템플릿 경로 확정
- `project-milestone` 스킬: 입력 마일스톤의 `필수도=필수` 항목 모두 일괄 생성

### Lazy-load 접근 패턴 (v2.4)

본 파일 12KB. **전체 Read 금지**. 다음 패턴으로 접근:

- **project-document 스킬 (단건 매칭)**: `Grep pattern="<사용자입력>" output_mode="content" -C=1`
  - 한글 입력은 그대로, 영문 입력은 `.md` 포함해 매칭
  - 결과 1줄에 영문 파일명·한글명·필수도·마일스톤 행이 잡힘
- **project-milestone 스킬 (마일스톤 일괄)**: 섹션 헤더 위치만 확보 후 부분 Read
  - `Grep pattern="^## {milestone}"` → 시작 라인 N
  - 다음 섹션 시작 라인 또는 `## 합계` 라인 → 종료 라인 M
  - `Read offset=N, limit=(M-N+1)` → 해당 마일스톤 ~25줄만 로드
- **마일스톤 섹션 라인 가이드** (변경 시 갱신 필요):
  - 00-kickoff ≈ line 22~, 01-requirements ≈ 39~, 02-architecture ≈ 55~, 03-design ≈ 73~,
  - 04-implementation ≈ 101~, 05-testing ≈ 114~, 06-management ≈ 123~, 07-delivery ≈ 147~

## 필수도 기준

- **필수**: 발주처 표준 납품물. 대부분의 SI 프로젝트에서 요구됨
- **권장**: 프로젝트 규모·성격에 따라 작성. 일반적으로 작성
- **선택**: 특수 상황(품질감사·이슈 발생·외주 등)에서만 작성

---

## 00-kickoff (12개) — 착수·계획

| 영문 파일명 | 한글 원본명 | 필수도 | 용도 |
|------------|-----------|-------|------|
| project-charter.md | 프로젝트정의서 | 필수 | 프로젝트 목적·범위·제한사항·접근법 정의 |
| project-plan.md | 프로젝트수행계획서 | 필수 | 단계·일정·자원·의사소통·게이트 계획 |
| project-overview.md | 프로젝트개요서 | 권장 | 경영진/이해관계자용 요약 |
| project-proposal.md | 프로젝트기안서 | 권장 | 내부 착수 승인용 기안 |
| project-strategy.md | 프로젝트수행전략기술서 | 권장 | 방법론·기법·접근 전략 상세 |
| project-revision-plan.md | 프로젝트수정계획서 | 선택 | 일정/범위 변경 시 |
| project-budget.md | 프로젝트예산내역서 | 권장 | 비용 구성·집행 계획 |
| risk-register.md | 위험분석서 | 필수 | 위험 식별·관리표·해결방안 |
| mini-project-plan.md | 미니프로젝트수행계획서 | 선택 | 점진적 개발용 미니 단위 계획 |
| phase-work-plan.md | 단계작업계획서 | 권장 | 단계 진입 시 상세 작업 계획 |
| dev-environment-definition.md | 프로젝트개발환경정의서 | 필수 | HW/SW/네트워크/도구 환경 명세 |
| kickoff-document.md | 착수문서 | 권장 | 킥오프 미팅 자료 |

## 01-requirements (11개) — 요구분석

| 영문 파일명 | 한글 원본명 | 필수도 | 용도 |
|------------|-----------|-------|------|
| requirements.md | 요구사항기술서 | 필수 | 기능·비기능 요구사항 명세 |
| system-vision.md | 시스템비전기술서 | 필수 | 비전·목표·핵심가치·KPI |
| initial-system-vision.md | 초기시스템비전정의서 | 권장 | 착수 직후 초안 비전 |
| user-needs.md | 사용자요구수집서 | 권장 | 인터뷰·조사 정리 |
| actors.md | 사용자정의서 | 필수 | 행위자·페르소나·외부시스템 |
| process-hierarchy.md | 업무프로세스계층모형기술서 | 권장 | 업무 프로세스 계층 구조 |
| process-activity.md | 업무프로세스활동모형기술서 | 권장 | 프로세스별 활동 모형 |
| as-is-analysis.md | 현행시스템분석서 | 권장 | AS-IS 시스템 분석 |
| collected-data.md | 수집자료정의서 | 선택 | 수집 자료 메타 정리 |
| glossary.md | 용어집 | 필수 | 도메인 용어 정의 |
| system-environment.md | 시스템환경정의서 | 권장 | 운영 환경 요구사항 |

## 02-architecture (13개) — 아키텍처

| 영문 파일명 | 한글 원본명 | 필수도 | 용도 |
|------------|-----------|-------|------|
| software-architecture.md | 소프트웨어아키텍처기술서 | 필수 | SW 아키텍처 결정·근거·뷰 |
| software-architecture-spec.md | 소프트웨어아키텍처정의서 | 권장 | SW 아키텍처 상세 정의 |
| software-architecture-evaluation.md | 소프트웨어아키텍처평가서 | 선택 | 아키텍처 검토 결과 |
| system-architecture.md | 시스템아키텍처정의서 | 필수 | 인프라·배포·네트워크 |
| architecture-deployment-plan.md | 아키텍처전개계획서 | 권장 | 아키텍처 적용 계획 |
| architecture-deployment-report.md | 아키텍처전개보고서 | 선택 | 아키텍처 적용 결과 |
| design-patterns.md | 설계패턴기술서 | 권장 | 적용 디자인 패턴 |
| variability-design.md | 가변성설계서 | 선택 | 제품 라인 가변성 |
| security-definition.md | 보안정의서 | 필수 | STRIDE·보안 모델·통제 |
| conceptual-model.md | 개념모형기술서 | 권장 | 도메인 개념 모형 |
| object-model.md | 객체모형기술서 | 권장 | 객체지향 모형 |
| data-model.md | 데이터모형기술서 | 필수 | 데이터 모형(개념·논리) |
| logical-access-model-rdb.md | 논리적접근모형기술서(RDB) | 권장 | RDB 논리적 접근 모형 |

## 03-design (22개) — 상세설계

| 영문 파일명 | 한글 원본명 | 필수도 | 용도 |
|------------|-----------|-------|------|
| application-spec.md | 어플리케이션정의서 | 권장 | 애플리케이션 단위 정의 |
| service-definition.md | 서비스컴포넌트정의서 | 권장 | 서비스 컴포넌트 명세 |
| component-spec.md | 컴포넌트명세서 | 필수 | 컴포넌트 인터페이스·역할 |
| component-design.md | 컴포넌트설계서 | 권장 | 컴포넌트 내부 설계 |
| transaction-design.md | 트랜잭션설계서 | 권장 | 트랜잭션 흐름 설계 |
| transaction-spec.md | 트랜잭션정의서 | 권장 | 트랜잭션 단위 정의 |
| db-design.md | 데이터베이스설계서(RDB) | 필수 | ERD·테이블·인덱스·마이그레이션 |
| db-spec.md | 데이터베이스정의서(RDB) | 권장 | DB 인스턴스·스키마 정의 |
| table-spec.md | 테이블정의서 | 필수 | 테이블·컬럼·제약 명세 |
| index-spec.md | 인덱스정의서(RDB) | 권장 | 인덱스 전략 |
| view-spec.md | 뷰정의서 | 권장 | DB 뷰 정의 |
| sql-spec.md | SQL정의서 | 선택 | 주요 SQL 카탈로그 |
| ui-design.md | UI설계서 | 권장 | UI 설계 원칙·가이드 |
| ui-flow.md | UI흐름정의서 | 권장 | 화면 흐름·내비게이션 |
| screen-spec.md | 화면정의서 | 필수 | 화면 단위 상세 |
| report-spec.md | 보고서정의서 | 권장 | 리포트·출력물 정의 |
| namespace-spec.md | 네임스페이스정의서 | 선택 | 네임스페이스 구조 |
| directory-spec.md | 디렉토리정의서 | 선택 | 소스 디렉토리 구조 |
| package-server-spec.md | 패키지정의서(서버) | 권장 | 서버 패키지 구조 |
| package-web-spec.md | 패키지정의서(웹) | 권장 | 웹 패키지 구조 |
| batch-web-spec.md | 배치정의서(웹) | 선택 | 웹 배치 작업 정의 |
| reusable-component-candidates.md | 재사용컴포넌트후보목록 | 선택 | 재사용 후보 식별 |
| api-design.md | (현대 추가) | 권장 | RESTful API 엔드포인트 설계 |

## 04-implementation (4개) — 구현

| 영문 파일명 | 한글 원본명 | 필수도 | 용도 |
|------------|-----------|-------|------|
| implementation-class-model.md | 구현클래스모형기술서 | 권장 | 클래스 다이어그램·구조 |
| work-order.md | 작업지시서 | 선택 | 작업 단위 지시 |
| critical-issues-resolution.md | 핵심과제해결방안내역서 | 권장 | 주요 이슈 해결 방안 |
| review-comment.md | 검토의견서 | 권장 | 코드/문서 리뷰 의견 |
| product-backlog.md | (애자일 추가) | 필수 | 제품 백로그(에픽·스토리) |
| definition-of-done.md | (애자일 추가) | 필수 | 완료 기준 정의 |
| sprint-template.md | (애자일 추가) | 권장 | 스프린트 계획서 양식 |
| sprint-retrospective.md | (애자일 추가) | 권장 | 스프린트 회고 양식 |

## 05-testing (4개) — 시험

| 영문 파일명 | 한글 원본명 | 필수도 | 용도 |
|------------|-----------|-------|------|
| test-plan.md | 테스트계획서 | 필수 | 테스트 전략·범위·일정 |
| test-design.md | 테스트설계서 | 필수 | 테스트 케이스 설계 |
| test-execution-report.md | 테스트수행보고서 | 필수 | 테스트 실행 결과 |
| test-result-report.md | 테스트결과보고서 | 필수 | 테스트 종합 결과 |

## 06-management (18개) — 공통관리

| 영문 파일명 | 한글 원본명 | 필수도 | 용도 |
|------------|-----------|-------|------|
| change-request.md | 변경요청기술서 | 필수 | 변경 요청 단위 기록 |
| change-request-template.md | (인라인 추가) | 필수 | 변경 요청 양식 |
| quality-plan.md | 품질관리계획서 | 필수 | 품질 목표·게이트·SLA |
| quality-procedure.md | 품질관리절차서 | 권장 | 품질 활동 절차 |
| quality-team-plan.md | 품질관리팀운영계획서 | 선택 | 품질팀 운영 계획 |
| quality-result.md | 품질관리수행결과서 | 권장 | 품질 활동 결과 |
| qa-strategy.md | 품질보증전략기술서 | 권장 | QA 전략 |
| quality-inspection-result.md | 품질점검결과서 | 권장 | 품질 점검 결과 |
| quality-checklist.md | 사전품질점검목록 | 권장 | 사전 점검 체크리스트 |
| config-management-list.md | 형상관리목록 | 권장 | 형상관리 대상 목록 |
| config-management.md | 형상관리절차서 | 필수 | 브랜치·커밋·CI/CD |
| config-status-report.md | 형상상태보고서 | 선택 | 형상 상태 정기 보고 |
| document-management.md | 문서관리절차서 | 권장 | 문서 명명·버전·승인 절차 |
| project-progress-report.md | 프로젝트진척보고서 | 필수 | 정기 진척 보고 |
| project-issues.md | 프로젝트현안목록 | 권장 | 현안·이슈 트래킹 |
| performance-evaluation.md | 수행능력평가서 | 선택 | 팀/개인 수행 능력 |
| phase-evaluation.md | 단계평가서 | 권장 | 단계 종료 평가 |
| phase-evaluation-result.md | 단계평가결과서 | 권장 | 단계 평가 결과 |
| mini-project-result.md | 미니프로젝트수행결과서 | 선택 | 미니 프로젝트 결과 |

## 07-delivery (16개) — 설치·인도·종료

| 영문 파일명 | 한글 원본명 | 필수도 | 용도 |
|------------|-----------|-------|------|
| system-installation-report.md | 시스템설치보고서 | 필수 | 시스템 설치 결과 |
| application-installation-report.md | 응용시스템설치보고서 | 권장 | 응용 시스템 설치 결과 |
| platform-installation-report.md | 플랫폼설치보고서 | 권장 | 플랫폼 설치 결과 |
| data-migration-report.md | 데이터전환보고서 | 필수 | 데이터 이전 결과 |
| system-transition-plan.md | 시스템전환계획서 | 필수 | 시스템 전환 계획 |
| system-observation-report.md | 시스템관찰보고서 | 권장 | 운영 관찰 결과 |
| user-training-plan.md | 사용자교육계획서 | 권장 | 사용자 교육 계획 |
| user-training-report.md | 사용자교육보고서 | 권장 | 사용자 교육 결과 |
| training-report.md | 교육훈련보고서 | 권장 | 운영자 교육 결과 |
| user-manual.md | 사용자지침서 | 필수 | 최종 사용자 매뉴얼 |
| operations-plan.md | 운영계획서 | 필수 | 운영 계획·체계 |
| contingency-plan.md | 비상대책방안정의서 | 권장 | 장애·재해 대응 |
| project-closure-report.md | 프로젝트완료보고서 | 필수 | 프로젝트 종료 보고 |
| project-result-document.md | 프로젝트결과문서 | 권장 | 종합 결과 문서 |
| project-resource-return.md | 프로젝트자원반환방안 | 권장 | 자원 반환 절차 |
| as-is-backup-list.md | 현행시스템백업목록 | 권장 | AS-IS 백업 목록 |

---

## 합계

| 마일스톤 | 필수 | 권장 | 선택 | 합계 |
|---------|------|------|------|------|
| 00-kickoff | 3 | 7 | 2 | 12 |
| 01-requirements | 3 | 7 | 1 | 11 |
| 02-architecture | 3 | 8 | 2 | 13 |
| 03-design | 4 | 14 | 5 | 23 |
| 04-implementation | 2 | 5 | 1 | 8 |
| 05-testing | 4 | 0 | 0 | 4 |
| 06-management | 4 | 11 | 4 | 19 |
| 07-delivery | 6 | 10 | 0 | 16 |
| **합계** | **29** | **62** | **15** | **106** |

> 본 카탈로그는 100개 원본 .doc + 6개 현대 추가(api-design, product-backlog, definition-of-done, sprint-template, sprint-retrospective, change-request-template) = 106개 문서를 다룬다.
