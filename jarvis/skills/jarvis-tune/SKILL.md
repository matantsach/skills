---
name: jarvis-tune
description: >-
  Curates the local memory that Jarvis learns from, so its triage stays accurate and noise
  drops over time. Reads the append-only activity log, distills what the user consistently
  ignores or acts on into provenance-backed rules, and runs health checks (contradictions,
  stale claims, missing evidence, gaps). Use when triage surfaces junk or misses people, or
  the user says "reduce the noise", "you keep showing me X", "stop surfacing Y", "learn my
  preferences", "tune my inbox/Teams", "clean up my rules"; also runs weekly on a schedule.
  Interactive: shows a diff and applies on confirmation. Unattended: writes proposals for
  review and never hard-mutes a source or adds a VIP on its own. Auxiliary companion to the
  main jarvis skill.
allowed-tools:
  # Built-in file tools — read the log and groom the wiki in STATE_DIR
  - Read
  - Write
  - Edit
  - Bash               # optional: git commit STATE_DIR if you version it
  # Optional: apply a mute / filter at the source (rename to your server's tools)
  - mcp__teams__update_channel_notifications
  - mcp__outlook__create_inbox_rule
  - mcp__teams__send_chat_message       # deliver the proposal summary to your self-chat
metadata:
  version: 0.5.0
---

# Jarvis — Tune (memory curator)

Jarvis's explicit "learn & adapt" step, and the one piece of maintenance it can't safely do
mid-triage. It turns raw behavior (`log.md`) into reliable, provenance-backed pages
(`wiki/*.md`) and keeps the wiki healthy so **jarvis** keeps scoring well. This is what makes
a generic assistant feel personal — and stay *correct* — after a week of use.

Read **[../../shared/memory.md](../../shared/memory.md)** for `STATE_DIR`, the layers,
provenance conventions, and the two execution modes. This skill is the **`lint`/curate**
client of that memory.

## When to use
- Triage is noisy or misses people → "tune", "reduce the noise", "you keep showing me X",
  "stop surfacing Y", "learn my preferences", "clean up my rules".
- A weekly hygiene habit (see **[../../routines/PROMPTS.md](../../routines/PROMPTS.md)**).

## When not to use
- A one-off "mute this channel" — **jarvis** handles single corrections inline as it learns.

## Steps

Copy this checklist and track progress:
```
- [ ] 1 Read the log since the cursor
- [ ] 2 Distill patterns (evidence + confidence)
- [ ] 3 Lint the wiki
- [ ] 4 Group the diff (safe vs. stronger)
- [ ] 5 Apply per mode + policy
- [ ] 6 Regenerate rules + index
- [ ] 7 Optional source actions
- [ ] 8 Refresh voice / log / cursor
```

1. **Read** `log.md` since the `jarvis-tune` cursor, plus the current `wiki/` pages +
   `index.md`.
2. **Distill** patterns by sender, channel, and topic, each with **evidence + confidence**:
   consistently ignored/archived/muted → down-weight or mute; consistently replied-to fast →
   up-weight or VIP; recurring noise phrases in archived broadcasts → topic = noise. Require
   several consistent examples before proposing a change.
3. **Lint the wiki** (the reliability pass — flag, then fix): contradictions between
   pages/rules; stale claims not confirmed in 30d (unless `conf: high`); orphan/low-evidence
   rules; missing provenance; gaps (frequent senders/channels absent from the wiki); index
   drift.
4. **Group the diff**: "safe" (down-weights, FYI demotions, provenance/index fixes,
   stale-prune) vs. "stronger" (hard mutes, new VIPs).
5. **Apply per mode + `profile.automation.memory_writes`**, presented as a "what I learned
   about you" changelog in the shape of
   **[reference/proposal-format.md](reference/proposal-format.md)**:
   - *Interactive*: show the changelog/diff; apply what the user approves.
   - *Unattended + `propose-only`* (default): write the diff to `proposals.md`, deliver the
     changelog to `brief_destination`, change nothing else.
   - *Unattended + `apply-safe`*: apply the "safe" group; note what was applied vs. left;
     leave "stronger" items in `proposals.md`.
   Never apply "stronger" changes (hard mutes, new VIPs) unattended.
6. **Regenerate `rules.md`** as the distilled set (merge; preserve hand-edits) and refresh
   `index.md` for whatever was applied.
7. **Optional source actions** (only when applied + allowed): mute the Teams channel or add
   an Outlook inbox rule so the noise stops at the source.
8. **Refresh voice / log / cursor.** Add 1–2 newly approved replies to `voice.md` (keep ~5);
   append a `lint` entry to `log.md`; advance the `jarvis-tune` cursor; commit if versioned.

## Running unattended (scheduled Workflow)
Conservative by default: with `propose-only`, this only reads history and *writes proposals* —
triage behavior is unchanged until you approve. **Never hard-mute or add a VIP unattended** —
a wrong mute hides something important. Deliver the proposal summary to `brief_destination`
and persist the wiki so proposals + cursor survive to your review. Every applied change is a
plain diff (and a git commit, if you version `STATE_DIR`) you can revert.

## Guardrails
- Diff before any write; confirm before any source mute.
- Provenance-first: never write a rule you can't back with evidence from `log.md`.
- Local + private; operates only on your `STATE_DIR` and your own sources.

## Adapt to your setup
Rename the optional `mcp__*` mute/rule tools to your connectors. Without them, tuning still
works — it grooms the wiki so **jarvis** filters in-app instead of at the source.
