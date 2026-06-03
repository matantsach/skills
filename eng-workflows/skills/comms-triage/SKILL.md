---
name: comms-triage
description: >-
  Triage and rank the user's incoming communications across Outlook (email) and
  Microsoft Teams (chats, @mentions, busy group channels) into one short brief of what
  actually needs them. Use when the user feels buried in email or Teams, or asks to
  "catch me up", "what needs my attention", "triage my inbox", "summarize my unread",
  "I have too many messages", "did I miss anything important", or at the start of the
  day. Also runs unattended as a scheduled Workflow. Reads a local, user-owned wiki of
  VIPs, important topics, and muted channels to score importance, collapses noisy group
  threads into a single summary, and suggests one action per item (draft a reply, turn
  into a task, schedule, or archive/mute). Learns from what the user does by maintaining
  that wiki. Read-only and draft-only by default: never sends or posts without consent.
allowed-tools:
  # Built-in file tools — to read/maintain the wiki in STATE_DIR (~/.eng-workflows)
  - Read
  - Write
  - Edit
  - Bash               # optional: git commit STATE_DIR if you version it
  # Outlook — read + draft only (NEVER a send tool here). Rename to your server's tools.
  - mcp__outlook__list_messages
  - mcp__outlook__get_message
  - mcp__outlook__create_draft
  - mcp__outlook__list_events           # optional: today's calendar, for meeting-prep
  # Microsoft Teams — read + (optional) self-chat delivery only
  - mcp__teams__list_chats
  - mcp__teams__list_channel_messages
  - mcp__teams__send_chat_message       # ONLY for delivering the brief to your self-chat
  # Optional: turn an item into a tracked task (only if profile.automation allows)
  - mcp__azure-devops__create_work_item
  - mcp__github__issue_write
metadata:
  version: 0.3.0
---

# Comms Triage

Turn an overwhelming inbox + Teams into one ranked brief of *what needs you* — written
like a good chief-of-staff would: judgment first, bottleneck-first, in your voice — and get
smarter every run by grooming a local wiki.
**Detect mode → query the wiki → collect → cluster → score → enrich → deliver → ingest.**

This skill is a `query`/`ingest` client of the **LLM-Wiki memory** in
**[../../shared/memory-contract.md](../../shared/memory-contract.md)** — read it first for
`STATE_DIR`, the layers, provenance rules, execution modes, and privacy. Score from the
**compiled `wiki/` pages**, never from the raw `log.md`.

## When to use
- "Catch me up", "what needs my attention", "triage my inbox/Teams", "did I miss
  anything", "I'm drowning in messages" — or on a schedule (see `routines/PROMPTS.md`).

## When NOT to use
- Drafting a specific known reply (just write it).
- Recurring team status → `weekly-delivery-report`. "Reduce the noise"/"learn my prefs" →
  `comms-tune` (the wiki curator).

## Steps
1. **Detect mode** (interactive vs Workflow) and **query the wiki**: read `index.md`, then
   `profile.md` and the pages you need (`wiki/people.md`, `channels.md`, `topics.md`,
   `rules.md`, `voice.md`) + `cursors.md`. Missing `profile.md`: interactive → onboard;
   Workflow → deliver "profile not found" and exit.
2. **Collect incrementally** — items newer than the `comms-triage` cursor (else last 16h).
   Outlook: unread + flagged + addressed/cc'd to you. Teams: @mentions, 1:1 + group chats,
   posts in non-muted channels. *(Optional)* today's Outlook calendar for meeting-prep. Map
   each message to the **Item schema**. Note any unavailable source.
3. **Cluster** by `thread_id`; collapse each noisy group thread into ONE item with a 1–2
   line summary + the open question/decision — not every message.
4. **Score** into 🔴/🟠/🟡/⚪ using the wiki; `rules.md` overrides. One-line **why** per 🔴/🟠.
5. **Enrich each surfaced item** so the brief can be useful, not just a list:
   - **action** it needs from you: Decide / Reply / Review / Approve / Unblock.
   - **who's waiting / are you the blocker** (someone asked you directly and you haven't
     replied; you're the owner/assignee) → drives the "🚧 waiting on you" and "Owed" lines.
   - **age** (since received / since they last pinged) and any **explicit deadline** in the
     text ("by EOD", "before Fri"); only state a deadline if it's real.
   - **effort** estimate (~5m quick reply / ~20m / deep) and a **draft** for quick ones.
   - **confidence**: if you're unsure whether to surface or mute something, hold it for the
     brief's "🤔 Wasn't sure" section instead of guessing.
6. **Deliver** in the shape of [templates/brief.md](templates/brief.md): **BLUF** (a 1–2
   line judgment) → pulse chips → a "**Handled:**" line (drafts prepared, auto-archived,
   snoozed) → **Needs you today** (bottleneck-first, numbered) → **Owed** (aging) →
   This week → **Meeting prep** (if calendar) → **Wasn't sure** → **FYI digest** →
   **Filtered** count + footer. Sort "Needs you" by impact, then age. Omit empty sections;
   if nothing needs them, say so plainly.
   - *Interactive*: print it; offer the footer's quick actions.
   - *Workflow*: write it to `profile.automation.brief_destination`. Reply drafts already
     sit in Outlook Drafts.
7. **Act only within consent.** Interactive → do what the user picks after confirming.
   Workflow → only what `automation` allows (default `draft-only`); everything else is a
   *suggestion in the brief*, not an action.
8. **Ingest (learn).** Append a run header + one line per decision to `log.md`; update any
   affected `wiki/` page **with provenance** (evidence + confidence + today's date) and
   refresh `index.md`; advance the `comms-triage` cursor. If `STATE_DIR` is a git repo,
   commit. Apply explicit corrections immediately (interactive). Suggest `comms-tune` if
   the log has grown a lot since the last lint.

## Running as a scheduled Workflow (unattended)
- **No questions, no approval prompts.** Never block.
- **Draft-only unless pre-authorized** in `automation`.
- **Idempotent & incremental** via `cursors.md`; re-runs must not duplicate drafts —
  de-dupe by `thread_id` + processed time.
- **Deliver, don't print** (no chat surface): route the brief to `brief_destination`
  (a Teams self-chat or an Outlook draft to yourself is a safe default).
- **Self-report** connector gaps/errors in the brief and exit cleanly (green ≠ success).
- **Persist** the wiki to disk (it's local and durable); commit if you version it.

## Guardrails
- Draft-only / read-only by default; no send, auto-post, or merge.
- Snippets, not bodies; never echo secrets. Don't infer mood/sentiment — state facts
  (e.g. "flagged blocked twice"), not feelings.
- Provenance over guesses; explain every surfaced item; hold low-confidence calls for
  "Wasn't sure" rather than acting on them.
- Private, local wiki only.

## Adapt to your team
Rename the `mcp__*` tools to match your installed Outlook/Teams/ADO/GitHub connectors. In
a Workflow this list is your real safety boundary — the platform won't prompt. Tune
priorities and the `automation` policy in `profile.md`; personalization is data, not code.
