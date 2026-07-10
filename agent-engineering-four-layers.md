# Agent Engineering: Prompt, Context, Harness & Loop

*Agents are a while loop. Everything else is the architecture that wraps it.*

---

Strip away the branding, and every agent is a while loop: call the model, read
what it produced, decide what happens next, repeat until something says stop.
Everything people call "agent engineering" is really engineering the code and
content that wraps that loop — at four different distances from the model.

These four layers don't compete with each other. They nest, like a set of
Russian dolls, with the model sitting at the centre and each layer wrapping
the one before it:

- **Prompt engineering** — one call
- **Context engineering** — everything visible on one turn
- **Harness engineering** — the machinery around a call, and around a full multi-step attempt
- **Loop engineering** — the entire run, start to finish, with no human between turns

---

## Layer 1 — Prompt Engineering

Prompt engineering is the input the model sees on a single call — composed of a
role, instructions, examples, and an output format. Techniques at this layer work
by altering the internal computation and reasoning the model runs, purely because
of the wording it's given.

- **Chain-of-thought** — asking the model to work in steps before answering. Changes
  the reasoning path itself, not just the style of the final answer.
- **Few-shot examples** — two to five input/output pairs that define the format and
  demonstrate edge cases, instead of describing them abstractly.
- **Structured output** — JSON schema or XML tags that force machine-parseable
  output, so downstream code can reliably extract a field instead of scraping prose.
- **Self-consistency** — sample a few independent reasoning chains and take the
  majority answer. Trades tokens and cost for reliability.

**Claude Code equivalent:** task instructions, or a Skill's `SKILL.md` — the role,
the example outputs, the "answer only in JSON" line.

**Ceiling of this layer:** prompt engineering can only improve what happens inside
one call. It has no power over what gets retrieved, what tools exist, or when a
multi-step run decides it's finished.

---

## Layer 2 — Context Engineering

Context engineering covers everything the model sees on a turn — not just the
prompt, but the query, retrieved documents, memory, prior turns, and outputs from
tool calls made earlier in the same run. The context window is finite and fills
fast, so the actual engineering work is curation: rank what matters, cut what
doesn't.

- **Retrieve, then rerank** — pull a wide candidate set, then keep only chunks
  genuinely relevant to this query.
- **Position matters** — models are measurably less accurate using facts buried in
  the middle of a long context. Critical facts get placed near the start or end.
- **Summarize, evict, offload** — old turns get compressed into summaries, stale
  tool outputs get dropped, large blobs get written to disk and referenced by path
  instead of kept live in the conversation.

### Memory

Memory is the part of context engineering concerned with information that must
survive across turns, or across entire sessions.

- **Working memory** — alive in the context window right now, for this turn only.
  Gone the moment it's evicted.
- **Session memory** — state that must persist across many turns within one run.
  Usually a running scratch file or structured state re-injected every turn.
- **Long-term / persistent memory** — facts that should survive across entirely
  separate sessions: a user profile, brand context, or record of past decisions.
- **Procedural memory** — not facts, but *how to do something* — a distilled
  playbook of "this approach worked, this one didn't," written back into a Skill
  so the next run starts smarter without retraining anything.

**The failure mode specific to memory:** staleness and bloat. Memory grows until it
crowds out the current task — or asserts something that was true months ago as if
it's still true today. Good memory systems write selectively and expire deliberately.
This is why a set of narrow, bounded Skills beats one giant ever-growing document —
each is a curated memory store for one job, not a landfill.

**Claude Code equivalent:** pushing large outputs to files instead of chat; STATE
files loaded at session start; the RAW → WIKI → STATE architecture.

---

## Layer 3 — Harness Engineering

The harness is the code wrapped around each model call: it defines the tools
available, parses tool calls, retries on failure, can route sub-tasks to specialised
sub-agents, and verifies the result against something objective — a test suite, a
schema, a rubric — rather than trusting the model's own claim of success.

- **Tool definitions** — the concrete actions the model is allowed to take (Read
  File, Write File, Bash, a Shopify API call), and what each one returns.
- **Call parsing & retries** — when a tool call is malformed or a tool errors out,
  the harness decides whether to retry, reformat, or surface the error back to the
  model to self-correct.
- **Sub-agent routing (orchestrator–worker)** — a lead agent delegates a narrow
  sub-task to a separate sub-agent, then folds the result back in, keeping each
  agent's context focused.
- **Verifiers (evaluator–optimizer)** — a separate check — running the real test
  suite, validating a schema, or having a second model grade against a rubric —
  decides pass or fail. The verifier is deliberately not the same voice that
  produced the output.

**Why this layer punches above its weight:** prompt and context engineering are
about getting one call right. Harness engineering is about everything that has to
be true *around* that call for the whole system to work in the real world — file
paths that resolve, tools that exist, retries that don't loop forever, and a
pass/fail signal that's actually trustworthy.

**Claude Code equivalent:** Skills, hooks, tool permissions, and any script that
validates output before you accept it.

---

## Layer 4 — Loop Engineering

In the usual setup, *you* are the outer loop: write a prompt, read the turns the
agent runs, write the next prompt, repeat. Loop engineering hands that job to the
agent itself — it kicks off on a schedule or an event and runs many turns with no
prompt in between.

The problem this layer exists to solve: a loop left alone doesn't inherently know
when it's finished — and an agent's own report of "done" isn't trustworthy. The
stop condition has to be a real signal, not the model's narration:

- **Turn and token caps** — a hard ceiling that ends a run that's spinning without
  progress, so a stuck loop doesn't burn unlimited cost.
- **A no-progress detector** — catches the agent repeating the same failed call or
  making edits that keep reverting, and halts before more turns are wasted.
- **A completion check run independently** — a separate model, or better, a
  deterministic test, actually verifies the goal state instead of accepting the
  agent's self-report.

A related failure at this layer is **context rot**: over a long run, accumulated
tool outputs, earlier mistakes, and superseded plans degrade later decisions unless
something actively summarises or evicts stale material. This is why L2 and L4 are
tightly coupled — a long-running loop is only as good as its ongoing context hygiene.

**Claude Code equivalent:** a completion check separate from the agent — actually
opening the file, not trusting its summary.

### The decision: what are you handing off?

The right loop type is determined by where *you* are the bottleneck — not by task
complexity:

| Loop type | What you hand off | Best for | Stop mechanism |
|-----------|-------------------|----------|----------------|
| **Turn-based** | The check — you review each output, write the next prompt | Exploring, deciding, one-off tasks | Claude judges; you verify |
| **Goal-based** | The stop condition | Tasks with a verifiable exit criterion | An evaluator model refuses early stops |
| **Time-based** | The trigger | Recurring work, external systems | Work completes or you cancel |
| **Proactive** | The whole prompt | Well-defined recurring streams | Each task exits on goal; routine runs until off |

**The goal-based detail that changes how you think about stop conditions:** when the
stop condition is handed to a *second, evaluator model* — not the same agent that
produced the output — that evaluator actively refuses early stops. Every time the
agent tries to declare done, the evaluator checks the criterion and sends it back to
work if the check fails. This is materially different from the agent self-reporting
completion. Deterministic criteria work best: number of tests passing, a score
threshold, a file at an exact path — things that cannot be rationalized away.

**Quality encoding:** when a loop produces a result that misses, the right response
is not to fix that instance. Encode the fix into the system — write a Skill, add a
stop-hook, extend the verification step — so the next run starts from a better
baseline. This is how a loop gets smarter without more turns: the harness improves,
not the retry count.

---

## Case Study A — "Don't Train the Model, Evolve the Harness"

*Source: Joel Niklaus, Hugging Face / Harvey's Legal Agent Benchmark.*

A frozen, open-weight model was run against a hard legal-agent benchmark and scored
0%. Not "did badly" — zero. The obvious read is that the model can't do legal
reasoning. That read was wrong.

The benchmark's judge checks one brutal thing with zero tolerance: did a file land
at the exact path and filename it expects. The model was consistently doing the
legal analysis correctly, then failing at the very last step — saving under the
wrong filename, dropping the file in a scratch folder, or never writing a file at
all. The 0% was measuring the harness's file-handling discipline, not the model's
legal reasoning.

**The fix — a proposer/keep-if-better loop:**

- A Claude "proposer" looks at where the current harness is failing and proposes
  exactly one new mechanism per iteration — never a batch of changes at once.
- That single change is tested head-to-head against the current best harness.
- It's kept only if it clearly wins; otherwise it's discarded.
- Repeat — so accepted mechanisms compound instead of interfering with each other.

**The outcome:** by the end of the loop, the frozen model — with zero weight changes
— essentially matched Claude Sonnet 4.6 on the benchmark's headline metric, at
roughly 7x lower cost per task.

**Three findings, in order of how surprising they are:**

1. **The single biggest gain was a filing clerk, not a smarter brain.** An automatic
   step that reliably lands the deliverable exactly where the judge expects it beat
   every prompt rewrite at zero extra model tokens. The model wasn't upgraded; it
   just stopped tripping in the last three feet.

2. **Code fixes transferred across models; prompt playbooks did not.** The same
   harness mechanism lifted a different, smaller model from the same family by 14
   points with no retuning. But a prompt playbook tuned for one model actively hurt
   a different model family — even on tasks it could already finish. Harness code
   behaves like durable infrastructure; a tuned prompt behaves like model-specific
   folklore.

3. **The harness swamped every other variable measured.** Same model, same judge,
   same tasks — swapping which harness wrapped the model swung the score between
   3.5% and 80.1%. Until the harness is controlled for, a benchmark score tells you
   "how good is this model plus whatever wrapper happened to surround it" — not "how
   good is this model."

**Where it stops working:** the gains flatten eventually. Once filing mistakes,
formatting slips, and tool-call fumbles are cleaned up, what's left is a genuine
capability gap — no amount of scaffolding papers over "didn't know the answer."
Harness engineering fixes "knew the right answer, tripped on delivery." It cannot
fix the missing knowledge.

---

## Case Study B — Harvey's Legal Agent Benchmark (internal experiment)

A separate experiment run internally by Harvey on twelve real legal tasks — lease
review, drafting complaints, tax memos, due-diligence responses, and similar work —
each with source documents, instructions, and a detailed grading rubric.

**The loop:**
1. The agent attempts the task.
2. An LLM judge scores the attempt and writes specific feedback: what was right,
   what was missed, where the reasoning broke.
3. A separate agent reads that feedback, spots recurring failure patterns, and edits
   the harness — writing new skills (cross-document review playbooks), new stop-hooks
   (block the run from ending until a deliverable passes a check), or new output
   templates.
4. Rerun, rescore, repeat.

**The result:** five of the twelve tasks started between 2–7% success. After the
loop ran, average performance across all twelve tasks moved from **40.8% to 87.7%**.
Every single task improved. Several finished above 90%.

**What connects Case A and Case B:** both are the same shape of loop — attempt,
independent judgment, harness edit, retry — applied at different scales. Both land
on the same conclusion: the fastest, most durable gains came from fixing what
surrounds the model, not the model itself.

---

## Synthesis — One Table to Diagnose Any Agent

| Layer | What it controls | Typical failure signature | Claude Code equivalent |
|-------|-----------------|--------------------------|----------------------|
| Prompt | Wording, format, examples, reasoning style of one call | Reasoning is fine but output can't be parsed, or an edge case never shown gets missed | Task instructions, or a SKILL.md's example section |
| Context | What's visible this turn: retrieved docs, memory, prior turns | Model contradicts itself, "forgets" something from 10 turns ago, or drowns in irrelevant material | Pushing large outputs to files instead of chat; STATE files; RAW→WIKI→STATE |
| Harness | Tools, retries, sub-agents, verification | Model reasons correctly but the deliverable is missing, misnamed, or never checked | Skills, hooks, tool permissions, output validation scripts |
| Loop | The whole run and its stop condition | Agent declares "done" while the real test still fails, or a run spins forever | A completion check separate from the agent — actually opening the file |

---

## Diagnostic Order

When an agent-driven workflow underperforms, check in this order — cheapest and
most-likely-to-be-the-real-problem first:

1. **Harness first** — did the model actually do the job correctly, but save, output,
   or format it in the wrong place — with nobody checking it? Case Study A should
   make this the default suspect. Second repeat failure of the same kind → hard-code a
   deterministic check; don't re-prompt and hope.

2. **Context next** — did the model have the right reference material visible at the
   moment it needed it, or was the relevant context buried, evicted, or never loaded?

3. **Loop next** — if this runs unattended, does something other than the agent's own
   word confirm it actually finished correctly?

4. **Prompt last** — only once the above are ruled out is this actually a wording or
   reasoning problem worth rewriting instructions for.

---

## The One Line to Keep

A benchmark score — or a bad output from any agent — measures the model and its
harness together, and until the harness is controlled for, it's impossible to know
which one actually failed.

Model swaps are expensive and slow. Harness fixes are cheap, fast, and — per Case
Study A — the improvements often transfer to other models for free.
