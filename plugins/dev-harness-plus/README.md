# dev-harness-plus

주제만 주면 **메인 대화(PM)가 기획→설계→구현을 자동 오케스트레이션**하는 Claude Code 하네스 플러그인.
[`dev-harness`](../dev-harness)의 **강화판** — 전담 검증자와 품질 게이트를 더해 더 깐깐하게 만든다.

## dev-harness 대비 강화점

| 영역 | 강화 내용 |
|------|-----------|
| **새 에이전트 4종** | `security-reviewer`(보안 전담), `code-reviewer`(코드 품질 전담), `tech-writer`(문서 전담), `debugger`(난항 버그 근본원인) |
| **오케스트레이션** | Discovery 단계(greenfield/brownfield 분기·코드베이스 감사), **리스크 기반 게이팅**(경량/표준/엄격 트랙), **적대적 검증**(중대 결정 다수 표결) |
| **품질 게이트** | **추적성 매트릭스**(요구→설계→구현→테스트), **DoR 추가 + DoD 강화**(보안·코드품질·테스트 항목), **보안 게이트**(치명/높음은 진행 차단) |
| **프롬프트 품질** | 전 에이전트에 안티패턴·출력 계약·역할 경계 보강 |

## 구성

- **에이전트 12종** (`agents/`): `product-planner`, `chief-architect`, `reviewer`, `tech-lead`, `frontend-dev`, `backend-dev`, `qa-engineer`, `devops-engineer`, `security-reviewer`, `code-reviewer`, `tech-writer`, `debugger`
- **스킬 2종** (`skills/`):
  - `/dev-harness-plus:dev-orchestrator` — 기획→설계 파이프라인 (PM이 직접 오케스트레이션)
  - `/dev-harness-plus:team-dev` — 설계 확정 후 개발팀 소집·구현

## 설치 (한 번만)

```bash
# 1) 마켓플레이스 등록 (저장소를 직접 연결)
claude plugin marketplace add https://github.com/SimJaeSugn/marketplace.git

# 2) 플러그인 설치 (모든 프로젝트에서 사용 가능 — user scope 기본)
claude plugin install dev-harness-plus@lvtlvt-harness

# 3) 확인
claude plugin list
```

설치 후 아무 프로젝트에서나 에이전트(`product-planner` 등)와 스킬(`/dev-harness-plus:dev-orchestrator`)이 노출된다.

> **원본과의 공존 주의.** `dev-harness`와 `dev-harness-plus`를 **동시에 설치하면** 스킬 자동 발동
> 트리거가 겹쳐 어느 쪽이 뜰지 모호해진다. 둘 중 하나만 활성화해 쓰기를 권장한다.
> 두 플러그인의 에이전트/스킬은 `dev-harness:` / `dev-harness-plus:` 접두사로 구분된다.

## 사용

- **기획~설계 시작:** "이 제품 기획부터 설계까지 해줘", "PRD랑 아키텍처 만들어줘" → `dev-orchestrator` 스킬 자동 발동
- **구현 시작:** "설계 끝났으니 개발 시작", "개발팀 돌려줘" → `team-dev` 스킬 자동 발동
- **재실행/부분 수정:** "기획만 다시", "설계만 다시", "보완해줘"

산출물 경로: 기획 `docs/`, 설계 `docs/architecture/`, PM 산출물 `docs/pm/`, 검증 `docs/reviews/`, **추적성 매트릭스 `docs/traceability.md`**.

## (선택) 프로젝트 CLAUDE.md 스니펫

플러그인의 트리거는 스킬 `description`으로 자동 발동되므로 별도 설정 없이 동작한다.
하지만 라우팅 규칙을 프로젝트에 **명시**하고 싶으면 대상 프로젝트의 `CLAUDE.md`에 아래를 추가한다
(플러그인 자체의 CLAUDE.md는 컨텍스트에 로드되지 않기 때문):

```markdown
## 하네스: 소프트웨어 개발 (dev-harness-plus 플러그인)

- 기획·설계 요청("기획부터 설계까지", "PRD/아키텍처") → `dev-orchestrator` 스킬
- 설계 완료 후 구현 요청("개발 시작", "구현 진행") → `team-dev` 스킬
- 변경 리스크로 트랙을 고른다(경량/표준/엄격). 고위험(인증·결제·개인정보·외부연동)은 엄격 트랙 + 보안 게이트.
- 모든 산출 단계는 생성 → 검증(`reviewer`/`qa-engineer`/`code-reviewer`/`security-reviewer`) → 반영 루프를 거친다.
- 요구→설계→구현→테스트를 `docs/traceability.md` 한 표로 추적한다.
```

## 업데이트 / 재배포

플러그인을 수정한 뒤 버전을 올리고 카탈로그를 갱신한다:

```bash
# plugins/dev-harness-plus/.claude-plugin/plugin.json 의 version 을 올린 뒤
claude plugin marketplace update
```

> 버전은 `plugin.json` 한 곳에서만 관리한다(마켓플레이스 카탈로그에는 version을 두지 않는다).

## 주의

- 에이전트/스킬은 순수 마크다운이라 어느 드라이브에서도 동작하지만, **실제 빌드·테스트 검증**은
  네이티브 의존성(pnpm install 등) 문제로 가상 드라이브(E:)가 아닌 `C:\dev` 등 실제 경로에서 실행해야 한다.
