# Codex·Claude Code 연결

문서 확인일: 2026-10-26. 지원 대상은 로컬 Codex와 Claude Code의 루트 프로젝트 지침이다. 클라우드·조직 정책·개인 설정·하위 디렉터리 규칙의 동작까지 동일하다고 가정하지 않는다.

## 공통 원본과 연결

Codex는 `AGENTS.md`를 프로젝트 지침으로 사용한다. 같은 디렉터리의 `AGENTS.override.md`가 우선할 수 있고 실행 위치에 따라 적용되는 하위 지침이 달라진다. override가 있으면 `AGENTS.md`를 생성했다는 이유만으로 연결이 완료됐다고 보고하지 않는다.

Claude Code는 `CLAUDE.md`에서 `@`와 경로를 사용해 다른 파일을 가져올 수 있다. 공통 지침을 복제하는 대신 가져오기를 사용한다. 최신 버전의 `AGENTS.md` 직접 지원은 설정과 기존 Claude 파일의 존재에 영향을 받으므로 모든 환경에서 직접 로딩된다고 가정하지 않는다.

별도 기존 로딩 경로가 없을 때 루트 `CLAUDE.md`의 최소 내용:

```markdown
@AGENTS.md
```

기존 `.claude/CLAUDE.md`를 유지할 경우 가져오기는 해당 파일 기준이다.

```markdown
@../AGENTS.md
```

기존 Claude 전용 내용은 가져오기 뒤에 보존할 수 있다. 이미 루트와 `.claude/`에 파일이 모두 있다면 내용을 확인하고 연결은 필요한 한 곳에만 둔다. 어느 한 파일도 일괄 삭제하지 않는다. `AGENTS.md`에서 다시 Claude 파일을 가져오게 만들어 순환시키지 않는다.

일반 문서 링크와 자연어 읽기 안내는 네이티브 가져오기가 아니다. Codex가 Claude의 `@` 문법을 동일하게 처리한다고 가정하지 않는다. 공통 지침 자체는 도구 전용 문법 없이 작성한다. Windows 이식성을 위해 새 심볼릭 링크를 만드는 방식은 사용하지 않는다.

## 기존 Claude 자동 로딩 경로 보존

Claude의 기본 `claude-md-or-agents-md` 설정에서는 프로젝트 경로에 Claude 지침 파일이 없을 때 루트 `AGENTS.md`뿐 아니라 `.claude/AGENTS.md`도 읽는다. 새 `CLAUDE.md`를 만들면 이 자동 선택이 바뀔 수 있다. 기존 `.claude/AGENTS.md`를 수정하지 않았더라도 가져오기에서 빠지면 지침 보존에 실패한다.

예를 들어 기존 `.claude/AGENTS.md`가 있고 두 대상 사이에 중복·충돌·순환이 없다면, 새 루트 `CLAUDE.md`는 다음처럼 기존 로딩 내용을 유지할 수 있다.

```markdown
@AGENTS.md
@.claude/AGENTS.md
```

`.claude/CLAUDE.md`를 연결 파일로 사용할 때 같은 대상의 경로는 각각 `@../AGENTS.md`, `@AGENTS.md`이다. 원문의 상대 가져오기가 계속 같은 위치에서 해석되도록 기존 파일을 유지한다. 기존 가져오기 체인으로 이미 도달하는 대상은 다시 추가하지 않는다. `.claude/AGENTS.md`의 내용을 무조건 루트 공통 지침으로 복사하거나 Claude 전용 내용을 Codex 전체 범위로 확대하지 않는다.

파일 존재·내용 보존과 로딩 경로 보존을 따로 점검한다. 개인·관리자 설정 때문에 실제 적용 여부가 불명확하면 그 한계를 보고하고, 사용자 설정 변경이나 개인 문서의 공유 문서 복사는 하지 않는다. 이 확인은 파일 연결 검증이며 Claude 런타임 로딩 검증을 대신하지 않는다.

## 스킬 자체의 설치와 호출

같은 `repo-bootstrap/` 폴더 전체를 아래 경로 중 필요한 곳에 복사한다. 저장소의 `skills/`는 배포 원본 위치이며 그 위치만으로 자동 설치되지는 않는다.

| 환경 | 개인 설치 | 프로젝트 설치 | 호출 |
|---|---|---|---|
| Codex | `~/.agents/skills/repo-bootstrap/` | `.agents/skills/repo-bootstrap/` | `$repo-bootstrap` |
| Claude Code | `~/.claude/skills/repo-bootstrap/` | `.claude/skills/repo-bootstrap/` | `/repo-bootstrap` |

`agents/openai.yaml`은 Codex용 표시 메타데이터다. 공통 작업 절차는 `SKILL.md`에 있으며 Claude 전용 명령 치환이나 특정 도구 이름에 의존하지 않는다. 조직 정책으로 로컬 스킬이 제한되면 그 사실을 보고하고 설정을 우회하지 않는다.

## 실제 로딩 확인

파일 참조 점검만으로 각 제품의 로딩 성공을 증명할 수 없다. 대상 프로젝트에서 새 세션을 열고 사용한 지침의 출처를 확인한다. 가능하면 해당 제품이 제공하는 컨텍스트 표시나 로딩 기록을 함께 확인한다. 실제 세션 검증을 하지 않았다면 `참조 확인 완료 / 런타임 로딩 미검증`으로 구분한다.

## 공식 자료

- [Codex AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
- [Codex 스킬](https://learn.chatgpt.com/docs/build-skills)
- [Claude Code 프로젝트 지침과 가져오기](https://code.claude.com/docs/en/memory)
- [Claude Code 스킬](https://code.claude.com/docs/en/skills)
