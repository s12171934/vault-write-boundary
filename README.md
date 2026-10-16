---
title: vault-write-boundary
created: 2026-10-16
updated: 2026-10-16
tags: [domain/지식관리, area/문서관리, agent/claude-1b219bef-b3db-4ef6-8088-81f7f0d19120]
source: ai
---

# vault-write-boundary

> 볼트 안에서 도는 에이전트 세션의 쓰기 권한을 볼트 내부로 묶는 규칙 스킬이다. 볼트 밖 파일·저장소·외부 서비스 쓰기는 기본 거부다.

## 하는 일

- 예외는 사용자의 명시적 지시(실행 전 대상·범위·내용 확인), 볼트 규칙의 상시 승인, 사용자의 지속 지시 셋뿐이다.
- 안팎 판정은 경로 문자열이 아니라 심볼릭링크를 푼 realpath 로 한다.

## 설치

에이전트 도구의 스킬 경로에 clone 하거나 링크한다.

```bash
git clone https://github.com/s12171934/vault-write-boundary.git ~/.claude/skills/vault-write-boundary
# 다른 도구는 그 도구의 스킬 경로에 링크한다. 예: ln -s ~/.claude/skills/vault-write-boundary ~/.codex/skills/vault-write-boundary
```

- 볼트 세션이 시작할 때와 매 쓰기 직전에 로드되도록 도구 스킬 경로에 둔다.

## 구성과 공개 면

- [[20_Agents/skills/vault-write-boundary/SKILL|SKILL.md]] — 스킬 본문. 형식은 에이전트 도구가 정한다.
- 공개 면은 스킬 이름(`vault-write-boundary`)뿐이다. 다른 스킬은 이 이름으로만 이 스킬을 가리킨다.
