---
title: 컴포넌트 명세서
project: "{{PROJECT_NAME_KR}}"
version: 0.1
status: 초안
date: "{{DATE}}"
owner: "{{ARCHITECT}}"
---

# 컴포넌트 명세서: {{PROJECT_NAME_KR}}

## 컴포넌트 목록

| ID | 컴포넌트명 | 책임 | 기술 | 의존 컴포넌트 |
|----|----------|------|------|--------------|
| C01 | {{COMP_1}} | {{RESP_1}} | {{TECH_1}} | {{DEP_1}} |
| C02 | {{COMP_2}} | {{RESP_2}} | {{TECH_2}} | {{DEP_2}} |
| C03 | {{COMP_3}} | {{RESP_3}} | {{TECH_3}} | {{DEP_3}} |

## 인터페이스 명세

### {{COMP_1}}

| 메서드/엔드포인트 | 입력 | 출력 | 사전조건 | 사후조건 |
|----------------|------|------|---------|---------|
| {{METHOD_1}} | {{INPUT_1}} | {{OUTPUT_1}} | {{PRE_1}} | {{POST_1}} |
| {{METHOD_2}} | {{INPUT_2}} | {{OUTPUT_2}} | {{PRE_2}} | {{POST_2}} |

### {{COMP_2}}

| 메서드/엔드포인트 | 입력 | 출력 | 사전조건 | 사후조건 |
|----------------|------|------|---------|---------|
| {{METHOD_3}} | {{INPUT_3}} | {{OUTPUT_3}} | {{PRE_3}} | {{POST_3}} |

## 에러 처리

| 에러 코드 | 상황 | 메시지 | 처리 방법 |
|----------|------|--------|----------|
| E001 | 인증 실패 | Unauthorized | 401 반환, 로그 기록 |
| E002 | 권한 없음 | Forbidden | 403 반환 |
| E003 | 리소스 없음 | Not Found | 404 반환 |
| E500 | 내부 오류 | Internal Server Error | 500 반환, 알림 발송 |
| {{ECODE}} | {{ESIT}} | {{EMSG}} | {{EHANDLING}} |

## 관련 문서

- [SW 아키텍처 기술서](../02-architecture/software-architecture.md)
- [API 설계서](api-design.md)
- [DB 설계서](db-design.md)
