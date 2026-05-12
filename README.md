# claude-project-plugin

Claude Code용 SI 프로젝트 산출물 생성 플러그인 마켓플레이스.

본 저장소는 **team-tools** Claude Code 마켓플레이스를 제공하며, 현재 1개 플러그인을 포함합니다:

- **`si-project`** (v2.0.0) — AI 에이전트 기반 SI 프로젝트 산출물 생성. 5개 스킬, 106개 표준 문서 템플릿, GHA workflow·pre-commit·SessionStart hook 자동 셋업, lazy loading, 팀 lesson 공유.

> v2.0.0 (2026-05) — `project` → `si-project` namespace 변경 + 5개 스킬 이름 변경 (breaking change). v1.x에서 업그레이드 시 재설치 필요.

---

## 사용 시나리오

발주처는 표준 SI 산출물 형식을 요구하지만, 개발은 AI 에이전트와 자유롭게 진행. 이 플러그인은 **출력 형식 보장**(단계 워크플로우 강제 ❌)을 목표로 합니다.

- 새 프로젝트: `/si-project:project-setup` — 초기 환경 일괄 설정 (CLAUDE.md, settings, git, pre-commit, GHA, ADR-0001)
- 개별 문서: `/si-project:project-document 인터페이스요구사항정의서` — 발주처 1건 요청 시
- 마일스톤 묶음: `/si-project:project-milestone 01-requirements` — 단계 납품 시 일괄 생성
- 현황 요약: `/si-project:project-summary` — 30줄 5섹션 요약 (현황·할 일)
- 아키텍처 결정 기록: `/si-project:project-adr "PostgreSQL 채택"` — MADR 형식 결정 기록

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
    "si-project@team-tools": true
  }
}
```

팀원은:
```bash
git clone <your-project-repo>
cd <your-project-repo>
claude   # 신뢰 확인 → 자동 설치
> /si-project:project-setup
```

### 방법 2: 수동 설치

```
/plugin marketplace add romis9724/claude-project-plugin
/plugin install si-project@team-tools
```

---

## 포함 플러그인

### `si-project` (v2.0.0)

| 스킬 | 호출 | 용도 |
|------|------|------|
| project-setup | `/si-project:project-setup` | 신규 SI 프로젝트 초기 환경 일괄 설정 (CLAUDE.md·docs·git 자동화·CI·ADR-0001) |
| project-document | `/si-project:project-document <문서명>` | 발주처 요청 시 개별 산출물 1건 생성 |
| project-milestone | `/si-project:project-milestone <마일스톤>` | 마일스톤 단위 일괄 생성 + 일관성 점검 |
| project-summary | `/si-project:project-summary` | 30줄 5섹션 현황 요약 (읽기 전용) |
| project-adr | `/si-project:project-adr <제목>` | MADR 형식 아키텍처 결정 기록 |

자세한 사용법: [`plugins/si-project/README.md`](plugins/si-project/README.md)

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

전체 목록: [`plugins/si-project/reference/doc-catalog.md`](plugins/si-project/reference/doc-catalog.md)

---

## 디렉토리 구조

```
claude-project-plugin/
├── .claude-plugin/
│   └── marketplace.json          ← 마켓플레이스 매니페스트 (team-tools)
├── plugins/
│   └── si-project/               ← 플러그인 (v2.0.0)
│       ├── .claude-plugin/plugin.json
│       ├── reference/
│       │   ├── methodology.md
│       │   └── doc-catalog.md
│       ├── skills/
│       │   ├── project-setup/
│       │   │   ├── SKILL.md      ← 283줄 (인라인 템플릿은 reference/로 분리)
│       │   │   └── reference/    ← questions, claude-md-template, settings, git-files/, project-files/, adr/, ci-workflows/
│       │   ├── project-document/SKILL.md
│       │   ├── project-milestone/SKILL.md
│       │   ├── project-summary/SKILL.md
│       │   └── project-adr/SKILL.md
│       ├── templates/            (106개 markdown 템플릿)
│       └── README.md
├── examples/
│   └── project-settings.json     ← 프로젝트 자동 등록 예시
├── README.md
└── LICENSE
```

---

## v1.x → v2.0.0 마이그레이션

v1.x를 쓰던 사용자는:

1. `~/.claude/settings.json`의 `enabledPlugins`에서 `project@team-tools: true` 제거
2. `~/.claude/plugins/installed_plugins.json`에서 `project@team-tools` 항목 제거
3. 캐시 디렉토리 `~/.claude/plugins/cache/team-tools/project/` 삭제
4. 마켓플레이스 재등록: `/plugin marketplace remove team-tools` → `/plugin marketplace add romis9724/claude-project-plugin`
5. 새 플러그인 활성화: `/plugin install si-project@team-tools`
6. 새 명령어 사용:
   - `/project:init` → `/si-project:project-setup`
   - `/project:doc` → `/si-project:project-document`
   - `/project:bundle` → `/si-project:project-milestone`
   - `/project:status` → `/si-project:project-summary`
   - `/project:adr` → `/si-project:project-adr`

기존 프로젝트의 `.claude/settings.json`에 `enabledPlugins.project@team-tools`가 있으면 `si-project@team-tools`로 키 교체 필요.

---

## 업데이트

저장소에 새 commit이 push되면 팀원은:
```
/plugin update si-project@team-tools
```

`autoUpdatesChannel: "latest"` 설정 시 자동.

---

## 라이선스

MIT — [`LICENSE`](LICENSE) 참조.

## 기여

이슈·PR 환영합니다. 새 산출물 템플릿 추가 시 `plugins/si-project/reference/doc-catalog.md`에도 매핑 반영 필수.
