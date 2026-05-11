---
name: bundle
description: >
  마일스톤(단계)의 필수 산출물을 일괄 생성하고 일관성을 점검한다.
  발주처 단계별 납품 시점에 호출.
  입력: 마일스톤 식별자(00-kickoff ~ 07-delivery) 또는 한글(요구분석, 아키텍처 등).
  트리거: "마일스톤 산출물", "단계 납품물 생성", "요구분석 산출물 일괄", "아키텍처 번들"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(ls *)
  - Bash(Get-ChildItem *)
  - AskUserQuestion
  - TodoWrite
---

# /project:bundle — 마일스톤 산출물 번들 생성

특정 마일스톤에 발주처가 요구하는 필수 문서들을 일괄 생성하고 문서간 일관성을 보장한다.

---

## STEP 0 — 사전 확인

- `CLAUDE.md` 존재 여부 → 없으면 `/project:init` 안내 후 중단
- 사용자 입력 마일스톤 식별

**마일스톤 매핑**:
| 입력 | 정규화 |
|------|--------|
| `00-kickoff`, `착수`, `kickoff` | 00-kickoff |
| `01-requirements`, `요구분석`, `requirements` | 01-requirements |
| `02-architecture`, `아키텍처`, `architecture` | 02-architecture |
| `03-design`, `설계`, `design` | 03-design |
| `04-implementation`, `구현`, `implementation` | 04-implementation |
| `05-testing`, `테스트`, `testing` | 05-testing |
| `06-management`, `관리`, `management` | 06-management |
| `07-delivery`, `인도`, `이관`, `delivery` | 07-delivery |

---

## STEP 1 — 필수 문서 목록 추출

`{플러그인_경로}/reference/doc-catalog.md` 읽고:
- 해당 마일스톤의 모든 문서 추출
- `필수도 = 필수` 항목 우선 (필요 시 `권장` 포함 여부 사용자 확인)

TodoWrite로 각 문서를 task로 등록 (진행 추적).

---

## STEP 2 — 기존 문서 점검

각 필수 문서에 대해:
- `docs/{milestone}/{file}.md` 존재 여부 확인
- 기존 문서 발견 시 AskUserQuestion으로 일괄 정책 결정:
  - 모두 덮어쓰기
  - 모두 스킵 (기존 유지)
  - 모두 병합 (기존 + 신규 정보)
  - 개별 확인 (각 파일마다 물음)

---

## STEP 3 — 일괄 생성

각 누락 문서에 대해 `doc` 스킬과 동일한 로직으로 생성:
1. 템플릿 로드
2. methodology.md에서 작성 가이드 로드
3. 프로젝트 상태 분석
4. 생성 (YAML 프론트매터 + 섹션 채움 + TODO 표기)
5. 저장 + TodoWrite 완료 표시

**일관성을 위한 공통 컨텍스트**:
- 한 번에 모든 문서를 생성하므로 공통 변수 캐시 사용
  - 프로젝트명·스택·이해관계자·핵심 컴포넌트 등은 1회만 분석 후 모든 문서에 동일하게 적용

---

## STEP 4 — 일관성 점검

전체 생성 완료 후 다음을 검증:

### 4-1. 용어 통일
- 모든 신규 문서에서 사용된 주요 명사 추출
- 동의어·이형 검출 (예: `사용자` vs `이용자`, `시스템` vs `서비스`)
- 불일치 시 보고

### 4-2. 크로스 레퍼런스
- 문서간 참조 링크 (`[XXX](../YY-zz/file.md)`) 유효성 확인
- 깨진 링크 보고

### 4-3. 6관점 점검
methodology.md의 해당 마일스톤 6관점 체크리스트 적용:

| 관점 | 점검 항목 (예시 — 01-requirements) |
|------|----------------------------------|
| PMO | 범위·일정·예산 영향이 모든 문서에 반영 |
| PM | 이해관계자 요구가 추적 가능하게 기록 |
| 아키텍처 | NFR이 아키텍처 결정에 영향 줄 수 있도록 명확 |
| DBA | 데이터 관련 요구가 식별됨 |
| 하네스 | CI/CD·환경 관련 NFR 명시 |
| 보안 | 보안 요건·인증·개인정보 처리 명시 |

---

## STEP 5 — 완료 보고

```
✅ {{milestone}} 마일스톤 산출물 생성 완료

📁 생성된 문서 ({{N}}건):
- docs/{{milestone}}/{{file1}}.md ← 신규
- docs/{{milestone}}/{{file2}}.md ← 덮어쓰기
- docs/{{milestone}}/{{file3}}.md ← 스킵 (기존 유지)
...

⚠️ 사람 입력 필요 (TODO 통합 리스트):
- {{file1}}.md: 이해관계자 연락처
- {{file2}}.md: 발주처 SLA 수치
- {{file3}}.md: 인수기준 문구
...

🔍 일관성 점검:
- 용어 통일: ✅ 통과 / ⚠️ 불일치 N건
- 크로스 레퍼런스: ✅ 통과 / ❌ 깨진 링크 N건
- 6관점 점검: ⚠️ 보안 관점 보강 필요 (X.md, Y.md)
```

---

## 사용 예시

```
사용자: /project:bundle 01-requirements
Claude:
  → reference/doc-catalog.md에서 01-requirements 필수 문서 5개 추출
  → 기존 문서 점검: requirements.md 있음, 나머지 4개 없음
  → AskUserQuestion: 기존 requirements.md 정책?
  → 사용자: "병합"
  → 4개 신규 생성 + 1개 병합
  → 일관성 점검
  → 완료 보고
```
