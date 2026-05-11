---
title: API 설계서
project: "{{PROJECT_NAME_KR}}"
version: 0.1
status: 초안
date: "{{DATE}}"
owner: "{{ARCHITECT}}"
---

# API 설계서: {{PROJECT_NAME_KR}}

## API 설계 원칙

- **스타일**: {{API_STYLE}} (REST / GraphQL / gRPC)
- **버전 관리**: URL 경로 방식 (`/api/v1/`) 또는 헤더 방식
- **응답 포맷**: JSON (UTF-8)
- **인증**: {{AUTH_METHOD}} (JWT / API Key / OAuth2)
- **HTTPS 전용**: 모든 엔드포인트 TLS 필수

## 공통 규격

### 요청 헤더
```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
X-Request-ID: {uuid}
```

### 공통 응답 구조
```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": { "requestId": "", "timestamp": "" }
}
```

### 공통 에러 코드
| 코드 | HTTP 상태 | 설명 |
|------|-----------|------|
| AUTH_001 | 401 | 인증 토큰 없음/만료 |
| AUTH_002 | 403 | 권한 없음 |
| VAL_001 | 400 | 입력값 유효성 오류 |
| NOT_FOUND | 404 | 리소스 없음 |
| SERVER_ERR | 500 | 내부 서버 오류 |

### 페이지네이션
```
GET /api/v1/{resource}?page=1&size=20&sort=createdAt,desc
```

## 엔드포인트 목록

| Method | Path | 설명 | 인증 | 요청 바디 | 응답 |
|--------|------|------|------|----------|------|
| GET | `/api/v1/{{RESOURCE_1}}` | {{DESC_1}} | 필요 | - | {{RESOURCE_1}}[] |
| POST | `/api/v1/{{RESOURCE_1}}` | {{DESC_2}} | 필요 | {{REQUEST_BODY_1}} | {{RESOURCE_1}} |
| GET | `/api/v1/{{RESOURCE_1}}/{id}` | {{DESC_3}} | 필요 | - | {{RESOURCE_1}} |
| PUT | `/api/v1/{{RESOURCE_1}}/{id}` | {{DESC_4}} | 필요 | {{REQUEST_BODY_2}} | {{RESOURCE_1}} |
| DELETE | `/api/v1/{{RESOURCE_1}}/{id}` | {{DESC_5}} | 필요 | - | 204 |

## 보안 헤더

| 헤더 | 값 | 목적 |
|------|-----|------|
| `Strict-Transport-Security` | `max-age=31536000` | HTTPS 강제 |
| `X-Content-Type-Options` | `nosniff` | MIME 스니핑 방지 |
| `X-Frame-Options` | `DENY` | Clickjacking 방지 |
| `Content-Security-Policy` | `default-src 'self'` | XSS 방지 |

## Rate Limiting

| 대상 | 제한 | 초과 시 |
|------|------|--------|
| 인증된 사용자 | {{RATE_AUTH}} req/min | 429 Too Many Requests |
| 비인증 IP | {{RATE_ANON}} req/min | 429 Too Many Requests |

## 관련 문서

- [보안 정의서](../02-architecture/security-definition.md)
- [컴포넌트 명세서](component-spec.md)
