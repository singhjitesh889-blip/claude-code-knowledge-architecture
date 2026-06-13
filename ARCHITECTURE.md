# Architecture Reference

## The 3-Layer Model

```
┌─────────────────────────────────────────────────────────┐
│  RAW  (immutable source — never edited after creation)  │
│  Monthly exports · Research · Observations · Articles   │
└────────────────────────┬────────────────────────────────┘
                         │  monthly compile
                         ▼
┌─────────────────────────────────────────────────────────┐
│  WIKI  (compiled intel — one file, one question)        │
│  creative-brief · competitors · customer · brand        │
└────────────────────────┬────────────────────────────────┘
                         │  distills into
                         ▼
┌─────────────────────────────────────────────────────────┐
│  STATE  (live operational dashboard — ≤800 tokens)      │
│  current metrics · active rules · project status        │
└─────────────────────────────────────────────────────────┘
                         │  injected at session start
                         ▼
                    AI session
```

---

## Layer Map

### RAW

| Folder | What goes in | Rule |
|--------|-------------|------|
| `raw/monthly/` | Performance exports, CSVs, data pulls | Named `YYYY-MM-[domain].csv` — never edited |
| `raw/research/` | External articles, competitor screenshots, market notes | Dated prefix, verbatim copy |
| `raw/thinking/` | Quick observations, gut reads, half-formed ideas | One thought per file, timestamped |

**The immutability rule:** RAW files are never edited after creation. If something was true then and false now, that's information. The compile loop handles the contradiction — the RAW file stays as a dated record.

---

### WIKI

Each file answers exactly one question. This is the routing contract.

| File | Question it answers | Updated by |
|------|---------------------|------------|
| `wiki/creative-brief.md` | What works creatively — formats, angles, hooks, patterns | Monthly compile |
| `wiki/customer.md` | Who the buyer is — segments, JTBD, language, objections | Monthly compile |
| `wiki/competitors.md` | What rivals are doing — active strategies, positioning moves | Weekly/monthly sensors |
| `wiki/brand.md` | Voice, identity, guardrails — what we are and aren't | Rarely — brand DNA is stable |
| `wiki/market.md` | Category trends, demand signals, seasonal patterns | Monthly compile |

**The one-job rule:** If a WIKI file starts answering multiple questions, split it. A file that answers "creative patterns AND customer psychology" will be loaded when only one is needed, burning context on irrelevant material.

---

### STATE

The thin always-current layer. Loaded every session. Must stay under ~800 tokens.

```markdown
# STATE — [Project Name]
last_updated: YYYY-MM-DD

## Current Numbers
[Key metrics — revenue, conversion, ROAS, whatever matters]

## Active Rules
[3-5 current operating rules derived from last compile]

## Active Hypotheses
[Things we're testing — not confirmed yet]

## Blockers / Open Questions
[What's stuck — requires a decision or more data]

## Project Status
[One line per live project — status + last action]
```

**The staleness rule:** STATE is wrong the moment it's not updated. Set a staleness threshold per field (e.g. metrics → 7 days, rules → monthly). A hook can check the `last_updated` date and warn when STATE is stale before a session starts.

---

### Supporting Layers

These aren't knowledge — they're infrastructure.

| Layer | Location | What it is |
|-------|----------|------------|
| SCHEMA | `COMPILER.md`, `CLAUDE.md`, `rules/` | Behavior contracts — how the AI should act |
| MEMORY | `memory/MEMORY.md` | Claude's persistent cross-session context index |
| HOOKS | `.claude/hooks/` | Automated behaviors — context injection, routing, auto-commit |
| OUTPUTS | `outputs/` | Generated artifacts — reports, briefs, copy — feeds next compile as RAW |

---

## Routing Rules

**How the AI decides what to load:**

```
Every session → load STATE (always, small, injected by hook)

User asks about creative/content → load wiki/creative-brief.md
User asks about competitors → load wiki/competitors.md  
User asks about customers → load wiki/customer.md
User asks strategy question → load STATE + wiki/market.md
User asks about a specific project → load that project's CLAUDE.md

RAW → never load directly in sessions. Use compiled WIKI.
```

The routing rules live in `CLAUDE.md` as a context router table. The AI reads this table before responding and loads only what the task needs.

---

## Folder Convention

```
project-root/
├── CLAUDE.md                 # Workspace context + routing table
├── raw/
│   ├── monthly/              # Monthly data drops
│   ├── research/             # External intel
│   └── thinking/             # Quick observations
├── wiki/
│   ├── creative-brief.md     # Hot — updated monthly
│   ├── customer.md           # Medium — updated monthly
│   ├── competitors.md        # Hot — updated weekly
│   ├── brand.md              # Cold — rarely updated
│   └── market.md             # Medium — updated monthly
├── state/
│   └── STATE.md              # Always-current operational dashboard
├── outputs/                  # Generated artifacts (feeds next compile)
├── memory/
│   └── MEMORY.md             # Claude's persistent context index
├── COMPILER.md               # Compile loop rules + prompts
└── .claude/
    └── hooks/                # Automation layer
```

---

## The Compile Contract

The compile loop has one job: **rewrite WIKI to reflect current reality.**

Not append. Rewrite.

When you run a compile:
1. New RAW data is the challenger. Old WIKI is the incumbent.
2. For every claim in WIKI, ask: does new data confirm, contradict, or have no bearing?
3. Contradictions get resolved in favor of newer data, with a note on what changed.
4. Newly confirmed patterns get upgraded from HYPOTHESIS to RULE in STATE.
5. Stale rules that new data refutes get downgraded or removed.

**Output of a compile:**
- WIKI hot files are rewritten (not appended)
- STATE.md `active_rules` updated
- STATE.md `last_updated` bumped
- Compile log entry added

See `compile-loop.md` for the prompt and step-by-step instructions.
