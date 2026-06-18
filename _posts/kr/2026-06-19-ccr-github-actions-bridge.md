---
layout: single
title: "클라우드 AI 에이전트가 텔레그램을 못 보낼 때 — GitHub Actions 우회 패턴"
date: 2026-06-19 09:00:00 +0900
lang: kr
ref: ccr-github-actions-bridge
categories: [AI & Tooling]
tags: [Claude Code, CCR, GitHub Actions, Telegram, AI Agents, Building in Public, Automation]
permalink: /kr/2026/06/19/ccr-github-actions-bridge/
excerpt: "CCR 클라우드 환경은 텔레그램 API를 직접 호출할 수 없습니다. 이유를 파악하는 데 한참 걸렸고, 해결책은 생각보다 단순했습니다."
---

{% include lang-switch.html %}

[econ-radar 자동화 글]({{ site.baseurl }}/kr/2026/06/12/econ-radar-auto-publish/)에서 매일 자동으로 뉴스레터가 만들어지고 블로그에 올라간다고 했습니다. 텔레그램 채널(@econradar)에도 알림이 간다고 했습니다.

며칠이 지나도록 텔레그램 알림이 오지 않았습니다.

## 처음엔 토큰 문제인 줄 알았습니다

CCR(Claude Code Routine) 프롬프트에 텔레그램 발송 코드를 넣었습니다. 토큰을 넣고, 채팅 ID를 넣고, `sendMessage` API를 호출하게 했습니다. 로컬에서는 잘 됐습니다.

클라우드에서는 연결 자체가 안 됐습니다. 토큰을 다시 확인했습니다. 채팅 ID도 다시 확인했습니다. 같았습니다.

며칠 더 보다가 원인을 찾았습니다. CCR이 실행되는 Anthropic 클라우드 샌드박스 환경은 `api.telegram.org`로 나가는 아웃바운드 HTTPS가 차단돼 있었습니다. 설정 문제가 아니라 설계상 제약이었습니다.

## 해결 방향: 에이전트가 파일만 쓰고, 발송은 다른 곳에서

직접 부를 수 없다면, 인터넷 제한이 없는 곳에 맡기면 됩니다. GitHub Actions는 일반 인터넷 환경에서 실행됩니다.

구조는 간단합니다.

```
CCR (클라우드 에이전트)
  → 발송할 내용을 vault/push/YYYY-MM-DD.md 에 저장
  → git push origin main

GitHub Actions (인터넷 제한 없음)
  → vault/push/*.md 파일 변경 감지
  → 텔레그램 API 호출 → 채널 발송
```

에이전트는 파일을 쓰고 push만 합니다. 텔레그램 발송은 Actions가 맡습니다. git push가 이벤트 브리지 역할을 하는 구조입니다.

## 구현

### 발송 스크립트 (send_push.py)

push 파일을 읽어 텔레그램으로 보내는 스크립트입니다. 환경변수에서만 토큰을 읽습니다.

```python
import os, sys, requests

BOT_TOKEN = os.environ.get("TELEGRAM_BOT_TOKEN", "")
CHAT_ID   = os.environ.get("TELEGRAM_CHAT_ID", "")

if not BOT_TOKEN or not CHAT_ID:
    print("[ERROR] 환경변수 미설정")
    sys.exit(1)

date = sys.argv[1]
text = open(f"vault/push/{date}.md").read()

requests.post(
    f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage",
    json={"chat_id": CHAT_ID, "text": text, "parse_mode": "Markdown"}
)
```

### GitHub Actions 워크플로

`vault/push/` 아래 파일이 바뀔 때만 실행됩니다.

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'vault/push/*.md'

jobs:
  send:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: 날짜 감지
        id: detect
        run: |
          BEFORE="${{ github.event.before }}"
          if [ "$BEFORE" = "0000000000000000000000000000000000000000" ]; then
            BEFORE="HEAD~1"
          fi
          DATE=$(git diff --name-only "$BEFORE" HEAD \
            | grep -E '^vault/push/[0-9]{4}-[0-9]{2}-[0-9]{2}\.md$' \
            | sed 's|vault/push/||; s|\.md||' \
            | sort -r | head -1)
          echo "date=$DATE" >> "$GITHUB_OUTPUT"

      - name: 텔레그램 발송
        if: steps.detect.outputs.date != ''
        env:
          TELEGRAM_BOT_TOKEN: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          TELEGRAM_CHAT_ID: ${{ secrets.TELEGRAM_CHAT_ID }}
        run: python3 scripts/send_push.py ${{ steps.detect.outputs.date }}
```

GitHub Secrets에 `TELEGRAM_BOT_TOKEN`과 `TELEGRAM_CHAT_ID`를 등록하면 완성입니다.

## 작동하고 나서 발견한 두 번째 버그

첫날은 잘 됐습니다. 며칠 뒤 또 알림이 안 왔습니다.

이번에는 다른 이유였습니다. CCR이 한 번의 `git push`에 커밋을 여러 개 묶어서 보냈습니다.

```
b38e97f  econ-radar: 2026-06-16 자동 생성  ← vault/push 파일이 여기에
8834dde  topics MOC 갱신
027155e  반도체 topics MOC 갱신
4520dc9  topics MOC 최종 정합              ← HEAD
```

처음에 작성한 감지 로직이 `HEAD~1 HEAD`였습니다. 마지막 커밋 하나만 비교하는 방식입니다. `4520dc9` vs `027155e`만 봤으니 `vault/push/` 변경을 놓쳤습니다. 워크플로는 "success"로 끝났지만 발송 step은 조용히 skip됐습니다.

수정은 간단했습니다. `HEAD~1 HEAD` 대신 `github.event.before..HEAD`를 쓰면 해당 push에 포함된 모든 커밋을 통째로 비교합니다. `fetch-depth: 0`으로 전체 히스토리를 내려받아야 합니다.

처음부터 이렇게 짰으면 됐는데, 단일 커밋 push로 테스트했던 게 문제였습니다. 에이전트가 실제 운영에서 어떻게 commit을 구성하는지 먼저 확인했어야 했습니다.

## 채널 chat_id 찾는 법

새 채널을 만들고 chat_id를 찾는 과정도 예상 밖이었습니다. 흔히 쓰는 `getUpdates` API가 채널에서는 빈 배열을 반환했습니다.

`getChat`을 쓰면 됩니다.

```
https://api.telegram.org/bot{TOKEN}/getChat?chat_id=@채널명
```

`result.id` 필드가 chat_id입니다. 채널은 `-100`으로 시작하는 음수가 나옵니다.

## 이 패턴이 쓸 곳

텔레그램에만 국한된 이야기가 아닙니다. CCR 환경에서 외부 API를 직접 부를 수 없는 경우라면 어디든 같은 구조로 풀 수 있습니다.

- Slack: `vault/push/slack-YYYY-MM-DD.md` + `slackapi/slack-github-action`
- Discord, Notion, 사내 API — 모두 동일 패턴

에이전트가 파일을 쓰면 Actions가 읽는다. 이 한 문장으로 요약됩니다.
