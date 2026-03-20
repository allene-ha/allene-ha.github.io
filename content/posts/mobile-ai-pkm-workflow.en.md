---
title: "Building an AI PKM Workflow with Just a Galaxy Phone — No Server, No PC"
date: 2026-03-20
tags: ["termux", "claude-code", "obsidian", "android", "PKM", "mobile"]
description: "How I built a fully mobile AI-powered note management workflow using Termux + Claude Code + Obsidian on a Galaxy phone, with no PC or server required."
summary: "No PC, no server, no SSH. Just a Galaxy phone running AI that reads, writes, and organizes my notes. A Termux + Claude Code + Obsidian story."
showToc: true
TocOpen: true
draft: false
---

## TL;DR

I built a workflow on a Galaxy phone where AI reads, writes, and organizes my notes using Termux + Claude Code + FolderSync + Obsidian. **No PC, no server, no SSH.**

## Why I Built This

I've wanted an environment where AI directly reads, organizes, and writes my notes for a long time. But every guide I found came with strings attached:

- "SSH into your Mac..."
- "Run this on your desktop"
- "You need to spin up a server"

It always came back to needing a PC. What I actually wanted was to **hand off thoughts to AI the moment they came to me, on my phone, while on the move.** But that always required a laptop somewhere. The constant disconnect — ideas popping up on mobile with no way to pipe them into my knowledge base — was genuinely frustrating.

Nobody had figured out how to run it directly on a phone. So I built it myself.

## The Final Setup

```
Galaxy Phone
├── Termux + Claude Code  →  AI reads/writes/organizes notes
├── FolderSync            →  Local ↔ Google Drive two-way sync
└── Obsidian Mobile       →  View and edit results
```

Server cost: $0. No PC needed. One phone, fully self-contained.

## Getting It Running

### Termux + Claude Code

Install Termux from **F-Droid**. The Play Store version is no longer maintained.

```bash
pkg update && pkg upgrade
pkg install nodejs
npm install -g @anthropic-ai/claude-code
```

Smooth sailing up to this point.

### The `/tmp` Problem — 3 Hours I Won't Get Back

Claude Code launches fine. But every Bash tool call fails:

```
EACCES: permission denied, mkdir '/tmp/claude-10584'
```

Android's root (`/`) filesystem is read-only, so you can't create `/tmp`. Claude Code internally hard-codes `/tmp`, meaning environment variable workarounds don't work.

Things I tried that didn't work:

| Attempt | Result |
|---------|--------|
| `export TMPDIR=$PREFIX/tmp` | Claude Code references `/tmp` directly → ignored |
| `TMPDIR=$PREFIX/tmp claude` | Same failure |
| `ln -sf $PREFIX/tmp /tmp` | Root is read-only → can't create symlink |

Environment variables didn't work. Symlinks didn't work. I was completely stuck.

**The fix: `termux-chroot`**

```bash
pkg install proot
termux-chroot
```

`termux-chroot` creates a virtual root environment where `/tmp` appears to exist. It took 3 hours to find this. Once I did, it was embarrassingly simple.

### Essential Packages

```bash
pkg install git ripgrep tmux
```

| Package | Why It's Needed |
|---------|----------------|
| **git** | AI modifies your vault, so you need rollback capability. Non-negotiable |
| **ripgrep** | Claude Code uses it for file search. Without it, search becomes painfully slow |
| **tmux** | Keeps sessions alive when screen turns off. Nothing works on mobile without this |

### Connecting the Vault

```bash
termux-setup-storage
git config --global --add safe.directory /storage/emulated/0/Documents/my-vault
```

Set up two-way sync between Google Drive and local storage using FolderSync. That's it. Claude Code can now read and modify the same notes Obsidian sees.

### Daily Startup

```bash
termux-chroot    # Enter chroot
claude           # Launch Claude Code
```

Two lines, every time.

## What I Actually Do With This

- An idea pops up while commuting → "Hey Claude, file this in my Inbox" — done
- Vault structure auto-generation: folders, frontmatter, all in bulk
- Analyze existing notes and get connection suggestions
- Blog draft writing — this very post came out of this workflow

What I wanted was never some grand 24/7 AI automation. Just an environment where ideas could be captured on my phone and accumulated into my existing knowledge base. That's exactly what this became.

## Honest Limitations

- Typing long commands on a mobile keyboard is painful. I'd recommend a Bluetooth keyboard.
- FolderSync timing can cause sync conflicts.
- If Termux gets killed in the background, your session is gone. tmux helps, but I forget sometimes.
- A desktop is obviously more comfortable.

But the core point is: **you don't need one.** A Galaxy phone, some debugging patience, and the willingness to try is all it takes.

I finally built the thing I've been wanting to build for a long time.
