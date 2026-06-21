# dev-harness-structured-v1

주제만 주면 **메인 대화(PM)가 기획→설계→구현을 자동 오케스트레이션**하는 Claude Code 하네스 플러그인.
[`dev-harness-plus`](../dev-harness-plus)를 기반으로, **산출물의 구조(디렉토리·파일명·명명 규칙·섹션 구성)를
단일 출처로 고정**해 **어떤 프로젝트에서든 동일한 구조의 산출물**이 나오도록 만든 버전이다.

## 왜 이 버전인가 (dev-harness-plus 대비)

dev-harness / dev-harness-plus는 산출물의 *유형*(PRD·설계서·검증 보고 등)만 제시했을 뿐, **정확한 파일명·
디렉토리·섹션 구조가 명시돼 있지 않아 실행할 때마다 구조가 달라졌다.** 이 버전은 그 구조를 **고정**한다.

| 영역 | 고정 내용 |
|------|-----------|
| **디렉토리·파일** | `docs/PRD.html`·`FeatureSpec.html`·`UserStories.html`·`roadmap.html`·`traceability.html`·`api-contract.md`, `docs/architecture/{architecture,adr}.html`, `docs/reviews/*`, `docs/pm/*` 로 **고정** |
| **검증 보고 명명** | 설계/문서: `{planning\|roadmap\|architecture}_review_{n}.html`, 설계 보안: `architecture_security_review_{n}.html`, 구현(스프린트): `{code_review\|qa\|security_review}_s{N}_{n}.html` |
| **고정 규칙** | ① **`docs/api-contract.md`만 마크다운**(FE↔BE 단일 출처), 나머지는 전부 HTML  ② **ADR은 `adr.html` 한 문서에 `ADR-001`… 누적**(결정마다 파일 분리 금지) |
| **섹션 구조** | 각 산출물의 표준 섹션 순서·표 컬럼·SVG/배지/카드 사용을 에이전트 정의에 명시 |

> 구조 규칙의 **단일 출처**는 `dev-orchestrator` 스킬의 **"산출물 구조 명세 (고정)"** 절이며, 각 에이전트는
> 자기 산출물의 정확한 파일 경로·섹션을 함께 명시한다. (dev-harness-plus의 모든 강화 기능 — Discovery·리스크
> 게이팅·추적성 매트릭스·적대적 검증·보안 게이트·DoR/DoD·변경 이력·파트 기준 문서 — 은 그대로 유지된다.)

## 구성

- **에이전트 12종** (`agents/`): `product-planner`, `chief-architect`, `reviewer`, `tech-lead`, `frontend-dev`, `backend-dev`, `qa-engineer`, `devops-engineer`, `security-reviewer`, `code-reviewer`, `tech-writer`, `debugger`
- **스킬 2종** (`skills/`):
  - `/dev-harness-structured-v1:dev-orchestrator` — 기획→설계 파이프라인 (PM이 직접 오케스트레이션) + **산출물 구조 명세(단일 출처)**
  - `/dev-harness-structured-v1:team-dev` — 설계 확정 후 개발팀 소집·구현

## 산출물 구조 (고정 — 모든 프로젝트 동일) ★

```
docs/
├── PRD.html                  # 제품 요구사항 문서          ← product-planner
├── FeatureSpec.html          # 기능 명세서 (F-1, F-2…)     ← product-planner
├── UserStories.html          # 사용자 스토리 백로그(에픽별)  ← product-planner
├── roadmap.html              # 개발 로드맵 (S0..Sn)        ← tech-lead (구현)
├── api-contract.md           # 확정 API 계약 — 유일한 .md  ← tech-lead 중재 / backend-dev (구현)
├── traceability.html         # 추적성 매트릭스             ← PM
├── architecture/
│   ├── architecture.html     # 시스템 아키텍처 설계서       ← chief-architect
│   └── adr.html              # ADR 모음(ADR-001… 누적)     ← chief-architect
├── reviews/                  # 모든 검증 보고
│   ├── planning_review_{n}.html
│   ├── roadmap_review_{n}.html
│   ├── architecture_review_{n}.html
│   ├── architecture_security_review_{n}.html
│   ├── code_review_s{N}_{n}.html
│   ├── qa_s{N}_{n}.html
│   └── security_review_s{N}_{n}.html
└── pm/                       # PM 산출물
    ├── progress.html         # 통합 진행 현황판 — 단일 진입점(항상 생성·유지) ★
    ├── codebase-audit.html   # brownfield 감사 (해당 상황에만)
    ├── change-log.html       # 변경 이력(기존 산출물 변경 시)
    └── meeting-{주제}.html    # 회의록
```

- `{n}` = 적대적/반영 회차(`_1`,`_2`…), `s{N}` = 로드맵 스프린트 번호(`s0`,`s1`…).
- 입력 기준 폴더(`docs/Templates/`·`Planning/`·`Design/`·`Development/`)는 *산출물*과 별개의 *입력 룰* 폴더다.

## 통합 진행 현황판 (`docs/pm/progress.html`) ★

각 산출물은 자기 단계의 진행만 보여줄 뿐, **기획→설계→구현→검증 전체 흐름을 한눈에 보는 통합 현황**은
흩어져 있어 파악이 어렵다. 이 플러그인은 **`docs/pm/progress.html`** 를 **항상 생성·유지하는 단일 진입점**으로
두어, 전체 파이프라인과 각 단계 상태·검증 게이트·산출물을 **한 페이지**로 모은다(추적성 매트릭스와 동격 1급 산출물).

**고정 섹션** — ① 헤더(현재 단계·다음·트랙) ② 핵심 지표(KPI, 게이트 기준값) ③ 파이프라인 SVG(기획→설계→구현→검증 노드 상태)
④ 구현 단계별 현황(로드맵 S0~Sn) ⑤ 검증 게이트 이력(PASS/FAIL·회차·보고서 링크) ⑥ 요구 진척(추적성 요약)
⑦ 남은 일·리스크 ⑧ 산출물 인덱스(모든 산출물 링크).

**유지 규칙(슬림)** — 현황판은 **파생물**이라 원천(`reviews/`·`traceability.html`·`roadmap.html`)을 복제하지 않고
**요약 한 줄 + 링크**로 가리킨다. 갱신은 **마일스톤 단위로만**(기획 PASS 시 최초 생성 → 설계 확정 · 스프린트 완료 ·
방향 전환 회의 · 최종 보고). 매 게이트·매 추적성 수정마다 손대지 않는다(수기 동기화 = 드리프트의 원인). 기준은
`dev-orchestrator` 스킬의 "통합 진행 현황판 (progress.html)" 절이 단일 출처이고, 구현 단계 갱신은 `team-dev` 스킬이
이어받는다. 스켈레톤 템플릿: [`templates/progress-dashboard.html`](templates/progress-dashboard.html)
(`docs/Templates/progress.html`가 있으면 그쪽 우선).

## 설치 (한 번만)

```bash
# 1) 마켓플레이스 등록 (저장소를 직접 연결)
claude plugin marketplace add https://github.com/SimJaeSugn/marketplace.git

# 2) 플러그인 설치 — --scope로 설치 위치 선택 (생략 시 user)
claude plugin install dev-harness-structured-v1@lvtlvt-harness                  # user(기본): 내 모든 프로젝트
claude plugin install dev-harness-structured-v1@lvtlvt-harness --scope project  # 이 저장소 팀 공유(.claude/settings.json, 커밋)
claude plugin install dev-harness-structured-v1@lvtlvt-harness --scope local    # 이 저장소에서 나만(.claude/settings.local.json)

# 3) 확인
claude plugin list
```

설치 후 아무 프로젝트에서나 에이전트(`product-planner` 등)와 스킬(`/dev-harness-structured-v1:dev-orchestrator`)이 노출된다.

### 설치 scope (`--scope`)

`--scope`로 플러그인을 어디에 설치(활성화)할지 고른다. **생략하면 user**다.

| scope | 명령 | 기록 위치 | 효과 |
|-------|------|-----------|------|
| **user** (기본) | `--scope user` | `~/.claude/settings.json` | 내 **모든 프로젝트**에서 사용 |
| **project** | `--scope project` | `.claude/settings.json` (git 커밋됨) | 이 저장소를 받는 **팀원 모두** 사용 |
| **local** | `--scope local` | `.claude/settings.local.json` (gitignore) | 이 저장소에서 **나만** 사용 |

> 설치하면 해당 scope의 `enabledPlugins`에 등록된다. 제거 없이 잠깐 끄려면
> `claude plugin disable dev-harness-structured-v1@lvtlvt-harness --scope <scope>`, 다시 켜려면 `claude plugin enable ...`.
> (관리자가 서버 설정으로 강제하는 `managed` scope도 있으나 사용자가 설치하는 대상은 아니다.)

> **공존 주의.** `dev-harness` / `dev-harness-plus` / `dev-harness-structured-v1`를 **동시에 설치하면** 스킬 자동 발동
> 트리거가 겹쳐 어느 쪽이 뜰지 모호해진다. 하나만 활성화해 쓰기를 권장한다.
> 세 플러그인의 에이전트/스킬은 `dev-harness:` / `dev-harness-plus:` / `dev-harness-structured-v1:` 접두사로 구분된다.

## 사용

- **기획~설계 시작:** "이 제품 기획부터 설계까지 해줘", "PRD랑 아키텍처 만들어줘" → `dev-orchestrator` 스킬 자동 발동
- **구현 시작:** "설계 끝났으니 개발 시작", "개발팀 돌려줘" → `team-dev` 스킬 자동 발동
- **재실행/부분 수정:** "기획만 다시", "설계만 다시", "보완해줘"

## 산출물 형식 — HTML & 템플릿 (SVG 시각화)

**모든 문서 산출물은 자족형(self-contained) `.html`** 로 생성된다(소스 코드 제외). **유일한 예외는 `docs/api-contract.md`**
(FE↔BE 코드 대조용 단일 출처라 마크다운으로 유지).

**스타일 결정 순서**
1. **사용자 템플릿 우선** — 프로젝트의 **`docs/Templates/`** 에 해당 유형 템플릿(`*.html`)이 있으면 그 구조·스타일을 그대로 따른다.
   - 예: `docs/Templates/prd.html`, `docs/Templates/architecture.html`, 공통 베이스 `docs/Templates/_base.html`
2. **없으면 표준 스타일** — **색상 팔레트의 단일 출처는 [`templates/artifact-base.html`](templates/artifact-base.html)의 `:root` CSS 변수**다
   (다크 테마·시스템 폰트 `"Segoe UI","Malgun Gothic"`·헤더+카드/표·**인라인 CSS**, 외부 의존 없음).
   색상 hex 값을 산출물·에이전트 지시마다 다시 나열하지 말고 이 베이스 한 곳을 참조한다.

**시각 요소 규칙** — 구조·흐름·타임라인·신뢰경계는 **`<svg>`**, 항목 나열은 **`<table>`**, 판정/등급/우선순위는
**색상 배지**(PASS/FAIL·P1/P2/P3·치명/높음/중간/낮음), 정량 요약은 **카드/스코어카드**로 표현한다.

**베이스 템플릿 제공** — 표준 스타일의 참고 스켈레톤이 동봉돼 있다: [`templates/artifact-base.html`](templates/artifact-base.html).
이 파일을 프로젝트의 `docs/Templates/_base.html`로 복사해 커스터마이즈하면, 이후 모든 산출물이 그 템플릿을 따른다.
통합 진행 현황판 전용 스켈레톤도 동봉돼 있다: [`templates/progress-dashboard.html`](templates/progress-dashboard.html)
(→ `docs/pm/progress.html`로 산출. 커스터마이즈하려면 `docs/Templates/progress.html`로 복사).

## 파트 기준 문서 (입력 룰·참고자료)

각 파트는 작업 전에 **자기 파트의 기준 폴더**를 읽어 그 안의 룰·참고자료·가이드를 **준수 기준**으로 삼는다.
**폴더가 없으면 외부 기준 없이 스스로 진행**한다(중단하지 않음).

| 파트 | 기준 폴더 | 넣을 만한 것 |
|------|-----------|--------------|
| 기획 | `docs/Planning/` | 기획 템플릿·정책·브랜드 톤·용어·우선순위 기준 |
| 디자인 | `docs/Design/` | 설계 원칙·표준 아키텍처/패턴·디자인 시스템·UI 가이드·네이밍 컨벤션 |
| 개발 | `docs/Development/` | 코딩 표준·린트/테스트 정책·보안 룰·디렉토리/빌드 컨벤션 |

> 이 폴더들은 **입력 기준 문서**다. 산출물 저장 경로(위 "산출물 구조")와는 별개다.

## (선택) 프로젝트 CLAUDE.md 스니펫

```markdown
## 하네스: 소프트웨어 개발 (dev-harness-structured-v1 플러그인)

- 기획·설계 요청("기획부터 설계까지", "PRD/아키텍처") → `dev-orchestrator` 스킬
- 설계 완료 후 구현 요청("개발 시작", "구현 진행") → `team-dev` 스킬
- 산출물은 고정 구조를 따른다(파일명·디렉토리·명명 규칙 = `dev-orchestrator`의 "산출물 구조 명세").
- 변경 리스크로 트랙을 고른다(경량/표준/엄격). 고위험(인증·결제·개인정보·외부연동)은 엄격 트랙 + 보안 게이트.
- 모든 산출 단계는 생성 → 검증(`reviewer`/`qa-engineer`/`code-reviewer`/`security-reviewer`) → 반영 루프를 거친다.
- 요구→설계→구현→테스트를 `docs/traceability.html` 한 표로 추적한다.
- 전체 진행은 `docs/pm/progress.html`(통합 현황판·단일 진입점)을 유지한다 — 원천을 복제하지 말고 링크, 갱신은 마일스톤 단위(설계 확정·스프린트 완료·최종 보고).
```

## 업데이트 / 재배포

```bash
# plugins/dev-harness-structured-v1/.claude-plugin/plugin.json 의 version 을 올린 뒤
claude plugin marketplace update
```

> 버전은 `plugin.json` 한 곳에서만 관리한다(마켓플레이스 카탈로그에는 version을 두지 않는다).

## 주의

- 에이전트/스킬은 순수 마크다운이라 어느 드라이브에서도 동작하지만, **실제 빌드·테스트 검증**은
  네이티브 의존성(pnpm install 등) 문제로 가상 드라이브(E:)가 아닌 `C:\dev` 등 실제 경로에서 실행해야 한다.
