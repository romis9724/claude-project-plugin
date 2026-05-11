---
name: doc
description: >
  SI 표준 형식의 개별 산출물 문서를 현재 프로젝트 상태에서 생성한다.
  발주처가 특정 문서를 요청했을 때 즉시 1건 생성.
  입력: 한글 문서명(예: "인터페이스요구사항정의서") 또는 영문 파일명(예: "interface-requirements").
  트리거: "산출물 만들어줘", "doc generate", "인터페이스정의서 작성", "테스트설계서 만들어줘"
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
---

# /project:doc — 개별 산출물 문서 생성

발주처 요청 또는 내부 필요 시점에 SI 표준 형식의 문서 한 개를 현재 프로젝트 상태에서 생성한다.

---

## STEP 0 — 사전 확인

현재 디렉토리가 `/project:init`으로 초기화된 프로젝트인지 확인:
- `CLAUDE.md` 존재 여부 → 없으면 사용자에게 "init이 먼저 필요합니다" 안내 후 중단
- `docs/` 디렉토리 존재 여부

---

## STEP 1 — 입력 파싱

사용자가 제공한 문서명을 식별:
- 한글 입력(예: `인터페이스요구사항정의서`) → 카탈로그에서 영문 파일명 매칭
- 영문 입력(예: `interface-requirements`) → 직접 매칭
- 누락 시 AskUserQuestion으로 사용자에게 어떤 문서를 생성할지 확인

**카탈로그 위치**: 플러그인 디렉토리의 `reference/doc-catalog.md` (Read로 읽어 매칭)

---

## STEP 2 — 템플릿·가이드 로드

식별된 문서에 대해:
1. **템플릿 로드**: `{플러그인_경로}/templates/{milestone}/{file}.md` 읽기
   - 플러그인 경로 예: `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/project/`
2. **작성 가이드 로드**: `{플러그인_경로}/reference/methodology.md`에서 해당 문서 섹션만 추출

---

## STEP 3 — 프로젝트 상태 분석

문서 채움에 필요한 정보를 현재 프로젝트에서 추출:

| 소스 | 추출 정보 |
|------|----------|
| `CLAUDE.md` | 프로젝트명·목적·스택·기간·인증·LLM·보안요건 |
| `docs/00-kickoff/project-charter.md` | 이해관계자·범위·마일스톤 |
| `docs/01-requirements/requirements.md` | 기능/비기능 요구사항 |
| `docs/02-architecture/*.md` | 아키텍처 결정·컴포넌트·보안 모델 |
| `docs/03-design/*.md` | DB 설계·API 설계·컴포넌트 명세 |
| 소스 코드 (Glob/Grep) | 실제 구현된 모듈·클래스·API 엔드포인트 |

---

## STEP 4 — 문서 생성

다음 규칙으로 생성:

1. **YAML 프론트매터**: 템플릿 그대로 사용, 변수 치환
   - `{{PROJECT_NAME}}` → CLAUDE.md의 프로젝트명
   - `{{DATE}}` → 오늘 날짜
   - `{{OWNER}}` → CLAUDE.md에서 추출 또는 `TBD`

2. **섹션 채움 우선순위**:
   - **확신 정보**: 프로젝트 상태에서 직접 추출 → 그대로 작성
   - **추론 가능 정보**: 기존 문서·코드에서 합리적으로 도출 → 작성 + `> ⓘ {{source}} 기반 추론` 주석
   - **미입력 정보**: 사람만 알 수 있음 → `> ⚠️ TODO: {{설명}}` 표기

3. **표·다이어그램 처리**:
   - 표 컬럼 구조는 템플릿 유지
   - 행은 프로젝트 상태에서 추출하거나 `TBD` 행 1~2개로 채움
   - Mermaid 다이어그램은 추출된 컴포넌트·엔티티로 초안 생성

4. **6관점 점검**: methodology.md에 명시된 해당 문서의 6관점 체크포인트 적용

---

## STEP 5 — 저장 및 보고

저장 위치: `docs/{milestone}/{file}.md`
- 이미 존재 시 AskUserQuestion으로 사용자 확인 (덮어쓰기/스킵/이름 변경/병합)

생성 보고 출력:
```
✅ 생성 완료: docs/{milestone}/{file}.md

📊 정보 출처:
- 프로젝트 메타: CLAUDE.md
- 기능 요구사항: docs/01-requirements/requirements.md (5건 인용)
- 아키텍처 컴포넌트: docs/02-architecture/software-architecture.md (3건 인용)
- 실제 구현: src/api/ 디렉토리 (2건 추출)

⚠️ 사람 입력 필요 (TODO):
- {{TODO 항목 목록 — 보통 3~7개}}
```

---

## 사용 예시

```
사용자: /project:doc 인터페이스요구사항정의서
Claude:
  → reference/doc-catalog.md에서 매칭: interface-requirements.md (01-requirements)
  → templates/01-requirements/interface-requirements.md 로드
  → CLAUDE.md + 기존 docs/ + src/ 분석
  → docs/01-requirements/interface-requirements.md 생성
  → 보고
```

```
사용자: "외부 API 연동 문서 만들어줘"
Claude:
  → 의도 파악: 인터페이스 관련 문서
  → AskUserQuestion: "인터페이스요구사항정의서 / 인터페이스명세서 중 어떤 것?"
  → 사용자 선택 후 위와 동일하게 진행
```
