# Jarvis

A **personal chief of staff** for software engineers and engineering managers, packaged as a
**Claude Code plugin**. It sits on the MCP connectors you already have — **Outlook, Microsoft
Teams** (plus GitHub, Azure DevOps, Azure for roadmap skills) — and attacks the thing that
hurts most: **communication overload**.

Jarvis turns an overwhelming inbox + Teams into **one ranked brief of what needs you** —
bottleneck-first, in your voice — drafts the replies, and gets sharper every run by grooming a
small, private memory of who and what matters to you.

Nothing about your team, your VIPs, or your noisy channels is hard-coded. That lives in a
**local, version-controlled wiki** the skill grooms as you use it, so one install adapts to
each person. Jarvis runs **interactively** (chat) and **unattended** as scheduled local
**Workflows**. Works in Claude Code and other [Agent Skills](https://agentskills.io) hosts
(incl. GitHub Copilot).

---

## What's in here

**One main skill. Everything else is auxiliary to it.**

| Skill | Role | What it does |
|------|------|--------------|
| **`jarvis`** | ⭐ main | One ranked brief of what needs you across Outlook + Teams, in your voice. Collapses noisy threads, drafts replies, suggests one action per item, and learns from what you do. Read-only / draft-only. |
| **`jarvis-tune`** | auxiliary | The memory **curator**: distills what you ignore vs. act on into provenance-backed rules and lints them for contradictions/staleness, so `jarvis` stays accurate and noise drops over time. |
| `weekly-delivery-report`, `alert-triage`, … | 🛣️ roadmap | reporting + incident workflows over GitHub / ADO / Azure |

You talk to **`jarvis`**. `jarvis-tune` is the weekly tune-up that keeps it honest — invoke it
when triage gets noisy ("reduce the noise", "you keep showing me X") or run it on a schedule.

## The memory model — a curated wiki

The "learning" is a small **markdown wiki you own**, grooming itself like a real wiki. We
implement Andrej Karpathy's
[LLM-Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): an
append-only **log** of what you did → compiled **wiki pages** (one concept each, with
provenance + cross-refs) → a co-evolved **schema**, with `ingest` / `query` / `lint`
operations. Knowledge **compounds** instead of being re-derived every run, and a periodic
**lint** prunes contradictions and stale claims. Human-readable, auditable, git-revertible.
Full spec + credible sources: **[shared/memory.md](shared/memory.md)**.

It lives in **`~/.jarvis/`** (override with `$JARVIS_STATE`) — never in a shared or work repo.

---

## Install (one person)

In Claude Code:
```text
/plugin marketplace add matantsach/jarvis      # GitHub owner/repo (private repos work if you have access)
/plugin install jarvis@jarvis
/reload-plugins
```
Then just ask: *"triage my Teams and Outlook — what needs me?"* (first run seeds your wiki,
~2 min). Skills are namespaced, e.g. `/jarvis:jarvis` and `/jarvis:jarvis-tune`. Choose **user**
scope (all your projects), **project** scope (shared with collaborators), or **local**.

Other marketplace sources also work: a git URL (`…/jarvis.git#v0.4.0`) or a local path
(`/plugin marketplace add ./jarvis`).

## Roll it out to your whole team

Commit this to the **team repo's `.claude/settings.json`**. When teammates trust the folder,
Claude Code prompts them to install — zero manual steps:
```json
{
  "extraKnownMarketplaces": {
    "jarvis": {
      "source": { "source": "github", "repo": "matantsach/jarvis" }
    }
  },
  "enabledPlugins": ["jarvis@jarvis"]
}
```
(Admins can add `"autoUpdate": true` to the marketplace entry.) You can also publish to the
public **community marketplace** via [claude.ai/settings/plugins/submit](https://claude.ai/settings/plugins/submit).

## Run unattended (Workflows)

Workflows run autonomously with **no approval prompts**, so Jarvis is **draft-only by default**:
it leaves reply drafts in Outlook and delivers a brief to a self destination; you review and
send. Riskier actions require opting in via the `automation` block in `profile.md`. Because
Workflows run **on your machine**, the wiki persists locally — no extra setup. Copy-paste
prompts + cadences: **[routines/PROMPTS.md](routines/PROMPTS.md)**.

## Develop / extend

```text
claude --plugin-dir ./jarvis     # load locally without installing (also accepts a .zip)
claude plugin validate ./jarvis  # validate before sharing
/reload-plugins                  # pick up edits in-session
```
- **Connectors**: the illustrative `mcp__outlook__*` / `mcp__teams__*` in each skill's
  `allowed-tools` are placeholders — rename them to your installed servers. A plugin *can*
  also bundle MCP servers via a root `.mcp.json` (like the official `github`/`slack` plugins),
  but your Microsoft connectors are likely company-provided.
- **Adapt = data, not code.** Personalization is each person's `~/.jarvis/` wiki; you rarely
  touch a `SKILL.md`.
- **Self-contained.** Plugins are copied to a cache on install, so skills only reference files
  *inside* the plugin (`shared/`, `reference/` live in the plugin root and each skill).

## Safety

- **Draft + consent.** Nothing is sent/posted/merged/created without consent — asked inline
  when interactive, pre-authorized in `automation` when unattended; else it drafts.
- **Local + private.** Your wiki lives in `~/.jarvis/`, never uploaded; snippets only.
- **Provenance & reversibility.** Every learned claim cites evidence; every change is a visible
  edit you can revert.

MIT.
