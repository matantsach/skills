# eng-workflows

A **Claude Code plugin** of generic, self-learning skills for software engineers and
engineering managers. It sits on the MCP connectors you already have — **Outlook,
Microsoft Teams, GitHub, Azure DevOps, Azure** — and targets the thing that hurts most
first: **communication overload**.

Nothing about your team, your VIPs, or your noisy channels is hard-coded — that lives in a
small **local, version-controlled wiki** the skills groom as you use them, so one install
adapts to each person. Skills run **interactively** (chat) and **unattended** as scheduled
local **Workflows**. Works in Claude Code and other [Agent Skills](https://agentskills.io)
hosts (incl. GitHub Copilot).

---

## What's in here

| Skill (`/eng-workflows:<name>`) | What it does | Status |
|------|--------------|--------|
| **comms-triage** | One ranked brief of what needs you across Outlook + Teams, in your voice. Collapses noisy group threads; suggests an action per item. Read-only / draft-only. | ✅ |
| **comms-tune** | The wiki **curator**: distills what you ignore vs. act on into provenance-backed rules and lints them for contradictions/staleness. | ✅ |
| `weekly-delivery-report`, `thread-summarize-reply`, `meeting-to-actions` | reporting + thread + meeting workflows | 🛣️ roadmap |

## The memory model — a curated LLM-Wiki

The "learning" is a small **markdown wiki you own**, grooming itself like a real wiki. We
implement Andrej Karpathy's
[LLM-Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): an
append-only **log** of what you did → compiled **wiki pages** (one concept each, with
provenance + cross-refs) → a co-evolved **schema**, with `ingest` / `query` / `lint`
operations. Knowledge **compounds** instead of being re-derived every run, and a periodic
**lint** prunes contradictions and stale claims. Human-readable, auditable, git-revertible.
Spec + credible sources: **[shared/memory-contract.md](shared/memory-contract.md)**.

---

## Install (one person)

In Claude Code:
```text
/plugin marketplace add your-org/eng-workflows      # GitHub owner/repo (private repos work if you have access)
/plugin install eng-workflows@eng-workflows
/reload-plugins
```
Then just ask: *"triage my Teams and Outlook — what needs me?"* (first run seeds your
wiki, ~2 min). Skills are namespaced, e.g. `/eng-workflows:comms-triage`. Choose **user**
scope (all your projects), **project** scope (shared with collaborators), or **local**.

Other marketplace sources also work: a git URL (`…/eng-workflows.git#v0.3.0`), a local path
(`/plugin marketplace add ./eng-workflows`), etc.

## Roll it out to your whole team

Commit this to the **team repo's `.claude/settings.json`**. When teammates trust the folder,
Claude Code prompts them to install — zero manual steps:
```json
{
  "extraKnownMarketplaces": {
    "eng-workflows": {
      "source": { "source": "github", "repo": "your-org/eng-workflows" }
    }
  },
  "enabledPlugins": ["eng-workflows@eng-workflows"]
}
```
(Admins can add `"autoUpdate": true` to the marketplace entry.) You can also publish to the
public **community marketplace** via [claude.ai/settings/plugins/submit](https://claude.ai/settings/plugins/submit).

## Run unattended (Workflows)

Workflows run autonomously with **no approval prompts**, so the suite is **draft-only by
default**: it leaves reply drafts in Outlook and delivers a brief to a self destination; you
review and send. Riskier actions require opting in via the `automation` block in
`profile.md`. Because Workflows run **on your machine**, the wiki persists locally — no
extra setup. Copy-paste prompts + cadences: **[routines/PROMPTS.md](routines/PROMPTS.md)**.

## Develop / extend

```text
claude --plugin-dir ./eng-workflows     # load locally without installing (also accepts a .zip)
claude plugin validate ./eng-workflows  # validate before sharing
/reload-plugins                          # pick up edits in-session
```
- **Connectors**: rename the illustrative `mcp__outlook__*` / `mcp__teams__*` in each
  skill's `allowed-tools` to your installed servers. A plugin *can* also bundle MCP servers
  via a root `.mcp.json` (like the official `github`/`slack` plugins) — optional, since your
  Microsoft connectors are likely company-provided.
- **Adapt = data, not code.** Personalization is each person's `~/.eng-workflows/` wiki; you
  rarely touch a `SKILL.md`.
- **Note:** plugins are copied to a cache on install, so skills only reference files *inside*
  the plugin (ours do — `shared/`, `templates/` live in the plugin root).

## Safety

- **Draft + consent.** Nothing is sent/posted/merged/created without consent — asked inline
  when interactive, pre-authorized in `automation` when unattended; else it drafts.
- **Local + private.** Your wiki lives in `~/.eng-workflows/`, never uploaded; snippets only.
- **Provenance & reversibility.** Every learned claim cites evidence; every change is a
  visible edit you can revert.

MIT.
