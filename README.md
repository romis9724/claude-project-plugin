# claude-project-plugin

Claude Code용 SI 프로젝트 산출물 생성 플러그인 마켓플레이스.

본 저장소는 **team-tools** Claude Code 마켓플레이스를 제공하며, 현재 1개 플러그인을 포함합니다:

- **`project`** — AI 에이전트 기반 SI 프로젝트 산출물 생성 (init/doc/bundle 3개 스킬, 106개 표준 문서 템플릿)

---

## 사용 시나리오

발주처는 표준 SI 산출물 형식을 요구하지만, 개발은 AI 에이전트와 자유롭게 진행. 이 플러그인은 **출력 형식 보장**(단계 워크플로우 강제 ❌)을 목표로 합니다.

- 새 프로젝트: `/project:init` — 초기 환경 일괄 설정
- 개별 문서: `/project:doc 인터페이스요구사항정의서` — 발주처 1건 요청 시
- 마일스톤 묶음: `/project:bundle 01-requirements` — 단계 납품 시 일괄 생성

---

## 설치 방법 (2가지)

### 방법 1: 프로젝트 자동 등록 (권장)

새 SI 프로젝트의 `.claude/settings.json`에 [`examples/project-settings.json`](examples/project-settings.json)의 내용을 복사. 프로젝트 클론 후 Claude Code 신뢰 시 자동으로 마켓플레이스 추가 + 플러그인 활성화됩니다.

```json
{
  "extraKnownMarketplaces": {
    "team-tools": {
      "source": {
        "source": "github",
        "repo": "romis9724/claude-project-plugin"
      }
    }
  },
  "enabledPlugins": {
    "project@team-tools": true
  }
}
```

팀원은:
```bash
git clone <your-project-repo>
cd <your-project-repo>
claude   # 신뢰 확인 → 자동 설치
> /project:init
```

### 방법 2: 수동 설치

```
/plugin marketplace add romis9724/claude-project-plugin
/plugin install project@team-tools
```

---

## 포함 플러그인

### `project` (v1.0.0)

| 스킬 | 호출 | 용도 |
|------|------|------|
| init | `/project:init` | 프로젝트 초기 환경 일괄 설정 (CLAUDE.md·docs 골격·핵심 7개 문서) |
| doc | `/project:doc <문서명>` | 발주처 요청 시 개별 산출물 1건 생성 |
| bundle | `/project:bundle <마일스톤>` | 마일스톤 단위 일괄 생성 + 일관성 점검 |

자세한 사용법: [`plugins/project/README.md`](plugins/project/README.md)

#### 산출물 카탈로그 (106개)

| 단계 | 디렉토리 | 문서 수 |
|------|---------|--------|
| 착수 | 00-kickoff | 12 |
| 요구분석 | 01-requirements | 11 |
| 아키텍처 | 02-architecture | 13 |
| 상세설계 | 03-design | 23 |
| 구현 | 04-implementation | 8 |
| 시험 | 05-testing | 4 |
| 관리 | 06-management | 19 |
| 인도 | 07-delivery | 16 |
| **합계** | | **106** |

전체 목록: [`plugins/project/reference/doc-catalog.md`](plugins/project/reference/doc-catalog.md)

---

## 디렉토리 구조

```
claude-project-plugin/
├── .claude-plugin/
│   └── marketplace.json          ← 마켓플레이스 매니페스트
├── plugins/
│   └── project/                  ← 플러그인
│       ├── .claude-plugin/plugin.json
│       ├── reference/
│       │   ├── methodology.md
│       │   └── doc-catalog.md
│       ├── skills/
│       │   ├── init/SKILL.md
│       │   ├── doc/SKILL.md
│       │   └── bundle/SKILL.md
│       ├── templates/            (106개 markdown 템플릿)
│       └── README.md
├── examples/
│   └── project-settings.json     ← 프로젝트 자동 등록 예시
├── README.md
└── LICENSE
```

---

## 업데이트

저장소에 새 commit이 push되면 팀원은:
```
/plugin update project@team-tools
```

---

## 라이선스

MIT — [`LICENSE`](LICENSE) 참조.

## 기여

이슈·PR 환영합니다. 새 산출물 템플릿 추가 시 `plugins/project/reference/doc-catalog.md`에도 매핑 반영 필수.
