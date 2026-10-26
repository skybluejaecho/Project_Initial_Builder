# 실제 설치·호출 검증

검증일: 2026-10-26 (Asia/Seoul). 대상은 로컬 Windows CLI이며 스킬 구현 커밋은 `ae61f78`입니다. 스킬 본문을 프롬프트에 붙여 넣지 않고 개인 설치 경로의 스킬을 이름으로 호출했습니다. 모든 작업은 배포 저장소 밖의 임시 프로젝트에서 수행했습니다.

## 설치

- Codex: `~/.agents/skills/repo-bootstrap/`
- Claude Code: `~/.claude/skills/repo-bootstrap/`
- 설치 전 동일 경로가 없는 것을 확인하고 네 파일을 복사했습니다. 설치본과 원본의 SHA-256을 비교했습니다.
- 실제 개인 설치본은 유지했습니다. 전역 모델·PATH·인증 설정은 변경하지 않았습니다.

## Codex 결과

`0.163.0-alpha.11.1`에서 기존에 설정된 모델로 서로 다른 새 세션 세 번을 실행했습니다. `$repo-bootstrap`을 입력하고 JSONL의 도구 호출·파일 변경 이벤트·최종 결과를 확인했습니다.

| 시나리오 | 결과 및 근거 |
|---|---|
| 초기 설정 | 개인 설치 경로의 `SKILL.md`, `references/compatibility.md`, PR 템플릿을 읽은 도구 호출 확인. 협업 규칙·공통 지침·Claude 연결·PR 템플릿 네 파일만 생성 |
| 기존 파일 보존 | 초기 fixture의 README·package.json·index.js SHA-256과 세션의 파일 해시 출력 일치 |
| 재실행 | 별도 새 세션에서 스킬을 다시 읽음. 파일 변경 이벤트 없음, 일곱 파일의 해시 비교 결과 변경 0건 |
| 읽기 전용 점검 | 별도 fixture의 누락된 `@missing-policy.md`와 정의 없는 `npm test`를 식별. 부모 프로세스가 점검 전후 네 파일의 목록·SHA-256 동일함을 별도 확인 |

초기 설정 fixture에는 `packageManager: npm@10.8.2`, `check: node --check index.js`만 정의했습니다. 에이전트는 검증 명령의 정의와 실행 결과를 구분했으며 패키지 설치·검사 실행·커밋·푸시를 수행하지 않았습니다.

초기 생성 파일은 Windows 하위 샌드박스에서 만들어져 부모 검증 프로세스의 직접 읽기가 거절됐습니다. 해당 파일의 내용과 해시는 샌드박스 내 도구 출력 및 파일 변경 이벤트로 검증했습니다. 읽기 전용 점검 fixture는 부모가 직접 해시를 비교할 수 있었습니다. 두 검증 방법의 차이를 보존합니다.

실행 형태는 다음과 같습니다. `$codexBinary`는 실제 확인한 실행 파일 경로, `$projectRoot`는 임시 프로젝트 경로입니다. 로그와 프롬프트는 프로젝트 밖에 둡니다.

```powershell
Get-Content -LiteralPath $promptFile -Raw -Encoding utf8 |
    & $codexBinary exec -C $projectRoot -s workspace-write `
        -c 'approval_policy="never"' --ephemeral --json `
        -o $finalFile - 1> $eventLog 2> $errorLog
```

읽기 전용 점검에서도 에이전트의 무수정 동작을 확인하기 위해 `workspace-write`를 유지했습니다. Git 초기화가 없는 별도 점검 fixture에는 `--skip-git-repo-check`를 추가했습니다.

## Claude Code 결과

`2.1.288`에서 `/repo-bootstrap`을 포함한 실제 요청으로 새 세션을 시작했습니다. 초기화 이벤트의 `skills`와 `slash_commands`에 모두 `repo-bootstrap`이 포함됐습니다.

그러나 모델 요청이 `HTTP 401 / Invalid API key`로 실패했습니다. 따라서 **설치·목록 인식은 확인했지만 스킬 작업 실행 성공은 확인하지 못했습니다.** 예제 프로젝트의 파일 목록·SHA-256은 시도 전후 동일했습니다. 사용자의 요청으로 실행 검증은 보류했습니다.

호출 시 MCP 연결을 제외하고 파일 읽기·편집과 스킬 호출에 필요한 도구만 제공했습니다. 전체 권한 검사를 우회하지 않았습니다.

```powershell
Get-Content -LiteralPath $promptFile -Raw -Encoding utf8 |
    claude -p --output-format stream-json --verbose `
        --permission-mode acceptEdits --permission-prompts none `
        --no-session-persistence --strict-mcp-config `
        --tools 'Read,Write,Edit,Glob,Grep,Skill,Bash' `
        1> $eventLog 2> $errorLog
```

작업 디렉터리는 임시 프로젝트입니다. fixture에는 기존 `CONTRIBUTING.md`, `docs/PULL_REQUEST_TEMPLATE.md`, `.claude/CLAUDE.md`의 공통 규칙·전용 주석과 CRLF 줄바꿈, 사용자 메모를 준비했습니다. 인증 복구 후에는 기존 문서와 양식 보존, 공통 규칙 이동, 상대 경로 가져오기, 재실행과 읽기 전용 점검을 검증해야 합니다.

## 환경 문제와 재시도

- **Codex 실행 파일 중복:** 기본 PATH의 `0.148.0`은 설정 모델 `gpt-6-astra`에 더 최신 Codex가 필요하다는 서버 오류로 종료됐습니다. `Get-Command codex -All`로 이미 설치된 다른 실행 파일을 찾아 버전을 확인하고 직접 호출했습니다. 스킬을 수정하거나 모델을 바꿔 해결한 문제가 아닙니다. 모든 버전의 최소 지원 범위를 검증한 것은 아닙니다.
- **Claude 인증:** 현재 프로세스의 `ANTHROPIC_API_KEY`가 거절됐습니다. 값을 출력하지 않고 인증 방식만 확인했습니다. 진단용 하위 셸에서 해당 환경변수를 제외했을 때 별도 로그인도 없었습니다. 영구 환경변수나 자격 증명 저장소는 변경하지 않았습니다. 인증을 로컬에서 수정한 뒤 새 셸에서 `claude auth status`와 실제 호출을 다시 확인해야 합니다. 이 명령의 로그인 표시만으로 API 인증 성공을 보장하지 않습니다.

원시 세션 로그와 임시 프로젝트는 저장소에 커밋하지 않았습니다. CLI 로그에는 로컬 경로와 계정 환경 메타데이터가 포함될 수 있으므로 공유 전 검토해야 합니다. 스킬 본문 수정이 필요한 결함은 이번 실행에서 발견하지 않았으며, 문서의 미검증 상태를 실제 결과에 맞게 갱신했습니다.

이 결과는 위 버전·환경·예제의 명시적 호출에 대한 확인입니다. UI 선택기, 자연어 자동 선택, Claude의 작업 실행, 클라우드 실행과 다양한 프로젝트에서의 행동 일관성을 보장하지 않습니다.
