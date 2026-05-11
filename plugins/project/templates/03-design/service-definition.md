---
title: 서비스 컴포넌트 정의서
project: "{{PROJECT_NAME_KR}}"
version: 0.1
status: 초안
date: "{{DATE}}"
owner: "{{ARCHITECT}}"
---

# 서비스 컴포넌트 정의서: {{PROJECT_NAME_KR}}

## 서비스 목록

| ID | 서비스명 | 기술 | 포트 | 역할 | 의존 서비스 |
|----|---------|------|------|------|------------|
| S01 | {{SVC_1}} | {{TECH_1}} | {{PORT_1}} | {{ROLE_1}} | {{DEP_1}} |
| S02 | {{SVC_2}} | {{TECH_2}} | {{PORT_2}} | {{ROLE_2}} | {{DEP_2}} |
| S03 | {{SVC_3}} | {{TECH_3}} | {{PORT_3}} | {{ROLE_3}} | {{DEP_3}} |

## 서비스 간 통신

| 호출자 | 피호출자 | 방식 | 프로토콜 | SLA |
|--------|---------|------|---------|-----|
| {{CALLER_1}} | {{CALLEE_1}} | 동기 | REST/HTTP | P95 < {{LATENCY_1}}ms |
| {{CALLER_2}} | {{CALLEE_2}} | 비동기 | {{MQ_PROTOCOL}} | - |

## 장애 격리 전략

| 패턴 | 적용 대상 | 설정 |
|------|----------|------|
| Circuit Breaker | 외부 API 호출 | 실패율 {{CB_THRESHOLD}}%, 복구 대기 {{CB_WAIT}}s |
| Retry | 일시적 오류 | 최대 {{RETRY_COUNT}}회, 지수 백오프 |
| Timeout | 모든 외부 호출 | {{TIMEOUT}}ms |
| Bulkhead | DB 연결 풀 | 최대 {{DB_POOL_SIZE}}개 |

## 헬스체크 엔드포인트

| 서비스 | 경로 | 주기 | 판단 기준 |
|--------|------|------|---------|
| {{SVC_1}} | `/health` | 30s | HTTP 200 |
| {{SVC_2}} | `/actuator/health` | 30s | HTTP 200 |

## 관련 문서

- [시스템 아키텍처 정의서](../02-architecture/system-architecture.md)
- [컴포넌트 명세서](component-spec.md)
