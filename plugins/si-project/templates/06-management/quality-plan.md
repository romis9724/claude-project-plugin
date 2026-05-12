---
title: 품질 관리 계획서
project: "{{PROJECT_NAME_KR}}"
version: 0.1
status: 초안
date: "{{DATE}}"
owner: "{{QA_LEAD}}"
---

# 품질 관리 계획서: {{PROJECT_NAME_KR}}

## 품질 목표

| 지표 | 목표값 | 측정 방법 | 측정 주기 |
|------|--------|-----------|-----------|
| 단위 테스트 커버리지 | ≥ {{UNIT_COV}}% | CI 리포트 | 매 PR |
| 통합 테스트 커버리지 | ≥ {{INT_COV}}% | CI 리포트 | 매 스프린트 |
| 치명적 버그 수 | 0 | 이슈 트래커 | 매일 |
| 코드 중복도 | ≤ {{DUP_THRESHOLD}}% | SonarQube 등 | 매 PR |
| 기술 부채 비율 | ≤ {{DEBT_THRESHOLD}}% | SonarQube 등 | 주간 |
| API 응답 시간 P95 | ≤ {{RESP_TIME}}ms | APM | 실시간 |

## 품질 게이트

| 단계 | Go 조건 | 담당 |
|------|---------|------|
| PR 병합 | 리뷰어 승인 2인 + CI 통과 + 보안 스캔 통과 | 아키텍트 |
| 스프린트 완료 | DoD 체크리스트 100% + 커버리지 목표 달성 | QA 팀장 |
| 단계 게이트 | 이전 단계 품질 목표 달성 | PMO |
| 인도 | 인수 테스트 100% 통과 + 보안 취약점 0 | PM + 고객 |

## 리뷰 프로세스

### 코드 리뷰
- **대상**: 모든 PR
- **리뷰어**: 아키텍트 1인 + 동료 1인 이상
- **체크 항목**: 설계 준수, 보안, 성능, 테스트 커버리지, 코드 품질

### 문서 리뷰
- **대상**: 모든 핵심 산출물 (requirements, architecture, security, db-design)
- **리뷰어**: 해당 관점 전문가 (PM/아키텍트/DBA/보안)
- **시기**: 문서 초안 완료 후 3영업일 내

## 정적 분석 도구

| 도구 | 대상 | 임계값 | 차단 조건 |
|------|------|--------|---------|
| {{LINTER}} | 코드 스타일 | 에러 0 | PR 블록 |
| {{SAST}} | 보안 취약점 | Critical 0 | PR 블록 |
| {{COVERAGE_TOOL}} | 테스트 커버리지 | ≥ {{UNIT_COV}}% | PR 블록 |

## SLA (Service Level Agreement)

| 항목 | 목표 |
|------|------|
| 가용성 | {{AVAILABILITY}}% |
| API 응답 시간 P99 | ≤ {{RESP_P99}}ms |
| 장애 복구 시간 (RTO) | ≤ {{RTO}} |
| 데이터 복구 목표 (RPO) | ≤ {{RPO}} |

## 관련 문서

- [테스트 계획서](../05-testing/test-plan.md)
- [형상 관리 절차서](config-management.md)
