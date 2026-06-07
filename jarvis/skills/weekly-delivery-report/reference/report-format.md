<!--
weekly-delivery-report OUTPUT SPEC — a stakeholder delivery update.
Spec + worked example to imitate; NOT a templating engine.

The bar: a crisp update a busy stakeholder reads in 30 seconds and trusts.
- BLUF: one line — did we hit the plan this week, and the one thing that matters.
- Outcome-first, not a PR/commit dump. Group by what shipped, what's in flight, what's at risk.
- Every claim is backed by a real number (PRs merged, items done/planned, build pass rate,
  burndown). If a number isn't available, say "not available" — never estimate.
- In flight: show % done + remaining against the iteration plan.
- Risks/incidents/blockers: name the owner and the ask, if any.
- Link the detail (PRs, work items, dashboards); don't paste it.
- Skimmable, short lines, no wide tables (must render in Outlook + Teams). Omit empty sections.
- DRAFT ONLY: this is delivered as an Outlook draft + a Teams post draft for the user to send.

Section order (skip any that's empty):
  Subject/BLUF → ✅ Shipped → 🚧 In flight → ⚠️ Risks & incidents → ⏭️ Next week → 📊 Metrics

────────────────────────  WORKED EXAMPLE  ────────────────────────
-->

**Subject: Payments team — delivery update, week of Jun 2**

> **Bottom line:** On track for the Jun 13 SDK cutover — 8 of 11 planned items done and the retry rework shipped. One risk: the migration is blocked on a security review (owner: Priya).

### ✅ Shipped this week
- **Payment retry rework** — idempotent retries + backoff; cut duplicate-charge errors to ~0 in staging. (PR #1841 +3 follow-ups) · [PRs]
- **v1.4.0 released** Tue; error rate flat post-deploy. · [release]
- Onboarding docs for the new SDK auth flow. (PR #1855)

### 🚧 In flight (against the iteration plan)
- **SDK v2 migration — 70% (7/10 items).** On track for Jun 13. Remaining: token refresh + 2 edge cases.
- **Webhook delivery SLO dashboard — 40%.** Slipped a day; recoverable this week.

### ⚠️ Risks & incidents
- **Blocked:** SDK migration security review pending → **Priya**, requested Jun 4. Needs sign-off by Jun 9 to hold the date.
- **SEV-3** in payments Sun overnight, auto-mitigated by on-call; root cause = retry config, fixed in #1841. No customer impact.

### ⏭️ Next week
- Finish token refresh + edge cases; start the cutover runbook.
- Close out the SLO dashboard.

---

📊 **Metrics (Jun 2–6):** 14 PRs merged · CI pass rate 96% (1 red main build, reverted in 20m) · iteration burndown 8/11 done, on trend · 0 customer-facing incidents.

_Draft — review and send. Reply `send` to email the distribution list, `post` to publish the Teams draft, or `edit` to revise._
