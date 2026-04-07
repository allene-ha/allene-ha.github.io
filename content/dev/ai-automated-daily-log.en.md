---
title: "How AI Organizes My Day — A Daily Log System That Separates Emotions from Facts"
date: 2026-04-07T00:00:00+09:00
tags: ["AI", "workflow", "daily-log", "Claude-Code", "Obsidian"]
description: "A 3-stage pipeline that separates emotions from facts in daily work logs. Using Claude Code + Obsidian to detect burnout patterns through structured journaling."
series: ["Surviving the AI Era as a Developer"]
ShowToc: true
TocOpen: false
draft: false
---

> This is the second post in the "Surviving the AI Era as a Developer" series.
> In the first post, [Developer's AI Depression](/dev/developer-ai-depression/), I wrote that "what you can observe, you can manage." This post is about how I actually do that observing.

---

At the end of a workday, I can't remember what I did.

More precisely, I vaguely remember what I did, but it's tangled up with **how I felt**. The fact "I spent 3 hours on server configuration" and the emotion "I was frustrated thinking why can't I even do this" merge into one blob. So when I look back, all that remains is "today was hard." I forget what I accomplished; only the exhaustion accumulates.

When this repeats, you fall into a burnout loop. You actually got a lot done, but since only fatigue sticks in memory, you feel like "I accomplished nothing." The fuel for the **scarcity-fulfillment loop** I described in the first post is exactly this distorted memory.

So I built a logging system. The core idea is simple: **separate emotions from facts.**

## Before: Mixed-Up Logs

Earlier this year, when I first started logging, my daily entries looked like this:

```
- Organized notes from feature planning discussion
- Scope interpretation mismatch on document request → rolled back, created separate doc
- Felt pressure about execution density again after reading a colleague's lecture notes
- Improved quality of documentation today, but didn't reach experiments that directly impact key metrics
- Kept getting unwanted fallback logic interfering, felt frustrated
- Couldn't tell if my output was better than the existing approach,
  and continuing to revise the draft in that state made everything feel darker
```

Facts and emotions listed in the same bullet points. When I re-read this the next day, "felt frustrated" and "everything felt darker" dominate the tone. In reality, it was a productive day — I split the documentation into two tracks, organized the verification flow, and submitted a PR.

On the flip side, there were days like this:

```
## What I Did
- Merged main branch and rollback commit
- CS response: attempted DB token lookup → blocked by permission policy
- Tried alternative approaches (multiple paths) → all failed

## Reflection

```

The reflection section is empty. A day where I wrote zero emotions. On days like these, the record exists but there's no reflection, so later I have no idea "what state I was in during this period."

**There are two problems**: either emotions contaminate facts, or emotions go entirely unrecorded.

## The 3-Stage Pipeline

The system I built to solve this operates in three stages.

### Stage 1: Session Wrap-Up — `/session-log`

When a work session ends (usually 2–3 hours), I run `/session-log`. The AI summarizes what happened during the session and appends it to a file called **Daily Buffer**.

The key here is that anything goes in the Buffer. "Spent 3 hours blocked on DB access" goes in. "The yak-shaving was so long I felt self-doubt" goes in too. Categorization happens later. The goal at this stage is **preventing information loss**.

### Stage 2: End of Day — `/daily-log`

At the end of the day, I run `/daily-log`. The AI reads three sources:

1. **Daily Buffer** — accumulated notes from today's sessions
2. **Git log** — today's commit history
3. **Issue tracker** — ticket info extracted from commit messages

### Stage 3: Emotion-Fact Classification

This is the core. The AI classifies each item in the Daily Buffer into four categories:

| Type | Criteria | Destination |
|------|----------|-------------|
| **Fact** | What was done, what was blocked, how it was resolved | `What I Did` / `Issues & Resolutions` |
| **Emotion** | What caused anxiety, what felt rewarding | `Reflection` |
| **Idea** | Spontaneous thoughts or improvement ideas | Individual note in `Inbox/` |
| **Action** | What needs to be done next | `Tomorrow's Tasks` |

When a single sentence mixes fact and emotion, it gets split:

```
"Finished deploying the data pipeline and felt pretty good about it"
→ Fact: "Data pipeline deployment complete"
→ Emotion: "Sense of accomplishment"
```

After classification, the results are shown to me for review and correction. Then each item is automatically placed into the corresponding section of the daily note.

Some days have zero emotions. When neither the Buffer nor session notes contain any emotional expression, the AI runs a short reflection interview:

> "What consumed the most energy today?"
> "How's your condition right now?"
> "Describe today in one word."

A few short answers, and it writes 2–3 lines of reflection. If I say "pass," it stays empty.

## After: Separated Logs

The same day's record transforms into this:

```
## What I Did
### Search Analysis Enhancement
- Implemented search state cloud upload feature (PR submitted)
- Fixed portal data collection failure (PR submitted)

### Research Data Collection
- Site A statistics: 41 stats, 803 rows
- Site B medical info: discovered iframe JSON API
- Site C: bypassed bot detection, collected 304 entries

## Issues & Resolutions
- Site C bot detection → bypassed via hiding navigator.webdriver
- Site C filter parameter error → parameter codes differed from expected
- Site B empty response → discovered separate JSON API inside iframe

## Reflection
Running 7 crawlers in one day, each with different bot detection patterns —
it felt like a game. The unexpected parameter codes were genuinely funny.
Felt proud when I discovered the hidden API.
Heavy volume, but more accomplishment than drain today.
```

The **fact sections** contain only verifiable information — PR references, numbers, file paths. The **reflection section** collects only emotions, making "my state on this day" clearly visible.

## The Effect on Quarterly Reviews

When I wrote my Q1 review using this system, the difference was dramatic.

Reading just the reflection sections from three weeks of daily logs, I could see the emotional trajectory:

- Early W10: "pressure about execution density", "everything felt dark"
- Late W10: "felt good when I drove and the counterpart followed along"
- W11: "relief now that I can edit notes directly from my phone"

Because emotions were isolated in their own sections, just reading them in sequence made "what state was I in during Q1" self-evident. In my Q1 review, I was able to write:

> "FOMO is a loop. Anxiety → overinvestment → short-term relief → raised baseline → greater anxiety. Now that I recognize this structure, don't fall in during Q2."

That single line would not have been possible without emotion data being separated and accumulated over time.

## How It Works Technically

No special infrastructure. It's implemented as **slash commands (skills)** in Claude Code.

- `/session-log`: Summarize current session → append to Daily Buffer
- `/daily-log`: Buffer + git log + issue tracker → classify → generate daily note

Each skill is defined as a prompt in a single markdown file. When Claude Code executes it, it directly reads and writes files in the Obsidian vault. No server, no cron job, no separate app.

Classification accuracy is around 90% in my experience. Occasionally there's a misclassification like "this is emotion, not fact," but since it shows the results and gives me a chance to correct, it's not a major issue. In fact, **the act of reviewing the AI's classification itself becomes a reflection exercise**.

## Why This Helps Prevent Burnout

In the first post, I discussed the scarcity-fulfillment loop. The fuel for that loop is **distorted self-perception**. "I accomplished nothing," "I'm falling behind," "Another wasted day." These thoughts are mostly the result of emotions overwriting facts.

When emotions and facts are separated:

1. **Looking at facts alone** — you can confirm that you actually got a lot done
2. **Looking at emotions alone** — you can see anxiety patterns (when they occur, what triggers them)
3. **Cross-referencing both** — you can distinguish "productive day but anxious" from "unproductive day but calm"

Once this distinction becomes possible, **the reality behind "anxious even while resting"** reveals itself. When the object of anxiety becomes concrete, the response becomes concrete too. Vague dread can't be managed, but "FOMO spikes every Tuesday after peer comparison" is a pattern you can work with.

What you can observe, you can manage.

---

## FAQ

**Q. Do I need coding skills for AI daily log automation?**
A. It's at the level of writing prompts in a markdown file. Since it leverages Claude Code's slash commands (skills), no separate coding is required.

**Q. Can this work with note apps other than Obsidian?**
A. The key requirement is "a structure where you can directly read and write markdown files." Any local markdown-based app like Logseq or Dendron would work with the same approach.

**Q. What if the AI misclassifies emotions vs. facts?**
A. It shows the classification results first and gives you a chance to correct them. In fact, the act of reviewing the AI's classification itself becomes a reflection exercise.
