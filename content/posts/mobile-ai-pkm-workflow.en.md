---
title: "Building an AI PKM Workflow with Just a Galaxy Phone — No Server, No PC"
date: 2026-03-20
tags: ["termux", "claude-code", "obsidian", "android", "PKM", "mobile"]
description: "How I built a fully mobile AI-powered note management workflow using Termux + Claude Code + Obsidian on a Galaxy phone, with no PC or server required."
summary: "No PC, no server, no SSH. Just a Galaxy phone running AI that reads, writes, and organizes my notes. A Termux + Claude Code + Obsidian workflow story."
showToc: true
TocOpen: true
draft: false
---

I've wanted an environment where AI could directly read, organize, and write my notes for a long time. The kind of workflow where ideas that pop up while I'm on the move get handed off to AI right away, connected to existing notes, and shaped into something useful. But most guides I found assumed SSH access or a desktop setup. It always came down to needing a laptop, and that disconnect — ideas surfacing on mobile with no path to my knowledge base — kept bothering me.

Since I couldn't find a way to run it directly on a phone, I decided to build one myself.

## The Setup

The final configuration runs entirely on a Galaxy phone: Termux + Claude Code + FolderSync + Obsidian. No server, no PC, no SSH required.

```
Galaxy Phone
├── Termux + Claude Code  →  AI reads/writes/organizes notes
├── FolderSync            →  Local ↔ Google Drive two-way sync
└── Obsidian Mobile       →  View and edit results
```

## Getting It Running

### Installing Termux and Claude Code

Termux needs to be installed from **F-Droid**, not the Play Store. The Play Store version is no longer maintained.

```bash
pkg update && pkg upgrade
pkg install nodejs
npm install -g @anthropic-ai/claude-code
```

This part went smoothly.

### Fixing the `/tmp` Permission Issue

Claude Code itself launched without problems, but every Bash tool call would fail:

```
EACCES: permission denied, mkdir '/tmp/claude-10584'
```

Android's root (`/`) filesystem is read-only, so `/tmp` can't be created. Since Claude Code internally hard-codes `/tmp`, environment variables like `TMPDIR` didn't help. Symlinks weren't possible for the same reason.

The solution turned out to be `termux-chroot`:

```bash
pkg install proot
termux-chroot
```

`termux-chroot` creates a virtual root environment where `/tmp` appears to exist. It's straightforward once you know about it, but finding it took a fair bit of searching.

### Recommended Packages

```bash
pkg install git ripgrep tmux
```

`git` is essential because AI modifies vault files directly — you need change tracking and rollback capability. `ripgrep` is the search tool Claude Code relies on, so installing it noticeably improves performance. `tmux` keeps sessions alive when the screen turns off, which happens frequently on mobile and would otherwise interrupt work in progress.

### Connecting the Vault

```bash
termux-setup-storage
git config --global --add safe.directory /storage/emulated/0/Documents/my-vault
```

Setting up two-way sync between Google Drive and a local directory via FolderSync means Claude Code can immediately read and modify the same notes that Obsidian sees.

### The Two Lines I Run Every Day

```bash
termux-chroot
claude
```

## How I Actually Use This

What I originally wanted was never some elaborate automation. Just an environment where ideas could be captured on my phone and accumulated into my existing knowledge base — that level was enough.

Now I use it for everything from telling Claude to "file this in my Inbox" while commuting, to auto-generating vault folder structures, getting connection suggestions from existing note analysis, and drafting blog posts. This very post started from this workflow.

## Limitations

Typing long commands on a mobile keyboard is still uncomfortable. A Bluetooth keyboard helps. Sync conflicts can occur depending on FolderSync timing, and if Termux gets killed in the background, the session is lost. tmux provides some defense against this, but it's not perfect.

A desktop is naturally more comfortable. But the point of this setup is that **you don't need one**. I finally have the environment I've been wanting to build for a long time.
