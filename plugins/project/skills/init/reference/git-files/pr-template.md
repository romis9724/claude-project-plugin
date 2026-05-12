## 변경 컴포넌트 (해당 항목 체크)
- [ ] Backend
- [ ] User Frontend
- [ ] Admin Frontend
- [ ] Mobile
- [ ] Infra / CI
- [ ] Docs (docs/, CLAUDE.md)

> 두 개 이상의 컴포넌트가 동시 변경되면 본 PR이 **atomic**하게 묶여야 합니다.
> API 계약 변경(backend+frontend), 데이터 모델 변경(backend+mobile) 등이 해당.

## 요약

## 변경 이유

## 테스트
- [ ] 단위 테스트 추가/갱신 — 커버리지 {COVERAGE_TARGET} 유지
- [ ] E2E 영향 확인 ({TEST_E2E})
- [ ] Quality Gate 통과 확인 ({QUALITY_GATE})

## 보안 체크
- [ ] 외부 입력 검증 (OWASP)
- [ ] 비밀키·토큰 미노출
- [ ] 새 의존성 SCA 통과 ({STATIC_SCA})

## 산출물 영향
- 변경 시 갱신해야 할 docs/ 항목:
