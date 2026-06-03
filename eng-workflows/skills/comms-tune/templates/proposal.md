<!--
comms-tune OUTPUT SPEC — the weekly "what I learned about you" changelog + proposal.
Spec + worked example to imitate; NOT a templating engine.

Make it read like crisp release notes for the user's own preferences:
- BLUF: one line — what changed + the headline effect (noise ↓, VIPs learned, debt trend).
- Group proposals: People · Channels · Topics · Rules. Each line: before → after, evidence
  (counts/dates from log.md), confidence, and the *effect* on future briefs.
- Separate "safe to apply" (down-weights, demotions, fixes) from "needs your ok" (hard
  mutes, new VIPs). Never apply the latter unattended.
- A "Wiki health (lint)" block: what the lint found/fixed — contradictions, stale claims,
  gaps, orphans, missing provenance.
- Number every proposal with a stable ID for apply/reject; show how to apply.
- Always reversible (plain edit / git). Empty state: say the wiki's healthy + any auto-fixes.
- Skimmable, short lines, no wide tables. Delivered to brief_destination; nothing is applied
  unattended unless profile.automation.comms_tune_writes permits (and never hard mutes/VIPs).

────────────────────────  WORKED EXAMPLE  ────────────────────────
-->

**🧠 What I learned about you — week of Jun 2**

> **Bottom line:** I can cut ~22 low-value pings/week and learned 1 likely new VIP; also retired 2 stale rules and resolved 1 contradiction. **Nothing applied yet** — your call below.

📊 4 proposals (2 safe · 2 need your ok) · 🩹 4 health fixes · ↩️ reply debt ▼ 5 → 3

---

### ✅ Safe to apply  (low-risk — reply `apply safe`, or pick numbers)
1. **`#build-notifications`: low → mute** — you opened 1 of 23 last 2 wks. _conf: high_ → ~15 fewer pings/wk; still shown if it @mentions you.
2. **`*@substack.com`: normal → noise** — archived 9/9 unread. _conf: high_ → drops off "needs you".

### 🔶 Needs your ok  (higher-impact — won't apply on my own)
3. **New VIP: `priya@contoso.com`** — replied 5/5 within 1h; twice you tagged her thread "blocking". _conf: med_ → always Act-now + pre-draft. → `apply 3`
4. **Hard-mute: `#all-hands-social`** — 0 of 40 opened. _conf: high but broad_ → I'd stop surfacing it entirely. → `apply 4`

### 🩹 Wiki health (lint)
- **Contradiction resolved:** `#platform` was both `high` and `mute` → kept `high` (you post there daily). 
- **Stale (30d+):** retired "Q1 reorg watchlist". The "Fri deploy-freeze" rule is also stale — still true? → `keep deploy-freeze`
- **Gap:** `marco@` (skip-level) appears weekly but isn't on your people page. → `add marco`
- **Provenance:** backfilled evidence on 3 rules that lacked it.

### 📈 Effect on your morning brief
- Est. **−22 items/wk** surfaced · **1 false-mute risk** flagged for you (item 4) rather than auto-applied.
- Coverage now: 6 VIPs · 4 priority channels. Reply-debt trend: ▼ 5 → 3.

---

**Apply:** `apply safe` · `apply <#>` · `reject <#>` · `keep <rule>` · `show diff`
Everything is a plain edit you can revert (and a git commit if you version the wiki). I changed nothing yet.
