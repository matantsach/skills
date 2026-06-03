---
name: comms-tune
description: >-
  The wiki curator for the comms suite — runs the LLM-Wiki "lint" operation to keep the
  user's learned preferences reliable and cut notification noise over time. Use when the
  user says triage is surfacing junk or missing people, or asks to "reduce the noise",
  "you keep showing me X", "stop surfacing Y", "learn my preferences", "tune my
  inbox/Teams", "clean up my rules" — or on a weekly schedule. Reads the append-only log,
  distills consistent behavior into wiki pages with provenance, and runs health checks
  (contradictions, stale claims, orphan/low-evidence entries, missing provenance, gaps).
  Interactive: shows a diff and applies on confirmation. Unattended: writes proposals for
  review and never changes the wiki or mutes a source on its own.
allowed-tools:
  # Built-in file tools — to read the log and groom the wiki in STATE_DIR
  - Read
  - Write
  - Edit
  - Bash               # optional: git commit STATE_DIR if you version it
  # Optional: apply a mute / filter at the source (rename to your server's tools)
  - mcp__teams__update_channel_notifications
  - mcp__outlook__create_inbox_rule
  - mcp__teams__send_chat_message       # deliver the proposal summary to your self-chat
metadata:
  version: 0.3.0
---

# Comms Tune (wiki curator)

The explicit "learn & adapt" step. It is the **`lint` operation** of the LLM-Wiki memory:
turn raw behavior (`log.md`) into reliable, provenance-backed pages (`wiki/*.md`), and
keep the wiki healthy so `comms-triage` scores well. This is what makes the generic triage
feel personal — and stay *correct* — after a week of use.

Read **[../../shared/memory-contract.md](../../shared/memory-contract.md)** for `STATE_DIR`,
the layers, provenance conventions, and execution modes.

## When to use
- Triage is noisy or misses people → "tune", "reduce noise", "you keep showing me X".
- A weekly hygiene habit (see `routines/PROMPTS.md`).

## When NOT to use
- A one-off "mute this channel" — `comms-triage` handles single corrections inline.

## Steps
1. **Read** `log.md` since the `comms-tune` cursor, plus current `wiki/` pages + `index.md`.
2. **Distill** patterns by sender, channel, and topic, each with **evidence + confidence**:
   - consistently `ignored`/`archived`/`muted` → down-weight or mute.
   - consistently `replied` quickly → up-weight or VIP.
   - recurring noise phrases in archived broadcasts → topic = noise.
   Require several consistent examples before proposing a change.
3. **Lint the wiki** (the reliability pass — flag, then fix):
   - **contradictions** between pages or rules,
   - **stale claims** not confirmed in 30d (unless `conf: high`),
   - **orphan / low-evidence** rules (no citation, or evidence below threshold),
   - **missing provenance** (claims without evidence/date),
   - **gaps** (frequent senders/channels absent from the wiki),
   - **index drift** (pages missing from or stale in `index.md`).
4. **Produce a diff** grouped "safe" (down-weights, FYI demotions, provenance/index fixes,
   stale-prune) vs "stronger" (hard mutes, new VIPs).
5. **Apply per mode + `profile.automation.comms_tune_writes`**. Present every proposal as a
   **"what I learned about you" changelog** in the shape of
   [templates/proposal.md](templates/proposal.md):
   - *Interactive*: show the changelog/diff; apply what the user approves.
   - *Unattended + `propose-only`* (default): write the diff to `proposals.md` and deliver
     the changelog to `brief_destination`; **change nothing else**.
   - *Unattended + `apply-safe`*: apply the "safe" group, deliver the changelog noting what
     was applied vs. left for review; leave "stronger" items in `proposals.md`.
   Never apply "stronger" changes (hard mutes, new VIPs) unattended.
6. **Regenerate `rules.md`** as the distilled set (merge; preserve hand-edits) and refresh
   `index.md` for whatever was applied.
7. **Optional source actions** (only when applied + allowed): mute the Teams channel or add
   an Outlook inbox rule so noise stops at the source.
8. **Refresh voice / log / cursor.** Add 1–2 newly approved replies to `voice.md` (keep ~5);
   append a `lint` entry to `log.md`; advance the cursor. Commit `STATE_DIR` if versioned.

## Running as a scheduled Workflow (unattended)
- **Conservative by default.** With `propose-only`, this skill only reads history and
  *writes proposals* — triage behavior is unchanged until you approve.
- **Never hard-mute or add a VIP unattended.** A wrong mute hides something important.
- **Deliver + persist.** Route the proposal summary to `brief_destination`; save the wiki
  (commit if versioned) so proposals and the cursor survive to your review.
- **Reversible & auditable.** Every applied change is a plain diff (and a git commit if you
  version `STATE_DIR`) you can revert.

## Guardrails
- Diff before any write; confirm before any source mute.
- Provenance-first: never write a rule you can't back with evidence from `log.md`.
- Local + private; operates only on your `STATE_DIR` and your own sources.

## Adapt to your team
Rename the optional `mcp__*` mute/rule tools to match your connectors. Without them,
tuning still works — it just grooms the wiki so triage filters in-app instead of at source.
