# claude-code-knowledge-architecture

Most people use an AI coding assistant like a faster Stack Overflow: ask, get
an answer, lose it when the session ends. The context resets every time. The
assistant never gets to know your codebase, your decisions, or the mistakes it
already made with you last week.

This repo is a different bet: that the highest-leverage thing you can build
with an AI partner is **a place for it to keep what it learns** — and a
discipline for compiling that knowledge so it gets *sharper* over time instead
of just *bigger*.

---

## The Core Idea

It's a three-layer architecture:

**RAW** is everything unprocessed — exports, logs, half-formed observations,
external research. You drop things in. Nothing gets edited after creation.

**WIKI** is the compiled, deduplicated middle layer — the source of truth your
AI reads when it needs domain knowledge. It's not a dump; it's a distillation.
Creative intelligence lives here. Customer understanding lives here. Market
patterns live here. Each file has one job and answers one question.

**STATE** is the thin, always-current operational dashboard. Current metrics.
Active rules. Project status. It's what the AI reads *first* — before doing
anything in a session.

Raw flows up into wiki on a schedule. Wiki distills down into state. A monthly
compile loop forces the question every knowledge system fails to ask: *what
here is still true?*

---

## Why This Works

The insight that makes this worth building:

A single Claude Code session can be brilliant. But every session starts from
scratch unless you architect the memory yourself. The model doesn't carry
forward what it learned last Tuesday — you have to give it that.

Most people handle this with a CLAUDE.md file and a few paragraphs of context.
That works until the project gets complex. Then CLAUDE.md becomes a 400-line
sprawl that loads every session whether or not it's relevant, and the assistant
spends half its attention on irrelevant context.

The three-layer model solves this with **routing**. The STATE layer is small
and always loaded. The WIKI layer is medium and loaded on demand by topic. The
RAW layer is never auto-loaded — it feeds the compile, not the session. Context
loads in proportion to relevance.

The other thing it solves: decay. Every system grows stale. The monthly compile
loop is the mechanism that asks "what changed?" and rewrites the WIKI and STATE
accordingly. Without the loop, you're paying for memory that's lying to you.

---

## The Three Loops

```
Loop 1 — INGEST → COMPILE (monthly)
  You drop new data/exports → "compile [domain]" → COMPILER.md rules
  → rewrites WIKI hot files + updates STATE
  Enforces: what here is still true?

Loop 2 — QUERY → FILE BACK (every session)
  You ask → AI reads wiki → generates output → saved to Outputs/
  → feeds the next compile
  Enforces: outputs are inputs to future loops, not one-offs

Loop 3 — LINT → HEAL (on-demand)
  Scan all wiki → stale/gaps report → Loop 1 fixes
  Enforces: no silent staleness accumulating between monthly compiles
```

---

## What's In This Repo

```
README.md               ← you're reading it
ARCHITECTURE.md         ← folder conventions, layer map, routing rules
compile-loop.md         ← the monthly compile prompt + what it does
examples/
  before-compile/       ← what RAW looks like (anonymized)
  after-compile/        ← what WIKI looks like after processing RAW
```

---

## How to Adopt This

**Start with STATE.** Before anything else, write a `STATE.md` that answers
"what's true right now?" — current metrics, active priorities, known blockers.
Keep it under 800 tokens. Inject it at session start via a hook (see
[claude-code-hooks-kit](https://github.com/singhjitesh889-blip/claude-code-hooks-kit)).

**Then build WIKI by domain.** One file per domain that answers one question.
`creative-brief.md` answers "what works in our creative." `competitors.md`
answers "what are rivals doing." Don't merge domains — routing breaks when
files answer multiple questions.

**Then add RAW.** Start dropping raw inputs without editing them. The compile
loop does the editing. The immutability of RAW is the point — your compiled
wiki is always traceable back to a source of truth.

**Then run Loop 1 monthly.** Even if it's just 20 minutes. The loop is what
makes the system compound instead of accumulate.

---

## The Compile Discipline

The hardest part of this system isn't building it — it's running the compile
honestly. The compile prompt has to ask: *what in my WIKI contradicts what I'm
seeing in this new data?* Not just "what's new" but "what's wrong."

Most people skip this. They append instead of rewrite. The WIKI grows but never
heals. Within three months it's full of contradictions and the AI is
hallucinating from a confident-sounding but internally inconsistent knowledge
base.

The rule: the compile loop must have **write permission on WIKI**. If you're
only appending, you're not compiling — you're archiving. Archives don't make
your AI partner smarter.

---

## Related

- **[claude-code-hooks-kit](https://github.com/singhjitesh889-blip/claude-code-hooks-kit)** — the automation layer that makes this architecture run without manual effort. Start here after reading ARCHITECTURE.md.
- **[mcp-commerce-starter](https://github.com/singhjitesh889-blip/mcp-commerce-starter)** — an MCP server built on top of this architecture pattern.

---

*I ran this for a year across a six-app solo operation. This repo is the
generalized, domain-free version — conventions, prompts, and the loop. The
methodology is the artifact; adapt the folder names to your domain.*
