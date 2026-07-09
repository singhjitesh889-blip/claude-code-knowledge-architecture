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

## Where This Fits in the Stack

Strip away the branding from any agent system and you find four engineering layers,
each at a different distance from the model. This repo covers **Layer 2 — Context
Engineering**: everything the model can see on a given turn.

| Layer | Controls | Typical failure | This repo's role |
|-------|----------|-----------------|-----------------|
| L1 — Prompt | Wording, format, reasoning in one call | Output correct but unparseable; edge case missed | Out of scope — task instructions live in Skill files |
| **L2 — Context** | Everything visible this turn: retrieved docs, memory, prior turns | Model contradicts itself; forgets something from 10 turns ago | **This repo — RAW → WIKI → STATE** |
| L3 — Harness | Tools, retries, sub-agents, verifiers around each call | Model reasons correctly — deliverable missing, misnamed, never checked | [claude-code-hooks-kit](https://github.com/singhjitesh889-blip/claude-code-hooks-kit) |
| L4 — Loop | The full run: scheduler, token cap, stop condition | Agent declares done while real test still fails; run spins forever | Cron-triggered pipelines with independent completion checks |

The layers nest. A better context layer makes the harness more reliable. A better
harness makes the loop safer to run unattended. L2 is the right place to start
because it's the one that compounds over time.

---

### Why the harness is usually the real variable

A frozen open-weight model once scored 0% on Harvey's Legal Agent Benchmark — not
because of reasoning failure, but harness failure. The model understood the tasks.
It failed because outputs landed in the wrong folder, with the wrong filename, or
were never written at all. The 0% was measuring the harness's file-handling
discipline, not the model's legal reasoning. No amount of RAW → WIKI → STATE
architecture would have fixed it. The context was fine. The harness wasn't there.

The fix was a proposer/keep-if-better loop: one new harness mechanism proposed per
iteration, tested head-to-head, kept only if it wins. The biggest single gain was a
filing clerk — an automatic step that reliably lands the deliverable at the exact
expected path. It beat every prompt rewrite. It added zero extra model tokens. The
same model, same tasks, same judge — score swung from 3.5% to 80.1% by swapping the
harness. A related experiment across 12 real legal tasks moved average performance
from 40.8% to 87.7% using the same loop: attempt → independent judge → harness edit
→ rerun.

One finding that changes how to diagnose failures: harness fixes transferred across
models — the same harness lifted a separate smaller model 14 points — while prompt
playbooks tuned for one model actively hurt a different model family. A benchmark
score measures the model and its harness together. Until the harness is controlled
for, it's impossible to know which one failed.

---

### Diagnosing failures: start with the harness

When a run produces the wrong result, work outward from the model:

1. **Harness first** — did the model do the job correctly but save, output, or format
   it in the wrong place, with nobody checking? A missing verifier explains more
   failures than poor reasoning. Second repeat failure of the same kind → hard-code a
   deterministic check; don't re-prompt and hope.

2. **Context next** — did the model have the right reference material visible when it
   needed it? A stale STATE file, a missing WIKI entry, or a context window that
   evicted the relevant file is the second-most-common failure class. The RAW →
   WIKI → STATE architecture is the fix for this.

3. **Loop next** — if this runs unattended, does something independent of the agent
   confirm it finished correctly? The completion check has to open the file, not ask
   whether it was opened.

4. **Prompt last** — only once the above are ruled out is this a wording or reasoning
   problem. Prompt rewrites are the most expensive fix and the least transferable.

The ordering matters because most people go straight to step 4.

---

## Related

- **[claude-code-hooks-kit](https://github.com/singhjitesh889-blip/claude-code-hooks-kit)** — the automation layer that makes this architecture run without manual effort. Start here after reading ARCHITECTURE.md.
- **[mcp-commerce-starter](https://github.com/singhjitesh889-blip/mcp-commerce-starter)** — an MCP server built on top of this architecture pattern.

---

*I ran this for a year across a six-app solo operation. This repo is the
generalized, domain-free version — conventions, prompts, and the loop. The
methodology is the artifact; adapt the folder names to your domain.*
