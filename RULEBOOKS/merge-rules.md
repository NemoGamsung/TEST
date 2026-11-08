# 머지 규칙

PR이 어떤 조건을 만족해야 머지되는지, 누가 어떤 방식으로 머지하는지 정리합니다. **강제**는 레포 설정(ruleset, CI)이 막는 규칙이고, **약속**은 팀원이 지키는 규칙입니다. 설정값은 [레포 설정](./repo-settings.md)에 있습니다.

## 1. 대상 브랜치별 머지 조건

| 조건 | → `develop` | → `main` |
|---|---|---|
| PR 필수 (직접 push 금지) | ✅ | ✅ |
| 승인 수 | **1명** (작성자 제외) | **1명** (작성자 제외) |
| 경로별 담당 팀 승인 | 아래 2번 표 참고 (약속) | — |
| production 배포 승인 | — | **Lead** (Environment 승인으로 강제) |
| 필수 체크 | `ci-passed`, `pr-title` | `ci-passed`, `pr-title`, `guard-main` |
| 브랜치 최신화 필요 | ❌ | ✅ |
| 모든 리뷰 코멘트 해결 | ✅ | ✅ |
| 허용 머지 방식 | **Squash만** | **Merge commit만** |
| force push · 브랜치 삭제 | 금지 | 금지 |
| 머지하는 사람 | PR 작성자 | Lead (약속) |

- `guard-main`은 출발 브랜치가 `develop` 또는 `hotfix/*`가 아니면 실패합니다. 작업 브랜치에서 main으로 바로 PR을 열 수 없습니다.

## 2. 경로별 담당 팀 승인 (약속)

develop으로 가는 PR은 바뀐 경로의 담당 팀마다 **1명 이상** 승인받은 뒤 머지합니다. 리뷰 요청은 CODEOWNERS가 담당 팀 전원에게 자동으로 보냅니다. 설정이 강제하는 것은 "Write 권한자 1명 승인"뿐이므로, 아래 표는 작성자가 직접 확인하고 지킵니다.

| 경로 | 승인이 필요한 팀 |
|---|---|
| `apps/web/**` | `homepage-fe` 1명 |
| `apps/api/**` | `homepage-be` 1명 |
| `apps/ai/**` | `homepage-ai` 1명 |
| `packages/contracts/openapi.yaml` | `homepage-fe` 1명 **+** `homepage-be` 1명 |
| `packages/contracts/ai-internal.yaml` | `homepage-be` 1명 **+** `homepage-ai` 1명 |
| `infra/**`, `scripts/**`, `docker-compose.yml` | `homepage-lead-pm` 1명 |
| `.github/**` | `homepage-lead-pm` 1명 |
| `RULEBOOKS/**` | 없음 (팀원 아무나 1명, 리뷰 요청은 `homepage-lead-pm`) |
| 그 외 (README 등) | 없음 (팀원 아무나 1명) |

- 한 PR이 여러 경로를 건드리면 걸린 팀 조건을 **모두** 채웁니다. 예: `apps/web`과 `openapi.yaml`을 바꾼 PR → fe 1명 + be 1명.
- **작성자가 그 팀의 유일한 멤버이면 그 팀 승인은 생략하고** 나머지 팀 승인만 받습니다. 예: AI 담당이 `ai-internal.yaml`을 바꾼 PR → be 1명. 걸린 팀이 본인 팀뿐이면(예: `apps/ai`만 수정) 다른 팀원 아무나 1명에게 승인받습니다.
- 승인은 이 프로젝트 팀원(`Homepage-*` 팀 멤버)에게만 받습니다. 설정상으로는 조직의 Write 권한자 누구의 승인이든 1명으로 인정됩니다.

## 3. 리뷰 규칙

### 리뷰어 지정
- CODEOWNERS가 변경 경로의 담당 팀에 리뷰를 **자동으로 요청**하고, 팀원 전원이 알림을 받습니다.
- 팀원이 여러 명이면 먼저 보는 사람이 리뷰합니다. 리뷰를 시작한 사람은 PR에 본인을 Assignee로 추가합니다. (약속)

### 응답 시간 (약속)
- 첫 리뷰는 요청 후 **48시간 이내**에 합니다. 시험 기간 등으로 어렵다면 팀 채널에 알립니다.
- 48시간이 지나도 리뷰가 없으면 작성자가 팀 채널에서 다시 요청합니다.

### 코멘트 말머리 (약속)

| 말머리 | 의미 | 작성자 대응 |
|---|---|---|
| `[필수]` | 고치지 않으면 승인 불가 | 수정 후 답글 |
| `[권장]` | 고치면 좋지만 이번 PR에서 안 해도 됨 | 수정하거나, 별도 이슈로 남기고 답글 |
| `[질문]` | 의도 확인 | 답글 |
| `[사소]` | 오타, 취향 차이 | 자유 |

모든 코멘트 스레드는 해결(Resolve)해야 머지할 수 있습니다. 스레드는 **코멘트를 단 리뷰어가 해결**합니다. (약속)

### 승인 후 변경 (약속)
승인 후에 푸시해도 승인은 유지됩니다(설정상 재승인을 강제하지 않음). **승인 후 로직을 바꾸는 커밋을 추가했다면 리뷰어에게 다시 요청**합니다. 오타 · 포맷 수정은 예외입니다.

### 리뷰어가 보는 것
1. 이슈의 완료 조건을 만족하는가
2. 권한 검사가 빠지지 않았는가 (비회원 / General / Core / Lead)
3. 개인정보가 로그 · 응답 · 에러 메시지로 새지 않는가
4. 계약(yaml)과 구현이 일치하는가
5. 테스트가 있거나, 없는 이유가 타당한가

## 4. 릴리스 (develop → main)

1. Lead가 develop → main PR을 엽니다. 제목은 `release: YYYY-MM-DD`입니다.
2. PR 템플릿 하단의 "릴리스 PR" 체크리스트를 채웁니다. 특히 staging에서 주요 흐름을 확인합니다.
3. 팀원 1명이 승인하면 Lead가 **Create a merge commit**으로 머지합니다.
4. main 푸시로 production 배포 워크플로우가 시작되고, `production` 환경 승인 대기 상태가 됩니다. Lead가 승인하면 배포됩니다.
5. 배포 후 GitHub Release를 만들고 "Generate release notes"로 변경 목록을 남깁니다. 태그는 `vYYYY.MM.DD`입니다.

릴리스 주기는 고정하지 않습니다. 기능 묶음이 staging에서 확인되면 릴리스합니다. 단, **지원 기간 중에는 지원서 관련 변경을 릴리스하지 않습니다.** (약속)

## 5. 핫픽스 (production 긴급 수정)

```
main ──● hotfix/58-login-cookie-domain ──PR──▶ main ──back-merge PR──▶ develop
```

1. `main`에서 `hotfix/<이슈번호>-<설명>` 브랜치를 만듭니다.
2. `main`으로 PR을 엽니다. 1명이 승인하면 Lead가 merge commit으로 머지하고, `production` 환경 승인 후 배포됩니다.
3. main → develop **back-merge PR**을 엽니다. `back-merge` 워크플로우가 자동으로 열고, 자동으로 열리지 않으면 Lead가 직접 엽니다.
4. **Lead가 back-merge PR을 "Create a merge commit"으로 머지합니다.** develop은 squash만 허용하므로 Lead의 우회 권한(bypass)으로 머지합니다.

back-merge 전에는 main의 최신화 조건 때문에 다음 릴리스 PR을 머지할 수 없습니다.

## 6. 규칙 우회 (bypass)

- 우회 권한은 **Repository admin 역할**에만 있고, 모드는 **"PR에서만(For pull requests only)"**입니다. 레포의 Admin 권한은 `homepage-lead-pm` 팀(Lead, PM)에만 있으므로 우회할 수 있는 사람은 이 두 명입니다. 이들도 보호 브랜치에 직접 push할 수 없고, PR 머지 시점에만 규칙을 건너뛸 수 있습니다.
- Organization Owner는 레포 권한과 관계없이 Admin으로 취급되어 우회 권한을 함께 갖습니다.
- 우회는 back-merge 머지와, CI 장애로 체크가 돌지 않는 긴급 상황에만 씁니다.
- 우회 기록은 Settings → Rules → Insights에 남습니다. 긴급 우회 후에는 팀 채널에 사유를 공유합니다. (약속)
