---
name: weekly-delivery-report
description: >-
  Drafts a weekly delivery update for stakeholders from the team's real activity — merged PRs
  and CI health (GitHub), completed-vs-planned work items and burndown (Azure DevOps), and
  notable incidents (Azure, optional). Use when the user asks to "write the weekly update",
  "status report", "delivery report", "what shipped this week", "sprint/iteration summary", or
  "update for my manager/stakeholders"; also runs on a weekly schedule (e.g. Friday) or a
  release trigger. Auxiliary companion to the main jarvis skill. Draft-only: leaves an Outlook
  draft and a Teams post draft for review — never sends or posts without consent.
allowed-tools:
  # Built-in file tools — read config + log the run in STATE_DIR (~/.jarvis)
  - Read
  - Write
  - Edit
  - Bash               # optional: git commit STATE_DIR if you version it
  # GitHub — merged PRs, commits, CI health, releases (connector you already have)
  - mcp__github__search_pull_requests
  - mcp__github__list_pull_requests
  - mcp__github__list_commits
  - mcp__github__actions_list
  - mcp__github__list_releases
  # Azure DevOps — work items + pipeline health (bundled server; set your org in plugin config)
  - mcp__ado__core_list_projects
  - mcp__ado__wit_query_by_wiql
  - mcp__ado__wit_get_work_items_for_iteration
  - mcp__ado__wit_list_backlog_work_items
  - mcp__ado__pipelines_get_builds
  # Delivery — draft only (NEVER a send/post tool). Rename to your Outlook/Teams connectors.
  - mcp__outlook__create_draft
  - mcp__teams__send_chat_message       # ONLY to deliver the draft text to your own self-chat
  # Optional incidents/cost: rename to your Azure monitor connector if you have one.
  # - mcp__azure__monitor_query
metadata:
  version: 0.5.0
---

# Weekly Delivery Report

Turns a week of real engineering activity into one **stakeholder-ready delivery update** —
what shipped, what's in flight against the plan, what's at risk, what's next — drafted, never
sent. The reporting companion to **jarvis**: jarvis manages your incoming comms; this manages
your outgoing status.

It is a **consumer** of the shared memory's `profile.md` (team, repos, ADO project,
distribution list, automation) and execution-mode/draft-only contract — read
**[../../shared/memory.md](../../shared/memory.md)** for `STATE_DIR`, the two execution modes,
and the privacy rules. It does **not** score your inbox or groom the `wiki/` pages; it reads
config and writes drafts + a log entry.

## When to use
"Write the weekly update", "status report", "delivery report", "what shipped this week",
"sprint/iteration summary", "update for my manager/stakeholders" — interactive or on a
schedule (see **[../../routines/PROMPTS.md](../../routines/PROMPTS.md)**).

## When not to use
- Triaging what needs *you* → **jarvis**.
- Tuning learned preferences → **jarvis-tune**.

## Steps

Copy this checklist and track progress:
```
- [ ] 1 Detect mode + read delivery config
- [ ] 2 Set the window (since last report)
- [ ] 3 Gather GitHub (PRs, CI, releases)
- [ ] 4 Gather Azure DevOps (plan vs done, burndown)
- [ ] 5 (Optional) gather incidents
- [ ] 6 Synthesize the stakeholder update
- [ ] 7 Deliver as drafts
- [ ] 8 Log the run
```

1. **Detect mode** (interactive vs. unattended Workflow) and **read** `profile.md`'s
   `delivery` block (team, GitHub repos/org, ADO org/project/team + current iteration,
   `distribution_list`) + `automation` + `cursors.md`. Missing `delivery` config: interactive
   → ask for the repos, ADO project, and distribution list, then save them; unattended →
   deliver "delivery profile not set, configure once interactively" and exit.
2. **Set the window**: since the `weekly-delivery-report` cursor (else the last 7 days).
3. **GitHub** across the team's repos: merged PRs in the window (with author + one-line
   purpose), notable commits, **CI health** (workflow pass rate + any red main builds), and
   releases shipped. Summarize as *outcomes*, not a PR dump.
4. **Azure DevOps** for the current iteration: **completed vs. planned** work items, remaining
   **burndown**, and any blocked/at-risk items; pipeline/build health. Query by iteration or
   WIQL.
5. *(Optional)* **incidents/cost**: notable prod incidents or cost anomalies in the window, if
   an Azure monitor connector is available.
6. **Synthesize** a stakeholder update in the shape of
   **[reference/report-format.md](reference/report-format.md)**: BLUF (did we hit the plan?) →
   **Shipped** → **In flight** (with % done + burndown) → **Risks / incidents / blockers** →
   **Next week** → a metrics footer. Audience is stakeholders: outcome-first, low jargon, link
   the detail rather than pasting it.
7. **Deliver as DRAFTS — never send or post.** Create an **Outlook draft** to the
   `distribution_list` and prepare a **Teams post draft** (text only).
   - *Interactive*: show the update; offer to send/post on explicit confirmation.
   - *Workflow*: create the Outlook draft and deliver the Teams draft text to
     `profile.automation.brief_destination`. Posting to a channel requires
     `automation.may_post_to_teams_channels: true`.
8. **Log the run.** Append a `report` entry to `log.md` (window + counts) and advance the
   `weekly-delivery-report` cursor; commit if `STATE_DIR` is a git repo.

## Running unattended (scheduled Workflow)
No questions, no approval prompts — never block. **Draft-only** unless `automation`
pre-authorizes sending/posting. Idempotent via `cursors.md`. Deliver (don't print) to
`brief_destination`. **Degrade & report**: if GitHub, ADO, or the Azure connector is
empty/errored, say so in the update and continue with what you have — a green run means "no
error", not "the report is complete". Full contract:
**[../../shared/memory.md](../../shared/memory.md)**.

## Guardrails
- Draft-only: no send, post, or merge without consent.
- Facts from data, not vibes — cite counts (PRs merged, items done/planned, build pass rate);
  if a number isn't available, say "not available", don't estimate.
- Stakeholder-safe: summarize; don't leak secrets, internal-only links, or unreleased details
  beyond what the audience should see.

## Adapt to your setup
**Azure DevOps** is the bundled `mcp__ado__*` server — just set your org in the plugin's
config (`ado_org`); see the README. **GitHub** and the optional **Azure** monitor are
connectors you already have — rename the `mcp__github__*` / `mcp__azure__*` names if your
server keys differ. **Outlook/Teams** delivery tools are illustrative — rename to your
connectors. Team specifics (repos, project, distribution list) live in `profile.md` —
**data, not code**.
