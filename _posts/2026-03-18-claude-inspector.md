---
title: "[Tool] - Claude Code 가 AI 에게 보내는 프롬프트를 직접 뜯어봤다     "
date: 2026-03-18
categories:
 - Android
tags:
 - claude-code
 - llm
 - prompt-engineering
 - electron
 - reverse-engineering
---

Claude Code 를 쓰다 문득 궁금해짐. CLAUDE.md 를 수정하면 AI 동작이 달라지는데, 실제로 Anthropic API 에 무엇을 보내는 걸까? 그래서 직접 들여다보기로 했고 — Claude Inspector 를 만들어 실제 요청을 캡처해봤더니, 꽤 재미있는 걸 발견함.

<!-- more -->

### 계기

<aside>
💡Claude Code 는 내부적으로 Anthropic API 를 호출하는 클라이언트 — 그 요청을 직접 보면 동작 원리를 이해할 수 있음
</aside>

- Claude Code 를 쓰다 보면 이런 의문이 생김
  - CLAUDE.md 를 길게 쓰면 정말 비용이 늘어날까?
  - `/commit` 같은 스킬을 실행하면 AI 에게 뭐가 전달되는 걸까?
  - 스크린샷을 찍으면 토큰을 얼마나 쓰는 걸까?
  - 대화가 길어질수록 왜 느려지고 비싸지는 걸까?

- 공식 문서에는 상세히 나와 있지 않음
  - Anthropic API 스펙은 공개돼 있지만, Claude Code 가 실제로 어떻게 조립해서 보내는지는 블랙박스
  - 직접 보는 것 외에 방법이 없다고 판단

- 그래서 Claude Inspector 를 만들게 됨
  - MITM(Man-in-the-Middle) 프록시로 요청을 가로채어 시각화하는 앱

### Claude Inspector 란?

<aside>
💡Claude Code 가 Anthropic API 에 보내는 HTTP 요청을 실시간으로 캡처·시각화하는 macOS Electron 앱
</aside>

- 동작 방식
  - Claude Inspector 를 실행하면 localhost:9090 에서 HTTP 프록시 시작
  - `ANTHROPIC_BASE_URL=http://localhost:9090 claude` 로 Claude Code 실행
  - 이후 모든 API 요청이 Claude Inspector 를 통해 중계됨
  - 요청 내용(system, messages, tools)과 응답을 실시간으로 볼 수 있음

- 설치
  - `brew install --cask kangraemin/tap/claude-inspector`
  - 또는 GitHub Releases: [https://github.com/kangraemin/claude-inspector](https://github.com/kangraemin/claude-inspector)

    ![ko-1.png](/assets/images/posts/2026-03-18-claude-inspector/ko-1.png)

    - 좌측 패널: 캡처된 요청 목록 — 어떤 파일/도구가 연관됐는지, 요청 시점이 표시됨
    - 우측 패널: 선택한 요청의 실제 전송 내용 — Request / Response / Analysis 탭으로 구성
    - Request 탭을 열면 `<system-reminder>` 태그 안에 CLAUDE.md 내용이 그대로 박혀 있는 것을 볼 수 있음

- 요청별 비용과 토큰 수를 직접 확인할 수 있음

    ![ko-2.png](/assets/images/posts/2026-03-18-claude-inspector/ko-2.png)

    - 우측 상단에 해당 요청의 입력/출력 토큰 수 + 비용($) 이 실시간으로 찍힘
    - "이번 대화에 얼마 썼지?" 궁금할 때 요청 하나하나의 비용을 확인 가능
    - 직접 써보면 알겠지만 — CLAUDE.md 가 길면 아무 작업도 안 했는데 기본 입력 토큰이 이미 높게 찍혀 있음

- Analysis 탭 에서 요청 내용을 구조화된 형태로 분석 가능

    ![ko-3.png](/assets/images/posts/2026-03-18-claude-inspector/ko-3.png)

    - "You are Claude Sonnet, Anthropic's official CLI for Claude." — 이런 system 프롬프트가 어떻게 구성되는지 항목별로 볼 수 있음
    - 어떤 규칙, 어떤 메모리가 이번 요청에 실제로 포함됐는지 확인 가능

### 직접 확인해보기

Claude Inspector 를 켜두고 Claude Code 를 써보면, 아래 것들을 직접 눈으로 확인할 수 있음.

#### 1. CLAUDE.md 는 매 요청마다 전부 전송됨

<aside>
💡CLAUDE.md + rules + memory 전체가 system-reminder 블록으로 매 API 요청에 삽입됨
</aside>

- 확인 방법
  - Claude Inspector 켜고 Claude Code 에서 아무 말이나 입력
  - Request 탭 → `<system-reminder>` 태그 찾기
  - 본인의 CLAUDE.md 내용이 그대로 들어가 있는 것 확인 가능
  - 프로젝트 CLAUDE.md, 글로벌 CLAUDE.md, rules/, memory/ 파일들이 전부 합쳐져서 삽입됨

- CLAUDE.md 가 길수록 모든 요청의 입력 토큰이 올라감 → 비용 직결
  - 1000줄짜리 CLAUDE.md 는 단 한 번만 쓰는 게 아니라 매 요청마다 전송됨
  - 짧은 질문 하나를 보냈는데 페이로드가 수만 토큰 — 실제로 보면 충격적임

#### 2. MCP 도구 는 지연 로드됨

<aside>
💡빌트인 도구 27개는 전체 JSON 스키마 포함, MCP 도구는 이름만 먼저 전송 → 필요 시 스키마 동적 추가
</aside>

- 확인 방법
  - Tools 탭에서 `tools[]` 배열 확인
  - Read, Edit, Bash 등 빌트인 도구는 매번 전체 JSON 스키마가 포함됨
  - MCP 도구 (context7, til-server 등) 는 처음엔 `<available-deferred-tools>` 에 이름만 나열됨
  - 실제 MCP 도구를 사용하는 순간 그 도구의 스키마가 추가되는 것을 실시간으로 볼 수 있음
  - `tools[]` 배열 크기가 대화 중 늘어나는 이유가 이것

#### 3. 이미지 는 Base64 로 JSON 본문에 인라인 삽입됨

<aside>
💡스크린샷 → base64 인코딩 → JSON body 직접 삽입 → 스크린샷 한 장에 수백 KB 추가
</aside>

- 확인 방법
  - Claude Code 에서 스크린샷 캡처
  - 해당 요청의 Request 탭 → messages 에서 엄청나게 긴 base64 문자열 확인 가능
  - 우측 상단 비용 패널에서 스크린샷 찍기 전/후 입력 토큰 차이를 직접 비교해보면 됨
  - 스크린샷 한 장이 생각보다 훨씬 비쌈

#### 4. Slash Command 와 Skill 은 완전히 다른 경로로 처리됨

<aside>
💡/mcp, /clear 같은 커맨드는 로컬 처리, /commit 같은 스킬은 전체 프롬프트가 user message 에 삽입됨
</aside>

- 확인 방법
  - `/clear` 입력 → Claude Inspector 에 새 요청이 찍히지 않음 (로컬 처리)
  - `/commit` 같은 스킬 입력 → Messages 탭에서 `<command-message>` 래퍼로 스킬 전체 내용이 삽입된 것 확인
  - 스킬 실행 이후 요청들을 보면 messages[] 에 스킬 내용이 계속 남아있음
  - 긴 스킬일수록 그 이후 모든 요청의 입력 토큰이 올라간 것이 비용 패널에서 바로 보임

#### 5. messages[] 는 매 요청마다 전체 재전송됨

<aside>
💡HTTP stateless — 매 요청마다 처음 대화부터 전체 이력을 다시 전송
</aside>

- 확인 방법
  - 대화를 10턴, 20턴 진행하면서 각 요청의 크기(입력 토큰 수)를 비교
  - 턴이 쌓일수록 선형으로 증가하는 것이 비용 패널에서 그대로 보임
  - 30턴 넘어가면 단순한 한 마디 질문에도 입력 토큰이 이미 수만 개

- 대응 방법
  - `/clear` — 컨텍스트 초기화, messages[] 리셋 → 다음 요청부터 다시 낮아짐
  - `/compact` — 대화를 요약으로 교체 (크기 감소)
  - 작업 단위로 세션 분리 권장

오픈소스: [https://github.com/kangraemin/claude-inspector](https://github.com/kangraemin/claude-inspector)
