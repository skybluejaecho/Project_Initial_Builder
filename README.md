# Repo Bootstrap

기존 파일을 보존하면서 협업 규칙과 PR 양식을 준비하고, Codex·Claude Code의 공통 프로젝트 지침을 연결하는 최소 스킬입니다.

## 지원 기능

| 기능 | 동작 |
|---|---|
| 협업 규칙·PR 양식 | 기존 규칙을 우선 사용하며 필요한 부분만 작성·보완 |
| 공통 AI 지침 연결 | `AGENTS.md`에 공통 지침을 두고 Claude 파일에서 가져오기 |
| 기존 설정 점검 | 참조, 명령 정의, 지침 충돌 후보를 읽기 전용으로 확인 |

기본 코드를 만든 뒤 협업을 시작할 때, 기존 프로젝트에 다른 에이전트를 도입할 때, 팀 규칙을 변경한 뒤 점검할 때 사용합니다. 평소 기능 개발이나 PR 코드 리뷰에 사용하는 스킬은 아닙니다.

CI 구축, 린터 설치, `.gitignore` 생성, 브랜치 보호, Git hook, 스킬 추천·설치·관리는 포함하지 않습니다. 초기 설정만 요청하면 브랜치 생성·커밋·푸시를 수행하지 않습니다.

## 설치

배포 원본은 [skills/repo-bootstrap](skills/repo-bootstrap/SKILL.md)입니다. `skills/`에 파일이 있다는 것만으로 현재 에이전트에 설치되지는 않습니다.

`repo-bootstrap` **폴더 전체**를 사용하는 도구의 경로 한 곳에 복사합니다. `SKILL.md`, `agents/`, `references/`, `assets/`를 함께 복사하세요. 실행에 별도 Python·Node 패키지나 GitHub 인증은 필요하지 않으며, 에이전트에 대상 프로젝트의 파일 접근 권한은 있어야 합니다.

| 도구 | 개인 설치: 여러 프로젝트에서 사용 | 프로젝트 설치: 해당 저장소에 공유 |
|---|---|---|
| Codex | `~/.agents/skills/repo-bootstrap/` | `.agents/skills/repo-bootstrap/` |
| Claude Code | `~/.claude/skills/repo-bootstrap/` | `.claude/skills/repo-bootstrap/` |

`~`는 사용자 홈입니다. 처음에는 설정 담당자의 개인 설치를 권장합니다. 두 도구를 함께 사용하면 각각 설치합니다. 같은 도구의 여러 경로에 중복 설치하지 않는 편이 관리하기 쉽습니다.

Windows PowerShell에서 이 저장소를 내려받은 폴더를 연 뒤 Codex용으로 설치하는 예시입니다. 같은 이름의 기존 설치가 있으면 덮어쓰지 않고 중단합니다.

```powershell
$skillSource = Join-Path (Get-Location).Path 'skills\repo-bootstrap'
$skillParent = Join-Path $env:USERPROFILE '.agents\skills'
$skillTarget = Join-Path $skillParent 'repo-bootstrap'
if (-not (Test-Path -LiteralPath (Join-Path $skillSource 'SKILL.md'))) {
    throw '이 저장소의 루트 폴더에서 실행하세요.'
}
if (Test-Path -LiteralPath $skillTarget) {
    throw '기존 설치가 있습니다. 변경 내용을 비교한 뒤 수동으로 갱신하세요.'
}
New-Item -ItemType Directory -Path $skillParent -Force | Out-Null
Copy-Item -LiteralPath $skillSource -Destination $skillTarget -Recurse
```

Claude Code용으로는 위 코드의 `$skillParent`를 다음과 같이 바꿉니다.

```powershell
$skillParent = Join-Path $env:USERPROFILE '.claude\skills'
```

macOS·Linux에서는 같은 폴더를 표의 개인 설치 경로에 복사하면 됩니다. 프로젝트 설치를 선택하면 대상 프로젝트의 표에 나온 경로에 복사하고 팀의 절차로 공유합니다.

설치 후 **설정할 대상 프로젝트**에서 새 세션을 열어 스킬 선택 목록을 확인하세요. Codex는 `$repo-bootstrap`, Claude Code는 `/repo-bootstrap`으로 호출합니다. 나타나지 않으면 폴더가 이중으로 중첩되지 않았는지, `SKILL.md`가 설치 폴더 바로 아래에 있는지 확인합니다. 조직 정책으로 로컬 스킬이 제한되면 관리자 정책을 따라야 합니다.

안내 대상은 로컬 Codex와 Claude Code입니다. 클라우드 세션과 일반 웹 채팅의 설치 방식까지 지원한다고 가정하지 않습니다. 설치 위치와 제품별 차이는 [호환성 참고](skills/repo-bootstrap/references/compatibility.md)에 정리했습니다.

## 사용 예시

아래 문장은 터미널 셸 명령이 아니라 각 에이전트의 대화 입력입니다.

### Codex: 초기 설정

```text
$repo-bootstrap
이 저장소의 협업 초기 설정을 해줘.
간단한 기본 컨벤션을 적용하고 브랜치는 feat/, fix/, docs/를 사용해줘.
PR 양식과 Codex·Claude 공통 지침도 준비해줘.
기존 파일과 이미 정해진 팀 규칙은 유지해줘.
```

### Claude Code: 기존 프로젝트에 공통 지침 연결

```text
/repo-bootstrap 기존 CLAUDE.md의 공통 팀 규칙을 Codex도 활용하게 정리해줘.
Claude 전용 내용과 기존 협업 문서·PR 템플릿은 유지해줘.
```

### 일부만 준비

```text
$repo-bootstrap PR 양식만 추가해줘. 다른 설정은 유지해줘.
```

### 점검만 수행

```text
$repo-bootstrap 현재 협업 규칙과 AI 지침을 점검만 해줘.
파일은 수정하지 말고 누락과 충돌 후보를 알려줘.
```

Claude Code에서는 위 요청의 `$repo-bootstrap`을 `/repo-bootstrap`으로 바꿉니다. 설정·보완과 읽기 전용 점검을 자연어 요청으로 구분하며 별도의 명령 옵션은 없습니다.

## 결과와 기존 파일 처리

모든 항목을 요청했고 기존 문서가 없으면 다음 파일을 준비합니다.

```text
대상 프로젝트/
├── CONTRIBUTING.md
├── AGENTS.md
├── CLAUDE.md
└── .github/
    └── pull_request_template.md
```

- Git 컨벤션은 기존 협업 문서가 원본입니다. 없다면 `CONTRIBUTING.md`를 생성합니다.
- `AGENTS.md`에는 프로젝트 고유 지침과 확인된 명령을 간결하게 기록하고, 협업 작업 전에 원본 문서를 읽도록 안내합니다.
- 루트 `CLAUDE.md`는 `@AGENTS.md`로 공통 지침을 가져옵니다. 기존 `.claude/CLAUDE.md`를 사용할 때는 상대 경로를 맞춥니다.
- 기존 PR 템플릿이 루트·`docs/`·`.github/` 또는 복수 템플릿 디렉터리에 있으면 새 기본 템플릿을 중복 생성하지 않습니다.
- 사용자 지정 문단·주석·Claude 전용 지침과 관련 없는 변경을 보존합니다. 의미가 충돌하는 부분은 보류하고 해당 선택만 질문합니다.
- 같은 요청과 근거로 다시 실행했을 때 변경할 내용이 없다면 파일을 수정하지 않도록 지시합니다.

설정 담당자가 diff를 검토하고 결과를 저장소에 반영하면 다른 팀원은 이 스킬을 설치하지 않아도 생성된 프로젝트 지침을 활용할 수 있습니다. 다시 설정하거나 점검할 사람만 스킬이 필요합니다.

## 점검 결과의 의미

이 스킬은 에이전트가 따르는 작업 지침이며 독립적인 동기화 프로그램이나 정책 강제 도구가 아닙니다. 파일 보존과 재실행 원칙은 지침으로 제공하므로 실제 변경은 리뷰해야 합니다.

점검에서는 파일을 수정하지 않고 명령·참조의 근거를 확인합니다. 기본적으로 패키지 설치와 테스트·빌드를 실행하지 않습니다. 다음 결과는 구분해서 보고합니다.

```text
변경: AGENTS.md 생성, 기존 Claude 파일에 공통 지침 연결
유지: 기존 협업 문서와 PR 템플릿
확인: 참조 경로 정상, 검증 명령 정의 존재
미확인: 테스트 실제 실행, 각 에이전트의 새 세션 로딩
```

명령 정의가 존재하는 것은 실행 성공과 다릅니다. 파일 참조가 맞는 것은 에이전트의 실제 로딩이나 규칙 준수 보장과도 다릅니다. 실제 로딩은 대상 도구의 새 세션에서 컨텍스트 표시·로딩 기록 등으로 별도 확인합니다.

## 제작 시 검증과 재검증 방법

2026-10-26에 스킬 작성 도구의 `quick_validate.py`로 형식을 검사하고 UTF-8 읽기, 내부 링크, Codex 메타데이터 길이, 미완성 항목 및 Git 공백 오류를 점검했습니다.

초기 구현 때 위 PowerShell 설치 예시를 임시 사용자 경로에서 Codex·Claude용으로 각각 실행했습니다. 네 파일의 복사 결과가 원본과 같은지, 같은 경로에 다시 설치할 때 기존 파일을 덮어쓰지 않는지 확인했습니다. 이후 아래 실제 설치·호출 검증을 별도로 수행했습니다.

별도 임시 예제 폴더에서 작성 에이전트가 지침을 수동 적용하고 다음 결과를 파일 해시와 참조 경로로 확인했습니다. 아래 표는 같은 검증을 반복할 때의 기준이기도 합니다.

| 시나리오 | 확인 기준 |
|---|---|
| 지침 없는 프로젝트에 기본안 적용 | 필요한 네 파일만 추가하고 기존 소스 보존, 없는 테스트 명령 생성하지 않음 |
| 기존 협업 문서와 다른 위치의 PR 템플릿 | 기존 파일·사용자 주석 보존, 중복 템플릿 생성하지 않음 |
| `.claude/CLAUDE.md`의 공통 규칙 이동 | 공통 규칙 의미 보존, Claude 전용 내용과 CRLF 보존, 상대 가져오기 해결 |
| 두 지침의 패키지 매니저 충돌 | 충돌 문서를 유지하고 판단 보류 |
| 잘못된 참조·정의 없는 명령의 읽기 전용 점검 | 문제를 식별하고 전체 파일 해시 유지 |
| PR 양식만 요청 | AI 지침과 협업 규칙을 수정하지 않음 |
| 설정 후 반복 검토 | 추가 편집 없이 파일 해시 유지 |

위 표는 작성 에이전트의 수동 검증이며 독립적인 모델 평가가 아닙니다. 후속으로 개인 스킬 경로에 실제 설치하고 새 CLI 세션에서 다음을 확인했습니다.

| 도구 | 실제 검증 결과 |
|---|---|
| Codex `0.163.0-alpha.11.1` | 설치 경로의 스킬 읽기, 초기 설정, 재실행 시 변경 없음, 읽기 전용 오류 점검 통과 |
| Claude Code `2.1.288` | 초기화 로그의 스킬·슬래시 명령 목록 등록 확인. 작업 실행은 API 키 인증 오류(HTTP 401)로 보류 |

기본 PATH의 Codex `0.148.0`은 설정된 모델을 지원하지 못해 실패했고, 이미 설치된 최신 실행 파일로 재시도했습니다. CLI 경로나 인증 문제는 스킬 인식 실패와 구분해야 합니다. 실제 검증 범위, 명령, 증거의 한계와 재실행 방법은 [런타임 검증 기록](docs/runtime-validation.md)에 정리했습니다.

제품별 검증을 추가할 때는 예제 프로젝트를 복사하고, 위 사용 예시를 새 세션에서 실행한 뒤 결과 diff와 파일 해시를 비교하세요. 점검·재실행 시에는 파일 생성·삭제도 없는지 함께 확인합니다. `quick_validate.py`는 스킬 작성 도구의 개발용 검사기이며 이 스킬의 실행 의존성이 아닙니다.
