# dev-harness

주제만 주면 **메인 대화(PM)가 기획→설계→구현을 자동 오케스트레이션**하는 Claude Code 하네스 플러그인.

## 구성

- **에이전트 8종** (`agents/`): `product-planner`, `chief-architect`, `reviewer`, `tech-lead`, `frontend-dev`, `backend-dev`, `qa-engineer`, `devops-engineer`
- **스킬 2종** (`skills/`):
  - `/dev-harness:dev-orchestrator` — 기획→설계 파이프라인 (PM이 직접 오케스트레이션)
  - `/dev-harness:team-dev` — 설계 확정 후 개발팀 소집·구현

## 설치 (한 번만)

```bash
# 1) 로컬 마켓플레이스 등록 (이 디렉터리의 상위 = 마켓플레이스 루트)
claude plugin marketplace add https://github.com/SimJaeSugn/marketplace.git

# 2) 플러그인 설치 — --scope로 설치 위치 선택 (생략 시 user)
claude plugin install dev-harness@lvtlvt-harness                  # user(기본): 내 모든 프로젝트
claude plugin install dev-harness@lvtlvt-harness --scope project  # 이 저장소 팀 공유(.claude/settings.json, 커밋)
claude plugin install dev-harness@lvtlvt-harness --scope local    # 이 저장소에서 나만(.claude/settings.local.json)

# 3) 확인
claude plugin list
```

설치 후 아무 프로젝트에서나 에이전트(`product-planner` 등)와 스킬(`/dev-harness:dev-orchestrator`)이 노출된다.

### 설치 scope (`--scope`)

`--scope`로 플러그인을 어디에 설치(활성화)할지 고른다. **생략하면 user**다.

| scope | 명령 | 기록 위치 | 효과 |
|-------|------|-----------|------|
| **user** (기본) | `--scope user` | `~/.claude/settings.json` | 내 **모든 프로젝트**에서 사용 |
| **project** | `--scope project` | `.claude/settings.json` (git 커밋됨) | 이 저장소를 받는 **팀원 모두** 사용 |
| **local** | `--scope local` | `.claude/settings.local.json` (gitignore) | 이 저장소에서 **나만** 사용 |

> 설치하면 해당 scope의 `enabledPlugins`에 등록된다. 제거 없이 잠깐 끄려면
> `claude plugin disable dev-harness@lvtlvt-harness --scope <scope>`, 다시 켜려면 `claude plugin enable ...`.
> (관리자가 서버 설정으로 강제하는 `managed` scope도 있으나 사용자가 설치하는 대상은 아니다.)

## 사용

- **기획~설계 시작:** "이 제품 기획부터 설계까지 해줘", "PRD랑 아키텍처 만들어줘" → `dev-orchestrator` 스킬 자동 발동
- **구현 시작:** "설계 끝났으니 개발 시작", "개발팀 돌려줘" → `team-dev` 스킬 자동 발동
- **재실행/부분 수정:** "기획만 다시", "설계만 다시", "보완해줘"

산출물 경로: 기획 `docs/`, 설계 `docs/architecture/`, PM 산출물 `docs/pm/`, 검증 `docs/reviews/`.

## (선택) 프로젝트 CLAUDE.md 스니펫

플러그인의 트리거는 스킬 `description`으로 자동 발동되므로 별도 설정 없이 동작한다.
하지만 라우팅 규칙을 프로젝트에 **명시**하고 싶으면 대상 프로젝트의 `CLAUDE.md`에 아래를 추가한다
(플러그인 자체의 CLAUDE.md는 컨텍스트에 로드되지 않기 때문):

```markdown
## 하네스: 소프트웨어 개발 (dev-harness 플러그인)

- 기획·설계 요청("기획부터 설계까지", "PRD/아키텍처") → `dev-orchestrator` 스킬
- 설계 완료 후 구현 요청("개발 시작", "구현 진행") → `team-dev` 스킬
- 모든 산출 단계는 생성 → 검증(`reviewer`/`qa-engineer`) → 반영 루프를 거친다 (최대 2회 후 미해결은 사용자 보고).
```

## 업데이트 / 재배포

플러그인을 수정한 뒤 버전을 올리고 카탈로그를 갱신한다:

```bash
# plugin.json + marketplace.json 의 version 을 올린 뒤
claude plugin marketplace update
```

## 주의

- 에이전트/스킬은 순수 마크다운이라 어느 드라이브에서도 동작하지만, **실제 빌드·테스트 검증**은
  네이티브 의존성(pnpm install 등) 문제로 가상 드라이브(E:)가 아닌 `C:\dev` 등 실제 경로에서 실행해야 한다.
