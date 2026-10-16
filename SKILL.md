---
name: vault-write-boundary
description: >-
  Write-permission boundary for any agent session running inside the user's Obsidian vault.
  Load it at session start in the vault and BEFORE every write — Write/Edit, file creation,
  deletion, move, or any state-changing call to an external system (git remote, Jira, Notion,
  Confluence, GitHub, filesystem outside the vault). The session may modify vault-internal
  documents only; writes outside the vault are closed by default (read-only). Exceptions: the
  standing approvals the vault's own rules name (e.g. re-linking tool skill paths when a skill
  submodule is added or removed), standing instructions the user has given for a named target,
  and a command in which the user directly and explicitly names some other external
  modification — and even then the target, scope, and content must be confirmed with the user
  before executing. Inside/outside is judged on the symlink-resolved real path (realpath), not
  the path string: a tool skill path that links into the vault is an internal write, while a
  vault path that links out is external. Triggers: working in the vault, "볼트 세션",
  "외부 수정", editing Jira/Notion/repo files from a vault session, or any doubt about whether
  a write target is inside the vault.
---

# 볼트 쓰기 경계 (vault-write-boundary)

> 볼트 안에서 실행되는 세션의 쓰기 권한 규칙이다. 볼트 루트는 세션이 시작된 위치에서 위로 올라가며
> `AGENTS.md` 와 `00_Vault/` 가 함께 있는 폴더다 (볼트 밖 세션이 볼트를 찾는 법은 `vault-entry` 스킬).
> 이 스킬은 세션이 쓰기 직전마다 경계를 명심하도록 하는 리마인더다.

## 규칙

1. **볼트 내부만 쓴다.** 볼트 세션의 생성·수정·삭제·이동 대상은 볼트 내부 문서로 한정한다.
2. **외부 쓰기는 닫힌 권한(기본 거부).** 볼트 밖 파일시스템, 코드 저장소, Jira·Notion·Confluence·GitHub 등 외부 시스템은 읽기·조회만 허용한다. 조사·수집을 위한 읽기는 자유다.
3. **예외는 셋뿐이다.**
   - **① 명시적 지시** — 사용자가 해당 명령에서 그 외부 대상의 수정을 직접 명시한 경우. 과거 대화의 승인, 유사 작업의 관례, "필요해 보임"은 명시가 아니다.
   - **② 볼트 규칙의 상시 승인** — 볼트 규칙 노트가 명시한 외부 쓰기. 예: 내 스킬 서브모듈을 추가·제거할 때 도구 스킬 경로의 링크를 맞추는 스크립트 재실행. 그 규칙이 적은 범위를 넘지 않는다.
   - **③ 사용자의 지속 지시** — 사용자가 특정 대상에 대해 "항상 하라"고 지시해 둔 쓰기(예: 볼트 저장소·서브모듈의 push). 지시가 이름 붙인 대상에만 적용한다.
4. **①일 때는 꼼꼼한 진행 확인.** 명시적 지시가 있어도 쓰기 실행 전에 대상·범위·내용을 사용자에게 확인받은 뒤 진행한다. 확인 없이 바로 실행하지 않는다.
5. **명시 없는 외부 수정 필요는 보고만.** 작업 중 외부 수정이 필요해지면 실행하지 않고 사용자에게 알린다. 처리는 사용자의 명시적 지시 또는 볼트 밖 세션에서 한다.
6. **판정은 realpath 기준.** 내부/외부는 입력한 경로 문자열이 아니라 심볼릭링크를 해석한 실제 경로로 판정한다.
   - realpath 가 볼트 내부 → **내부 쓰기(허용)**. 경로가 볼트 밖이어도 무관하다. 예: `~/.claude/skills/<스킬>/SKILL.md` 의 실체가 볼트 `20_Agents/skills/<스킬>/SKILL.md` 이면 링크 경유로 고쳐도 볼트 문서를 고치는 것이다. 다만 **가능하면 볼트 쪽 실제 경로로 편집**한다.
   - realpath 가 볼트 밖 → **외부 쓰기(기본 거부)**. 경로가 볼트 안이어도 무관하다. 볼트 안에 밖을 가리키는 링크를 두어 규칙 1·2 를 우회할 수 없다.
   - **링크 자체를 만들거나 지우는 것은 별개다.** `ln -sfn`·링크 삭제·대상 변경은 볼트 밖 디렉터리 엔트리를 바꾸는 외부 쓰기다. 예외 ② 에 해당할 때만 그 작업의 일부로 바로 한다.
7. **서브모듈 안은 내부, 그 원격은 외부.** 볼트 안 서브모듈의 파일을 고치는 것은 내부 쓰기이고, 서브모듈 저장소로 push 하는 것은 외부 쓰기다 (예외 ①·③ 이 있어야 한다).

## 쓰기 전 체크

모든 Write/Edit/삭제/외부 API 상태 변경 호출 직전에 확인한다.

- [ ] 대상 경로에 심볼릭링크가 끼어 있는가? → 있으면 realpath 로 해석한 뒤 판정한다 (`readlink -f`, `ls -la` 로 확인).
- [ ] 해석된 실제 경로가 볼트 내부인가? → 내부면 볼트 규칙(`AGENTS.md`)에 따라 진행.
- [ ] 외부라면: 볼트 규칙의 상시 승인(②)이나 사용자의 지속 지시(③)에 해당하는가? → 맞으면 진행.
- [ ] 아니라면: 사용자가 이번 명령에서 이 대상의 수정을 직접 명시했는가? → 아니면 실행하지 않고 보고.
- [ ] 명시했다면: 대상·범위·내용을 사용자에게 확인받았는가? → 확인 후에만 실행.

## 경계 판정 예시

- 볼트 내부 (허용): 노트 갱신, `_index.md` 등록, 볼트 안 서브모듈의 파일 수정.
- 볼트 내부 — 링크 경유 (허용): 볼트 스킬을 가리키는 도구 스킬 경로(`~/.claude/skills/`, `~/.codex/skills/` 등) 안의 파일 편집.
- 외부 읽기 (허용): Jira 이슈 조회, GitHub PR 열람, 외부 저장소 코드 검색.
- 외부 쓰기 (기본 거부): Jira 이슈 상태 전환·코멘트, Notion 페이지 수정, 볼트 밖 저장소 커밋, `~/.claude/settings.json` 수정, 볼트 밖 파일 생성.
- 상시 승인 (예외 ②): 스킬 서브모듈 추가·제거에 따르는 스킬 링크 정리 — 볼트 규칙이 정한 스크립트로 같은 작업 안에서 수행한다.
- 외부 쓰기 (기본 거부) — 그 밖의 링크 관리: 스킬 변경과 무관하게 링크를 만들거나 지우는 일.
- 외부 쓰기 (기본 거부) — 볼트 안에서 밖을 가리키는 링크 경유: 볼트 안 경로처럼 보여도 realpath 가 코드 저장소·홈 설정 파일이면 외부 쓰기다. 외부 스킬 설치 도구가 볼트 스킬 목록에 만든 링크(원본 `~/.agents/skills/`)도 여기에 해당한다.
