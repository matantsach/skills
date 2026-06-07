# Workflow / Routine prompts (copy-paste)

In Claude Code's **Workflows / Routines**, *"the prompt is the most important part: the
routine runs autonomously, so the prompt must be self-contained and explicit about what to do
and what success looks like."* These are written that way.

> These assume **Local** Workflows (Desktop scheduled tasks that run on your machine). Your
> `~/.jarvis/` wiki persists between runs automatically — no git or "unrestricted push"
> needed. `git init` it only if you want version history.

## Setup checklist (once)
1. **Install the plugin** so the agent can discover its skills:
   ```text
   /plugin marketplace add matantsach/jarvis
   /plugin install jarvis@jarvis
   ```
   On enable you'll be asked for your **Azure DevOps org** (`ado_org`) — fill it in if you
   want work-item creation + `weekly-delivery-report`, and run `az login` once so the bundled
   `ado` MCP server can authenticate. No ADO? Leave it blank and decline the `ado` server.
2. **Onboard once interactively** (chat/CLI): *"triage my Outlook and Teams"* → Jarvis walks
   you through seeding `~/.jarvis/` (the wiki) and the `automation` block. Start with
   `unattended_mode: draft-only`.
3. **Create the Workflow** (Desktop → Workflows → New → **Local**):
   - **Connectors**: include only Outlook + Teams (add ADO/GitHub only if you let it create
     tasks). Unattended runs use connectors with full write access and no prompt — scope tight.
   - **Trigger / cadence**: see below.

---

## jarvis — morning brief
**Cadence:** Daily, weekdays, ~08:00. (Or hourly during work hours; it's incremental.)

```
Use the jarvis skill. The wiki is ~/.jarvis; if profile.md is missing, send a one-line note
to my Teams self-chat and stop.

Run unattended, draft-only (read profile.automation). Query the wiki (index.md, profile,
wiki/ pages), then collect Outlook + Teams items newer than the jarvis cursor (else last 16h).
Cluster noisy group threads into one summary each. Rank Act now / Today / FYI / Muted with a
one-line "why" per surfaced item — score from the compiled wiki pages, not the raw log.

For each "Act now" item leave a reply draft in Outlook Drafts in my voice — do NOT send and do
NOT post to any Teams channel. Deliver the brief to profile.automation.brief_destination.

Ingest: append a run entry + decisions to log.md, update affected wiki pages with provenance,
refresh index.md, advance the jarvis cursor (commit if ~/.jarvis is a git repo). If a connector
is empty or errors, say so in the brief and exit cleanly. Success = brief delivered and wiki
updated.
```

## jarvis-tune — weekly noise cleanup + wiki lint
**Cadence:** Weekly, Monday ~07:30.

```
Use the jarvis-tune skill in propose-only mode. The wiki is ~/.jarvis. Read log.md since the
jarvis-tune cursor; distill senders/channels/topics I consistently ignore or act on (with
evidence + confidence). Then run the lint pass: flag contradictions, stale claims (>30d,
conf<high), orphan/low-evidence rules, missing provenance, gaps, and index drift.

Write the proposed diff to proposals.md and deliver a short summary to my brief_destination.
Do NOT modify the wiki and do NOT mute any source. Append a lint entry to log.md, advance the
cursor (commit if versioned). If there isn't enough new data, say so and exit. Success =
proposals.md updated (or "no change") and a summary delivered.
```

## weekly-delivery-report — stakeholder update
**Cadence:** Weekly, Friday ~16:00 (and/or a GitHub `release` trigger).

```
Use the weekly-delivery-report skill. The wiki is ~/.jarvis; read profile.delivery (team,
github repos, azure_devops project/team/iteration, distribution_list) and profile.automation.
If profile.delivery is missing, send a one-line note to my Teams self-chat and stop.

Run unattended, draft-only. Window = since the weekly-delivery-report cursor (else last 7d).
Gather: merged PRs + CI health + releases (GitHub) across my repos; completed-vs-planned work
items + burndown + blocked items (Azure DevOps, current iteration). Synthesize a stakeholder
update (BLUF → Shipped → In flight → Risks/incidents → Next week → metrics), outcome-first.

Leave it as an Outlook DRAFT to distribution_list and deliver the Teams post draft text to
brief_destination — do NOT send and do NOT post to any channel. Append a report entry to
log.md and advance the cursor (commit if versioned). If a source is empty/errored, say so in
the update and continue. Success = drafts created and run logged.
```

---

## Roadmap prompts (skills not built yet)

**alert-triage** — Trigger: API (`/fire`) from monitoring; alert body arrives as `text`.
```
An alert arrived in the run input (text). Correlate the stack trace with recent commits, open
a DRAFT pull request with a proposed fix on a claude/ branch, and post a one-line status + PR
link to the incident Teams channel only if profile.automation.may_post_to_teams_channels is
true; otherwise leave a draft note to self. Success = draft PR opened and linked.
```
