# lvtlvt-harness — 개인 하네스 마켓플레이스

Claude Code 플러그인 마켓플레이스. 주제만 주면 **기획 → 설계 → 구현**을 자동 오케스트레이션하는 개발 하네스 모음이다.

## 수록 플러그인

| 플러그인 | 설명 | 구성 |
|----------|------|------|
| [`dev-harness`](plugins/dev-harness) | 기본 하네스 — 기획→설계→구현 오케스트레이션 | 에이전트 8종 + 스킬 2종 |
| [`dev-harness-plus`](plugins/dev-harness-plus) | 강화판 — 보안·코드리뷰·문서·디버깅 전담 추가, Discovery·리스크 게이팅·추적성 매트릭스·적대적 검증·보안 게이트 | 에이전트 12종 + 스킬 2종 |
| [`dev-harness-structured-v1`](plugins/dev-harness-structured-v1) | 산출물 구조 고정판 — 디렉토리·파일명·명명 규칙·섹션 구조를 단일 출처로 명시해 어떤 프로젝트에서든 동일한 구조의 산출물 생성. 산출물은 마크다운, 현황판만 HTML, 요청 시 HTML 내보내기 | 에이전트 12종 + 스킬 3종 |

> 세 플러그인은 트리거가 겹치므로 **하나만 활성화**해 쓰기를 권장한다. 에이전트/스킬은
> `dev-harness:` / `dev-harness-plus:` / `dev-harness-structured-v1:` 접두사로 구분된다.

## 설치

```bash
# 1) 마켓플레이스 등록 (한 번만)
claude plugin marketplace add https://github.com/SimJaeSugn/marketplace.git

# 2) 원하는 플러그인 설치 — --scope로 설치 위치 선택 (생략 시 user)
claude plugin install dev-harness-structured-v1@lvtlvt-harness                  # user(기본): 내 모든 프로젝트
claude plugin install dev-harness-structured-v1@lvtlvt-harness --scope project  # 이 저장소 팀 공유(.claude/settings.json, 커밋)
claude plugin install dev-harness-structured-v1@lvtlvt-harness --scope local    # 이 저장소에서 나만(.claude/settings.local.json)

# 3) 확인
claude plugin list
```

플러그인 이름만 바꾸면 `dev-harness` / `dev-harness-plus`도 동일하게 설치된다.

### 설치 scope (`--scope`)

`--scope`는 플러그인을 **어디에 설치(활성화)할지** 정한다. **생략하면 user**다.

| scope | 플래그 | 기록 위치 | 효과 | 언제 쓰나 |
|-------|--------|-----------|------|-----------|
| **user** (기본) | `--scope user` | `~/.claude/settings.json` | 내 **모든 프로젝트**에서 사용 | 개인이 어디서나 쓰고 싶을 때 |
| **project** | `--scope project` | `.claude/settings.json` (git 커밋됨) | 이 저장소를 받는 **팀원 모두** 사용 | 팀에 동일 하네스를 배포할 때 |
| **local** | `--scope local` | `.claude/settings.local.json` (gitignore) | 이 저장소에서 **나만** 사용 | 이 레포에서만, 공유 없이 시험할 때 |

- 설치하면 해당 scope 설정 파일의 **`enabledPlugins`** 에 등록된다.
- 제거하지 않고 잠깐 끄기/켜기: `claude plugin disable <plugin>@lvtlvt-harness --scope <scope>` / `claude plugin enable ...`.
- 관리자가 서버 설정으로 강제 배포하는 **`managed`** scope도 있으나, 읽기 전용이라 사용자가 직접 설치하는 대상은 아니다.

## 사용

- **기획~설계 시작:** "이 제품 기획부터 설계까지 해줘", "PRD랑 아키텍처 만들어줘" → `dev-orchestrator` 스킬 자동 발동
- **구현 시작:** "설계 끝났으니 개발 시작", "개발팀 돌려줘" → `team-dev` 스킬 자동 발동
- **재실행/부분 수정:** "기획만 다시", "설계만 다시", "보완해줘"

자세한 사용법·산출물 구조는 각 플러그인의 README를 참고한다.

## 업데이트 / 재배포

```bash
# 해당 플러그인의 .claude-plugin/plugin.json 의 version 을 올린 뒤
claude plugin marketplace update
```

> 버전은 각 `plugin.json` 한 곳에서만 관리한다(마켓플레이스 카탈로그에는 version을 두지 않는다).
