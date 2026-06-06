# Jarvis memory — a curated wiki (the second brain)

The shared "brain" both Jarvis skills read and write. It is what makes a *generic* assistant
**adapt to one person over time**, using plain, human-readable, **version-controlled
markdown** — no agent-specific memory feature, no vector DB, no model training. The same
files work in Claude Code, GitHub Copilot, Cursor, Codex, and in **local scheduled
Workflows**.

## Contents
- [Design basis: the LLM-Wiki pattern](#design-basis-the-llm-wiki-pattern)
- [Two execution modes (read this first)](#two-execution-modes-read-this-first)
- [Where the wiki lives](#where-the-wiki-lives)
- [Page formats (with seeds)](#page-formats-with-seeds)
- [Normalized Item schema](#normalized-item-schema-generic-over-outlook--teams)
- [Importance scoring](#importance-scoring-signals--buckets)
- [Privacy & safety (non-negotiable)](#privacy--safety-non-negotiable)
- [First-run onboarding (interactive only)](#first-run-onboarding-interactive-only)
- [Sources](#sources-credible-primary)

---

## Design basis: the LLM-Wiki pattern

Jarvis implements Andrej Karpathy's **LLM-Wiki** pattern
([gist, 2026](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)). Instead
of RAG over raw logs ("rediscovering knowledge from scratch on every question"), the agent
maintains a **persistent, compounding** wiki it grooms itself. Three layers, three
operations:

| Layer | In Karpathy's pattern | In Jarvis |
|------|------------------------|-----------|
| **Raw sources** (immutable) | `raw/` documents | `log.md` — append-only record of what you actually did |
| **Wiki** (LLM-maintained pages) | `wiki/*.md`, one per concept, `[[wikilinks]]`, provenance | `wiki/people.md`, `channels.md`, `topics.md`, `rules.md`, `voice.md` |
| **Schema** (co-evolved spec) | `CLAUDE.md` / `AGENTS.md` | `_schema.md` + `profile.md` |

| Operation | What it does | Which skill |
|-----------|--------------|-------------|
| **ingest** | record a decision/correction → update the relevant page(s) + `index.md` + append `log.md` | **jarvis** (every run) |
| **query** | read `index.md` first, drill into the compiled pages, act | **jarvis** (scoring) |
| **lint** | health checks (contradictions, stale claims, orphans, missing provenance, gaps) → consolidate | **jarvis-tune** (the curator) |

Reliability comes from the same things that make a real wiki trustworthy: **provenance-first**
(every learned claim cites its evidence), **one concept per page** with cross-references, an
**index** as the retrieval entry point, an append-only **log**, **git history**, and a
periodic **lint** that prunes contradictions and stale claims. The human directs; the model
does the bookkeeping. This is well-trodden ground — see [Sources](#sources-credible-primary).

---

## Two execution modes (read this first)

| | **Interactive** (chat / CLI / Copilot) | **Scheduled Workflow** (local desktop task) |
|---|---|---|
| Human present? | Yes — can answer questions | **No** — runs unattended, no approval prompts |
| Filesystem | Persists | **Persists** (runs on your machine — local files are durable) |
| Confirmation | Ask inline before sending | **Pre-authorized in `profile.md`**; otherwise **draft only** |
| Output | Shown in the conversation | **Delivered** via a connector (Outlook draft / Teams self-chat) |

**Golden rule for unattended runs:** never block on input, never ask a question, and **never
send/post/merge/create anything unless `profile.md`'s `automation` block pre-authorizes it.**
Default to producing *drafts* + a *brief*; your later review is the confirmation.

---

## Where the wiki lives

Resolve `STATE_DIR` once per run: `$JARVIS_STATE` if set, else `~/.jarvis/`. Because Workflows
run **locally on your machine**, this directory **persists between runs** — no special
handling needed. Keeping `STATE_DIR` under **git is recommended** (version history +
provenance) but *optional*, and only for versioning/sync, not persistence.

```
~/.jarvis/
  _schema.md         # the spec: page conventions + ingest/query/lint workflows (you co-evolve it)
  index.md           # catalog of wiki pages, one line each — read first for retrieval
  log.md             # append-only: every run, decision, and lint pass (the raw layer)
  profile.md         # you: identity, hours, tone, goals, and the `automation` policy
  cursors.md         # per-skill "last run" timestamps (incremental, idempotent runs)
  proposals.md       # curation proposals awaiting your review (written by jarvis-tune)
  wiki/
    people.md        # contacts that matter + how to handle them (provenance + confidence)
    channels.md      # Teams channels / Outlook folders: priority or mute
    topics.md        # keywords/projects: signal vs. noise
    rules.md         # distilled heuristics — the "learned model"
    voice.md         # tone guide + approved example replies
```

If `STATE_DIR` has no `profile.md`: interactive → run [onboarding](#first-run-onboarding-interactive-only);
unattended → deliver "profile not found, onboard once interactively" to the brief destination
and exit.

---

## Page formats (with seeds)

Every learned claim is **provenance-first**: it cites evidence from `log.md` and carries a
**confidence** and a **last-confirmed** date so the lint pass can age it out.

### `_schema.md` (the co-evolved spec)
```markdown
# jarvis schema
- pages live in wiki/, one concept per file; cross-link with [[page#anchor]].
- every learned row cites evidence (log dates/counts), confidence (low|med|high), updated (date).
- ingest: on each decision, append to log.md and update the affected wiki page + index.md.
- query: read index.md, then only the pages you need; never score from raw log.md.
- lint (jarvis-tune): flag contradictions, prune claims not confirmed in 30d unless conf=high,
  remove orphans, fix missing provenance, report gaps. Propose diffs; apply per automation.
```

### `index.md`
```markdown
# Wiki index   (updated on every ingest)
## people   — VIPs and how to handle them            → wiki/people.md
## channels — Teams/Outlook sources and priorities    → wiki/channels.md
## topics   — signal vs noise keywords                → wiki/topics.md
## rules    — learned triage heuristics (N rules)     → wiki/rules.md
## voice    — how my drafts should sound              → wiki/voice.md
```

### `log.md` (append-only raw layer)
```markdown
## [2026-06-02T08:00Z] triage | 14 items → 2 act-now, 3 today
- outlook | from=dana@contoso | "Q3 planning" | predicted=ActNow | action=replied
- teams   | "builds-and-ci: #842" | predicted=Today | action=ignored | correction=mute-channel
## [2026-06-02T07:30Z] lint | flagged 1 contradiction, pruned 2 stale rules
```

### `profile.md`
```markdown
# Profile
- name / role / team / timezone / working_hours / tone / goals: <fill in>
- response_sla: { vip: "2h", team: "same-day", fyi: "none" }

## automation   (ONLY used when run unattended as a Workflow)
- unattended_mode: draft-only          # draft-only | act-on-allowlist
- brief_destination: outlook-draft-to-self   # | email-to-self | teams-self-chat | state-file
- may_send_email: false
- may_post_to_teams_channels: false
- may_create_work_items: false         # if true → create with label "needs-review"
- memory_writes: propose-only          # propose-only | apply-safe | apply-all  (jarvis-tune)
```

### `wiki/people.md`
```markdown
# People   ([[topics]] referenced by handling notes)
| who               | relationship | priority | handling                  | evidence            | conf | updated   |
|-------------------|--------------|----------|---------------------------|---------------------|------|-----------|
| dana@contoso.com  | my manager   | VIP      | surface now; draft reply  | replied 6/6 <2h     | high | 2026-06-02|
| noreply@github.com| bot          | mute     | only if @me + "fail"      | ignored 18/18       | high | 2026-05-30|
```
`priority`: `VIP | high | normal | low | mute`.

### `wiki/channels.md`
```markdown
# Channels
| source | name              | priority | evidence              | conf | updated   |
|--------|-------------------|----------|-----------------------|------|-----------|
| teams  | Platform / General| high     | I post here daily     | high | 2026-06-01|
| teams  | builds-and-ci     | low      | opened 1/22 last month| high | 2026-05-30|
| outlook| Newsletters       | mute     | archived 30/30        | high | 2026-05-28|
```

### `wiki/topics.md`
```markdown
# Topics
## signal   — "payments-service", "SEV", "incident", "rollback", "<codenames>"
## noise    — "lunch", "kudos", "office reopening", "weekly newsletter"
(each line may carry: evidence, conf, updated)
```

### `wiki/rules.md` (the distilled model — regenerated by lint)
```markdown
# Learned rules
- Mute [[channels#builds-and-ci]] unless body has "failed" AND @me.  (ev: ignored 14/15; conf: high; confirmed: 2026-06-01)
- [[people#dana]] is VIP → always Act-now + draft.                    (ev: replied 6/6 <2h; conf: high; confirmed: 2026-06-02)
- Group threads >20 msgs where I'm only cc'd → collapse to FYI.       (ev: 9 examples; conf: med; confirmed: 2026-05-29)
```

### `wiki/voice.md`
```markdown
# Voice
- defaults: first-person, 2-4 sentences, no "I hope this finds you well". Sign-off "— <name>".
## examples (approved replies, style references)
> Thanks for flagging. I'll take payments; can you cover the webhook retry? Aiming for EOD.
```

### `cursors.md`
```markdown
- jarvis:      2026-06-02T08:00Z
- jarvis-tune: 2026-05-26T07:30Z
```

---

## Normalized Item schema (generic over Outlook + Teams)

Map every email / chat / channel post to one shape before scoring. Keep **snippets, not full
bodies** in any persisted artifact.

```yaml
id: ; source: outlook|teams ; kind: email|chat|channel_post|meeting_invite
audience: direct|group|broadcast ; from: {name,address} ; mentions_me: bool
thread_id: ; title: ; snippet: (<=200 chars) ; received_at: ; link: ; unread: ; flagged:
```

## Importance scoring (signals → buckets)

Read the compiled `wiki/` pages (via `index.md`) — never score from raw `log.md`. Buckets:
🔴 **Act now** · 🟠 **Today** · 🟡 **FYI** · ⚪ **Muted**. Raise for sender `priority`,
`mentions_me`, `direct` audience, a question/deadline to you, a `topics` signal, a thread
you're already in. Lower for a `mute`/`low` source, a `broadcast` you're not named in,
`topics` noise, big group threads you're only cc'd on. `rules.md` overrides. Always show a
one-line **why**.

---

## Privacy & safety (non-negotiable)

- **Draft, don't send.** No send/post/merge/create without consent — asked inline when
  interactive, pre-authorized in `automation` when unattended; otherwise produce a draft.
- **Private & local.** The wiki lives in your private `STATE_DIR`; never put it in a
  shared/work repo or upload it. Snippets only — never full bodies or secrets.
- **Provenance over guesses.** A learned claim must cite evidence; if you can't, don't write
  it as a rule. State facts, not feelings.
- **Degrade & report.** If a connector is missing/empty/errors, say so in the output. A
  green Workflow status only means "no infra error," not "the task worked."

## First-run onboarding (interactive only)

If `STATE_DIR` is empty and a human is present: create `_schema.md`, `index.md`, `log.md`,
`profile.md`, `cursors.md`, and the `wiki/` pages from the seeds above; ask 4–6 quick
questions (name/role/timezone, hours, tone, "up to 5 people who should always reach you");
set `automation` conservatively (`draft-only`); optionally `git init`. Workflows assume this
ran once.

---

## Sources (credible, primary)

- Karpathy, **LLM-Wiki** pattern — [gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) (primary).
- Karpathy, **"system prompt learning"** — models improve by editing their own notes/instructions ([summary](https://x.com/koltregaskes/status/1921453031427649920), May 2025).
- Shinn et al., **Reflexion**: verbal reinforcement / self-written reflections as memory — [arXiv:2303.11366](https://arxiv.org/abs/2303.11366).
- Park et al., **Generative Agents**: observation→reflection→planning memory stream — [arXiv:2304.03442](https://arxiv.org/abs/2304.03442).
- Packer et al., **MemGPT / Letta**: self-editing hierarchical memory — [arXiv:2310.08560](https://arxiv.org/abs/2310.08560).
- Zhang et al., **A Survey on the Memory Mechanism of LLM-based Agents** — [ACM TOIS](https://dl.acm.org/doi/10.1145/3748302).
- Anthropic, **Effective context engineering for AI agents** (structured note-taking / file-based memory) — [engineering blog](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents).
