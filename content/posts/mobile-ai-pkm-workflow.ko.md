---
title: "서버 없이 갤럭시 폰 하나로 AI PKM 워크플로우 만들기"
date: 2026-03-20
tags: ["termux", "claude-code", "obsidian", "android", "PKM", "모바일"]
description: "갤럭시 폰에서 Termux + Claude Code + Obsidian 조합으로 PC 없이 AI가 노트를 읽고 쓰고 정리하는 워크플로우를 구축한 경험기."
summary: "PC도 서버도 SSH도 없이, 갤럭시 폰 하나에서 AI가 내 노트를 읽고 쓰고 정리합니다. Termux + Claude Code + Obsidian 삽질기."
showToc: true
TocOpen: true
draft: false
---

## TL;DR

갤럭시 폰에서 Termux + Claude Code + FolderSync + Obsidian 조합으로, PC도 서버도 SSH도 없이 **폰 하나로** AI가 노트를 읽고 쓰고 정리하는 환경을 만들었습니다.

## 왜 만들었나

AI가 내 노트를 직접 읽고, 정리하고, 써주는 환경을 오래전부터 원했습니다. 근데 찾아보면 전부 이런 조건이 붙어 있었습니다.

- "SSH로 Mac에 접속해서..."
- "데스크톱에서 실행하세요"
- "서버를 띄워야 합니다"

결국 PC가 필요했습니다. 제가 원한 건 **이동 중에 떠오른 생각을 폰에서 바로 AI한테 넘기는 것**이었는데, 그걸 하려면 항상 노트북이 있어야 했습니다. 모바일에서 떠오른 아이디어를 바로 지식베이스로 연결하지 못하는 단절이 계속 답답했습니다.

폰에서 직접 돌리는 방법은 아무도 알려주지 않았습니다. 그래서 직접 만들었습니다.

## 최종 구성

```
Galaxy 폰 하나
├── Termux + Claude Code  →  AI가 노트를 읽고/쓰고/정리
├── FolderSync            →  로컬 ↔ Google Drive 양방향 동기화
└── Obsidian 모바일        →  결과물 열람/편집
```

서버 비용 $0. PC 불필요. 폰 하나로 완결됩니다.

## 셋업

### Termux + Claude Code 설치

Termux는 **F-Droid**에서 설치합니다. Play Store 버전은 업데이트가 끊겼습니다.

```bash
pkg update && pkg upgrade
pkg install nodejs
npm install -g @anthropic-ai/claude-code
```

여기까지는 순조로웠습니다.

### `/tmp` 권한 — 여기서 3시간 날렸습니다

Claude Code를 실행하면 뜨긴 뜹니다. 근데 Bash 도구가 전부 실패합니다.

```
EACCES: permission denied, mkdir '/tmp/claude-10584'
```

Android는 루트(`/`) 파일시스템이 읽기전용이라 `/tmp`를 만들 수가 없습니다. Claude Code가 내부적으로 `/tmp`를 직접 참조하기 때문에, 환경변수로는 우회가 안 됩니다.

시도했지만 안 된 것들입니다:

| 시도 | 결과 |
|------|------|
| `export TMPDIR=$PREFIX/tmp` | Claude Code가 `/tmp` 직접 참조 → 무시됨 |
| `TMPDIR=$PREFIX/tmp claude` | 동일하게 실패 |
| `ln -sf $PREFIX/tmp /tmp` | 루트가 읽기전용이라 심링크 불가 |

환경변수도, 심링크도 안 돼서 진짜 막막했습니다.

**해결: `termux-chroot`**

```bash
pkg install proot
termux-chroot
```

`termux-chroot`가 가상 루트 환경을 만들어서 `/tmp`가 존재하는 것처럼 해줍니다. 이걸 찾기까지 3시간이 걸렸는데, 찾고 나니 허무할 정도로 간단했습니다.

### 필수 패키지

```bash
pkg install git ripgrep tmux
```

| 패키지 | 왜 필요한가 |
|--------|------------|
| **git** | AI가 vault를 건드리기 때문에 실수 복구용으로 필수입니다 |
| **ripgrep** | Claude Code가 파일 검색에 사용합니다. 없으면 느려집니다 |
| **tmux** | 화면 꺼져도 세션을 유지해줍니다. 모바일에서 이거 없으면 진행이 안 됩니다 |

### vault 연결

```bash
termux-setup-storage
git config --global --add safe.directory /storage/emulated/0/Documents/my-vault
```

FolderSync 앱으로 Google Drive ↔ 로컬을 양방향 동기화하면 끝입니다. Obsidian에서 쓴 노트를 Claude Code가 바로 읽고 수정할 수 있게 됩니다.

### 실행 순서

```bash
termux-chroot    # chroot 진입
claude           # Claude Code 실행
```

매번 이 두 줄이면 됩니다.

## 이걸로 뭘 하고 있나

- 이동 중에 떠오른 아이디어 → Claude한테 "이거 Inbox에 정리해줘" 한마디면 끝입니다
- vault 구조 자동 생성. 폴더, frontmatter 일괄 적용
- 기존 노트를 분석해서 연결 관계를 제안받습니다
- 블로그 글 초안 작성 — 이 글 자체가 이 워크플로우에서 나왔습니다

원래 원했던 건 거창한 24시간 AI 자동화가 아니었습니다. 폰에서 떠오른 아이디어를 수거하고, 기존 컨텍스트에 누적해서 재활용할 수 있는 환경. 딱 그 정도가 됐습니다.

## 솔직한 한계

- 모바일 키보드로 긴 명령을 치는 건 고통입니다. 블루투스 키보드를 추천합니다.
- FolderSync 동기화 타이밍에 따라 충돌이 생길 수 있습니다.
- Termux가 백그라운드에서 죽으면 세션이 날아갑니다. tmux로 해결은 되지만 가끔 까먹습니다.
- 데스크톱이 있으면 당연히 더 편합니다.

근데 **없어도 된다**는 게 핵심입니다. 갤럭시 하나, 약간의 삽질, 해보겠다는 의지면 충분합니다.

오래 하고 싶었는데 못 했던 걸 드디어 해냈습니다.
