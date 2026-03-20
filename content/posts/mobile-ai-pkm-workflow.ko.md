---
title: "서버 없이 갤럭시 폰 하나로 AI PKM 워크플로우 만들기"
date: 2026-03-20
tags: ["termux", "claude-code", "obsidian", "android", "PKM", "모바일"]
description: "갤럭시 폰에서 Termux + Claude Code + Obsidian 조합으로 PC 없이 AI가 노트를 읽고 쓰고 정리하는 워크플로우를 구축한 경험기."
summary: "PC도 서버도 SSH도 없이, 갤럭시 폰 하나에서 AI가 노트를 읽고 쓰고 정리합니다. Termux + Claude Code + Obsidian으로 만든 모바일 완결형 워크플로우 이야기."
showToc: true
TocOpen: true
draft: false
---

AI가 내 노트를 직접 읽고, 정리하고, 써주는 환경을 오래전부터 원했습니다. 이동 중에 떠오른 생각을 바로 AI한테 넘기고, 기존에 쌓아둔 노트와 연결하고, 필요하면 새로운 글로 정리해주는 그런 흐름이요. 근데 찾아보면 대부분의 가이드가 SSH 접속이나 데스크톱 환경을 전제하고 있었습니다. 결국 노트북이 있어야 가능한 이야기였고, 모바일에서 떠오른 아이디어가 지식베이스로 연결되지 못하는 단절이 계속 아쉬웠습니다.

물론 맥이 켜져 있는 상태라면 Tailscale 같은 VPN으로 원격 접속해서 쓸 수도 있습니다. 실제로 그렇게 쓰기도 했고요. 다만 갤럭시에서 Tailscale을 켜면 삼성페이가 동작하지 않는 문제가 있어서, 결제 때마다 VPN을 끄고 켜는 게 은근히 번거로웠습니다. 무엇보다 맥이 꺼져 있으면 아무것도 할 수 없다는 근본적인 한계가 있었습니다.

그래서 아예 폰 자체에서 돌리는 방법을 만들어보기로 했습니다.

## 최종 구성

갤럭시 폰에서 Termux + Claude Code + FolderSync + Obsidian 조합으로 구성했습니다. 서버도, PC도, SSH도 필요 없는 폰 단독 완결형입니다.

```
Galaxy 폰 하나
├── Termux + Claude Code  →  AI가 노트를 읽고/쓰고/정리
├── FolderSync            →  로컬 ↔ Google Drive 양방향 동기화
└── Obsidian 모바일        →  결과물 열람/편집
```

## 셋업 과정

### Termux와 Claude Code 설치

Termux는 Play Store가 아닌 **F-Droid**에서 설치해야 합니다. Play Store 버전은 업데이트가 중단된 상태입니다.

```bash
pkg update && pkg upgrade
pkg install nodejs
npm install -g @anthropic-ai/claude-code
```

여기까지는 수월했습니다.

### `/tmp` 권한 문제 해결

Claude Code 자체는 정상적으로 실행되지만, Bash 도구가 모두 실패하는 현상이 있었습니다.

```
EACCES: permission denied, mkdir '/tmp/claude-10584'
```

Android의 루트(`/`) 파일시스템은 읽기전용이라 `/tmp` 디렉토리를 생성할 수 없고, Claude Code가 내부적으로 `/tmp`를 직접 참조하기 때문에 환경변수(`TMPDIR`)로는 우회가 되지 않았습니다. 심볼릭 링크도 같은 이유로 불가능했습니다.

해결 방법은 `termux-chroot`였습니다.

```bash
pkg install proot
termux-chroot
```

`termux-chroot`가 가상 루트 환경을 만들어서 `/tmp`가 존재하는 것처럼 동작하게 해줍니다. 알고 나면 간단한데, 이 방법을 찾기까지 꽤 헤맸습니다.

### 함께 설치하면 좋은 패키지

```bash
pkg install git ripgrep tmux
```

`git`은 AI가 vault 파일을 수정하기 때문에 변경 이력 관리와 복구 용도로 필수적이고, `ripgrep`은 Claude Code가 파일 검색에 사용하는 도구라 설치해두는 게 좋습니다. `tmux`는 화면이 꺼져도 세션을 유지해주는데, 모바일 환경에서는 이게 없으면 작업 도중 세션이 끊기는 경우가 잦습니다.

### vault 연결

```bash
termux-setup-storage
git config --global --add safe.directory /storage/emulated/0/Documents/my-vault
```

FolderSync 앱으로 Google Drive와 로컬 디렉토리를 양방향 동기화하면, Obsidian에서 작성한 노트를 Claude Code가 바로 읽고 수정할 수 있는 상태가 됩니다.

### 매일 실행하는 두 줄

```bash
termux-chroot
claude
```

## 실제로 어떻게 쓰고 있는지

원래 원했던 건 거창한 자동화가 아니었습니다. 폰에서 떠오른 아이디어를 수거하고, 기존 컨텍스트에 누적해서 재활용할 수 있는 정도면 충분했습니다.

지금은 이동 중에 떠오른 생각을 Claude한테 "Inbox에 정리해줘"라고 전달하는 것부터, vault 폴더 구조 자동 생성, 기존 노트 분석을 통한 연결 관계 제안, 블로그 글 초안 작성까지 활용하고 있습니다. 이 글 자체도 이 워크플로우에서 시작되었습니다.

## 한계와 보완점

모바일 키보드로 긴 명령을 입력하는 건 여전히 불편합니다. 블루투스 키보드가 있으면 나은 편입니다. FolderSync의 동기화 타이밍에 따라 충돌이 생길 수 있고, Termux가 백그라운드에서 종료되면 세션이 날아가기도 합니다. tmux로 어느 정도 방어가 되지만 완벽하지는 않습니다.

데스크톱이 있으면 당연히 더 편합니다. 하지만 **없어도 가능하다**는 점이 이 구성의 의미라고 생각합니다. 오래 만들어보고 싶었던 환경을 드디어 갖추게 됐습니다.
