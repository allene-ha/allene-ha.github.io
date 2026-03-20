---
title: "Building an AI PKM Workflow with Just a Galaxy Phone — No Server, No PC"
date: 2026-03-20
tags: ["termux", "claude-code", "obsidian", "android", "PKM", "mobile"]
description: "How I built a fully mobile AI-powered note management workflow using Termux + Claude Code + Obsidian on a Galaxy phone, with no PC or server required."
summary: "No PC, no server, no SSH. Just a Galaxy phone running AI that reads, writes, and organizes my notes. A Termux + Claude Code + Obsidian setup guide."
showToc: true
TocOpen: true
draft: false
---

## TL;DR

On a Galaxy phone, I built a workflow where AI reads, writes, and organizes my notes using Termux + Claude Code + FolderSync + Obsidian. **No PC, no server, no SSH.**

## Why I Built This

I've always wanted AI to directly read, organize, and write my notes. But every guide I found came with conditions:

- "SSH into your Mac..."
- "Run this on your desktop"
- "You need to spin up a server"

It always came back to needing a PC. Nobody explained how to **run it directly on a phone**. So I built it myself.

## The Final Setup

```
Galaxy Phone
├── Termux + Claude Code  →  AI reads/writes/organizes notes
├── FolderSync            →  Local ↔ Google Drive two-way sync
└── Obsidian Mobile       →  View and edit results
```

The point: **No PC. No SSH. No server. One phone, fully self-contained.**

## Setup Guide

### Step 1: Install Termux

Install from **F-Droid**, not Google Play. The Play Store version is no longer maintained.

```bash
pkg update && pkg upgrade
```

### Step 2: Install Claude Code

```bash
pkg install nodejs
npm install -g @anthropic-ai/claude-code
```

### Step 3: Fix the `/tmp` Permission Issue — The Longest Debugging Session

Claude Code launches fine, but every Bash tool call fails:

```
EACCES: permission denied, mkdir '/tmp/claude-10584'
```

Android's root (`/`) filesystem is read-only, so `/tmp` can't be created. Claude Code internally references `/tmp` directly, so environment variable workarounds don't work.

**Things I tried that didn't work:**

| Attempt | Result |
|---------|--------|
| `export TMPDIR=$PREFIX/tmp` | Claude Code references `/tmp` directly → ignored |
| `TMPDIR=$PREFIX/tmp claude` | Same failure |
| `ln -sf $PREFIX/tmp /tmp` | Root is read-only → can't create symlink |

**The fix: `termux-chroot`**

```bash
pkg install proot
termux-chroot
```

`termux-chroot` creates a virtual root environment where `/tmp` appears to exist. After entering chroot, add to `.bashrc`:

```bash
export TMPDIR=$PREFIX/tmp
```

Claude Code works after this.

### Step 4: Essential Packages

```bash
pkg install git ripgrep tmux
```

| Package | Why |
|---------|-----|
| **git** | Track changes. Essential for rollbacks when AI modifies your vault |
| **ripgrep** | Used by Claude Code for file search. Without it, search performance degrades |
| **tmux** | Keep sessions alive when screen turns off. Critically important on mobile |

### Step 5: Connect Your Obsidian Vault

```bash
termux-setup-storage
git config --global --add safe.directory /storage/emulated/0/Documents/my-vault
```

Use FolderSync to set up two-way sync between Google Drive and a local directory. Claude Code can then directly read and modify notes that Obsidian sees.

### Quick Start Summary

```bash
pkg install proot git ripgrep tmux   # One-time setup
termux-chroot                        # Enter chroot
claude                               # Launch Claude Code
```

## What Can You Actually Do With This?

What I've done so far with this workflow:

- **Quick idea capture** → Claude formats it using vault templates
- **Existing note analysis** → Suggests connections, detects duplicates
- **Vault structure automation** → Folder creation, bulk frontmatter application
- **Blog draft writing** → This very post was written using this workflow

## Comparison With Traditional Approaches

| | Traditional | This Approach |
|---|---|---|
| **Hardware** | Mac/PC + Phone | Phone only |
| **SSH** | Required | Not needed |
| **Server** | Sometimes needed | Not needed |
| **PC dependency** | Required | Fully independent |
| **Setup difficulty** | Medium-High | Medium (includes debugging) |

## Limitations

Honestly, there are rough edges:

- Typing long commands on a mobile keyboard is painful (Bluetooth keyboard recommended)
- FolderSync timing can cause sync conflicts
- If Termux gets killed in the background, your session is gone (tmux solves this)

But the core point stands: **you don't need a PC**. A Galaxy phone, some debugging patience, and the willingness to try is all it takes.
