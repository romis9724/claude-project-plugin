---
title: 테스트 설계서
project: {KR_NAME}
version: 0.1
status: 초안
---

# 테스트 설계서: {KR_NAME}

## 테스트 케이스 목록

| ID | 대상 | 시나리오 | 입력 | 기대 결과 | 실행 결과 |
|----|------|---------|------|----------|----------|
| TC-001 | | | | | |

## 경계값·예외 시나리오
- TODO

## 보안 테스트 케이스 (OWASP Top 10 기반)
| OWASP | 항목 | 테스트 방법 | 통과 기준 |
|-------|------|-----------|---------|
| A01 | Broken Access Control | 권한 없는 리소스 접근 시도 | 403 반환 |
| A02 | Cryptographic Failures | 민감정보 암호화 검증 | 평문 저장 없음 |
| A03 | Injection | SQL/Command Injection 시도 | 차단 및 로그 기록 |
