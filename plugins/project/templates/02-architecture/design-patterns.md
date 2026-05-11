---
title: 설계 패턴 기술서
project: "{{PROJECT_NAME_KR}}"
version: 0.1
status: 초안
date: "{{DATE}}"
owner: "{{ARCHITECT}}"
---

# 설계 패턴 기술서: {{PROJECT_NAME_KR}}

## 적용 패턴 목록

| 패턴 | 카테고리 | 적용 위치 | 목적 |
|------|---------|----------|------|
| Repository | 구조 | 데이터 접근 계층 | DB 추상화, 테스트 용이성 |
| Factory | 생성 | 객체 생성 로직 | 객체 생성 캡슐화 |
| Strategy | 행위 | 알고리즘 교체 지점 | 런타임 전략 교체 |
| Observer/Event | 행위 | 비동기 이벤트 처리 | 느슨한 결합 |
| Circuit Breaker | 복원력 | 외부 API 호출 | 장애 전파 방지 |
| {{PATTERN_NAME}} | {{CATEGORY}} | {{LOCATION}} | {{PURPOSE}} |

## 패턴별 구현 가이드

### Repository 패턴
```
interface {{Entity}}Repository:
  findById(id) → {{Entity}}
  save(entity) → {{Entity}}
  delete(id) → void
  findAll(filter) → List<{{Entity}}>
```
- 구현체: `{{Entity}}RepositoryImpl` (DB), `{{Entity}}MockRepository` (테스트)

### Circuit Breaker
- 라이브러리: {{CIRCUIT_BREAKER_LIB}} (예: resilience4j, hystrix)
- 임계값: 실패율 {{FAILURE_THRESHOLD}}%, 대기 시간 {{WAIT_DURATION}}ms

## 금지 패턴 (Anti-Patterns)

| 패턴 | 이유 | 대안 |
|------|------|------|
| God Object | 단일 클래스 과도한 책임 | 책임 분리 (SRP) |
| Magic Numbers | 유지보수 어려움 | 상수 또는 설정 파일 |
| Singleton (남용) | 테스트 어려움, 전역 상태 | 의존성 주입 |
| 직접 new 생성 (서비스 내) | 테스트 불가 | Factory/DI 사용 |

## 관련 문서

- [SW 아키텍처 기술서](software-architecture.md)
- [컴포넌트 명세서](../03-design/component-spec.md)
