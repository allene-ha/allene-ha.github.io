---
title: "서버 없이 갤럭시 폰 하나로 AI PKM 워크플로우 만들기"
date: 2026-03-20
tags: ["termux", "claude-code", "obsidian", "android", "PKM", "모바일"]
description: "갤럭시 폰에서 Termux + Claude Code + Obsidian 조합으로 PC 없이 AI가 노트를 읽고 쓰고 정리하는 워크플로우를 구축한 경험기."
summary: "PC도 서버도 SSH도 없이, 갤럭시 폰 하나에서 AI가 내 노트를 읽고 쓰고 정리한다. Termux + Claude Code + Obsidian 삽질기."
showToc: true
TocOpen: true
draft: false
---

## TL;DR

갤럭시 폰에서 Termux + Claude Code + FolderSync + Obsidian 조합으로, PC도 서버도 SSH도 없이 **폰 하나로** AI가 내 노트를 읽고 쓰고 정리하는 워크플로우를 만들었다.

## 왜 이걸 만들었나

AI가 내 노트를 직접 읽고, 정리하고, 써주는 워크플로우. 오래전부터 하고 싶었다. 근데 찾아보면 대부분 이런 조건이 붙었다:

- "SSH로 Mac에 접속해서..."
- "데스크톱에서 실행하세요"
- "서버를 띄워야 합니다"

결국 PC가 필요했다. **폰에서 직접 돌리는 방법**은 아무도 안 알려줬다. 그래서 직접 만들었다.

## 최종 구성

```
Galaxy 폰 하나
├── Termux + Claude Code  →  AI가 노트를 읽고/쓰고/정리
├── FolderSync            →  로컬 ↔ Google Drive 양방향 동기화
└── Obsidian 모바일        →  결과물 열람/편집
```

핵심: **PC 없이, SSH 없이, 서버 없이. 폰 하나로 완결.**

## 셋업 가이드

### 1단계: Termux 설치

Google Play가 아닌 **F-Droid**에서 설치한다. Play Store 버전은 업데이트가 중단되었다.

```bash
# Termux 실행 후 기본 패키지 업데이트
pkg update && pkg upgrade
```

### 2단계: Claude Code 설치

```bash
pkg install nodejs
npm install -g @anthropic-ai/claude-code
```

### 3단계: `/tmp` 권한 문제 해결 — 가장 오래 걸린 삽질

Claude Code를 실행하면 멀쩡하게 뜨는데, Bash 도구가 전부 실패한다.

```
EACCES: permission denied, mkdir '/tmp/claude-10584'
```

Android는 루트(`/`) 파일시스템이 읽기전용이라 `/tmp` 디렉토리를 만들 수 없다. Claude Code는 내부적으로 `/tmp`를 직접 참조하기 때문에, 환경변수로 우회가 안 된다.

**시도했지만 안 된 것들:**

| 시도 | 결과 |
|------|------|
| `export TMPDIR=$PREFIX/tmp` | Claude Code 내부에서 `/tmp` 직접 참조 → 무시됨 |
| `TMPDIR=$PREFIX/tmp claude` | 동일하게 실패 |
| `ln -sf $PREFIX/tmp /tmp` | 루트가 읽기전용이라 심링크 생성 불가 |

**해결: `termux-chroot`**

```bash
# proot 설치
pkg install proot

# chroot 환경 진입
termux-chroot
```

`termux-chroot`가 가상 루트 환경을 만들어서 `/tmp`가 존재하는 것처럼 해준다. 진입 후 `.bashrc`에 한 줄 추가:

```bash
export TMPDIR=$PREFIX/tmp
```

이후 Claude Code 정상 작동.

### 4단계: 필수 패키지 설치

```bash
pkg install git ripgrep tmux
```

| 패키지 | 왜 필요한가 |
|--------|------------|
| **git** | 변경사항 추적. AI가 vault를 건드리니까 실수 복구용 필수 |
| **ripgrep** | Claude Code가 파일 검색할 때 사용. 없으면 검색 성능 저하 |
| **tmux** | 화면 꺼져도 세션 유지. 모바일에서 치명적으로 중요 |

### 5단계: Obsidian vault 연결

```bash
# 스토리지 권한 부여
termux-setup-storage

# Git safe.directory 등록 (Android 권한 이슈)
git config --global --add safe.directory /storage/emulated/0/Documents/my-vault
```

FolderSync 앱으로 Google Drive ↔ 로컬 디렉토리를 양방향 동기화하면, Obsidian에서 쓴 노트를 Claude Code가 바로 읽고 수정할 수 있다.

### 실행 순서 요약

```bash
# 매번 이 순서로 시작
pkg install proot git ripgrep tmux   # 최초 1회
termux-chroot                        # chroot 진입
claude                               # Claude Code 실행
```

## 실제로 뭘 할 수 있나

이 워크플로우로 지금까지 해본 것들:

- **빠른 아이디어 캡처** → Claude가 vault 템플릿에 맞춰 정리
- **기존 노트 분석** → 연결 관계 제안, 중복 탐지
- **vault 구조 자동 생성** → 폴더 구조, frontmatter 일괄 적용
- **블로그 글 초안 작성** → 이 글 자체가 이 워크플로우로 작성됨

## 기존 방식과 비교

| | 기존 방식 | 이 방식 |
|---|---|---|
| **필요 장비** | Mac/PC + 폰 | 폰 하나 |
| **SSH** | 필요 | 불필요 |
| **서버** | 필요한 경우 있음 | 불필요 |
| **PC 의존** | 필수 | 완전 독립 |
| **설정 난이도** | 중~상 | 중 (삽질 포함) |

## 한계

솔직히 말하면 불편한 점도 있다:

- 모바일 키보드로 긴 명령 입력이 고통스럽다 (블루투스 키보드 추천)
- FolderSync 동기화 타이밍에 따라 충돌이 생길 수 있다
- Termux 앱이 백그라운드에서 죽으면 세션이 날아간다 (tmux로 해결)

하지만 **PC가 없어도 된다**는 게 핵심이다. 갤럭시 하나, 약간의 삽질, 그리고 해보겠다는 의지면 충분하다.
