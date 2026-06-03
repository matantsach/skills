<!--
comms-triage OUTPUT SPEC — render a brief in this shape from the Item data + the wiki.
This is a spec + worked example for the model to imitate, NOT a templating engine.

What makes it "mature" (the bar: a good chief-of-staff, not a notification dump):
- BLUF: open with a 1–2 sentence *judgment* of the day, then a pulse line of count chips.
- Lead with where the USER is the bottleneck (other people blocked on them).
- Track response debt: owed replies, sorted oldest-first, with aging.
- Be calendar-aware when calendar data is available (prep + protect focus time).
- Say what you already did ("Handled:") and surface low-confidence calls ("Wasn't sure")
  so nothing is silently hidden — this is how you earn trust to act more autonomously.
- Number actionable items with stable IDs so the user can say "send 2 / snooze 4 / task 3".
- Per item: [action] chip + who/age/effort/deadline chips + one-line ask + draft preview
  (if prepared) + a short why(provenance) + a link.
- Skimmable: short lines, sparing emoji as anchors, NO wide tables (must render in Teams,
  Outlook, and a terminal). Omit any empty section. If nothing needs them, say so plainly.
- Always show the filtered count + how to correct it (this feeds comms-tune).

Chip legend:
  [Decide|Reply|Review|Approve|Unblock]   action the item needs from you
  ⏰ age/owed (asked 2d)   ⌛ effort (~5m / ~20m / deep)   📌 deadline (due 5pm / overdue 1d)
  🚧 who's waiting on you   ✍️ draft ready
Section order (skip any that's empty):
  BLUF + pulse + Handled → 🔴 Needs you today → ↩️ Owed (aging) → 🟠 This week →
  📅 Meeting prep (calendar only) → 🤔 Wasn't sure → 📰 FYI digest → 🔇 Filtered + footer

────────────────────────  WORKED EXAMPLE  ────────────────────────
-->

**📬 Brief — Tue Jun 2 · since 17:30 yesterday**

> **Bottom line:** Two things gate other people today — Dana needs your Q3 headcount number and Sam is blocked on your API review; both have drafts ready. You also still owe Priya from Friday.

🔴 2 need you · ↩️ 3 owed (Priya, 4d) · 🚧 2 waiting on you · 📰 5 FYI · 🔇 41 filtered
🗓 4 meetings · 90m free before your 11:00
✅ **Handled:** prepared 4 drafts · auto-archived 41 per your rules · snoozed 2 newsletters to Fri.

---

### 🔴 Needs you today
1. **[Decide] Q3 headcount — final number** · ⏰ asked 1d · ⌛ ~5m · 📌 due today · 🚧 Dana (manager)
   She needs your backfill + req count to lock the Q3 plan in today's leads sync.
   ✍️ *Draft ready* — "Going with 4 backfills + 1 new req; rationale below…"
   _why: VIP / your manager · rule: planning-blockers = act-now_ · [open]
2. **[Review] PR #1841 — payments retry** · ⏰ requested 18h · ⌛ ~20m · 🚧 Sam (your report)
   Blocked on merge behind the on-call fix. 230 LOC, tests green.
   _why: report blocked + you're the requested reviewer_ · [open]

### ↩️ Owed replies (your response debt, oldest first)
- **Priya** · 4d · wants your call on the SDK deprecation timeline · ✍️ draft ready · [open]
- **Marco (skip-level)** · 2d · async question on the team split · needs you · [open]
- **#platform-leads** · 1d · you were @asked for the SLO target · ✍️ draft ready · [open]

### 🟠 This week (no rush)
- Recruiting wants availability for 2 onsite loops — ⌛ ~3m · [open]
- Vendor renewal decision, due Fri · [open]

### 📅 Meeting prep — today
- **11:00 · Design review — inventory sync** — they'll want your call on **sync vs async**. 2 RFC comments are addressed to you. Skim [RFC]. · [open]
- **14:00 · 1:1 — Sam** — Sam's been blocked twice this week; worth covering PR #1841 + scope. · [open]

### 🤔 Wasn't sure (quick confirm — teaches me)
- Held an **AcmeAI demo invite** as FYI (you usually decline vendor demos, but this one @mentions you). Promote to "needs you"? Reply **"yes 9"**.

### 📰 FYI — what changed in your areas (no action)
- ✅ Release **v1.4.0** shipped 22:10; error rate flat. · [open]
- 🟧 **SEV-3** in payments overnight — already mitigated by on-call. · [open]
- 🧵 Decision in #architecture: standardize on OpenTelemetry. · [open]
- 🔀 3 notable PRs merged across your repos. · [open]

---

🔇 **Filtered 41** — #social 12 · build-bot 18 · newsletters 7 · cc-only threads 4. Reply **"show filtered"** to review, or **"wrong: <who>"** if I hid something important.

**Reply debt:** 3 (▲1 since yesterday) · **Do next:** `send <#>` · `open <#>` · `snooze <#> <when>` · `task <#>` · `mute <source>` · `tune`
