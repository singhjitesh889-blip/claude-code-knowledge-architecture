# The Compile Loop

The compile loop is what separates a knowledge system that compounds from one that accumulates.

Run it monthly. Or whenever you have new data that should change how your AI partner thinks.

---

## What It Does

Takes new RAW data → updates WIKI to reflect current reality → distills changes into STATE.

Not a summary. Not an append. A rewrite that asks: *what was true before that isn't true now?*

---

## When to Run

- Monthly, after data exports from your key systems
- After a major strategic decision that changes operating rules
- Whenever you notice the AI giving outdated advice (sign that WIKI is stale)
- After any experiment concludes with a clear result

---

## The Compile Prompt

Copy this into a Claude Code session. Adjust the `[brackets]` to your domain.

```
I'm running my monthly knowledge compile. I have new [domain] data to process.

New data is in: raw/monthly/[YYYY-MM]-[domain].csv
(and any other files I've dropped in raw/ since last compile)

Your job is to update WIKI to reflect what this new data tells us.

Step 1 — Read current state
  Read wiki/[relevant-file].md
  Read state/STATE.md
  Note: what are the current active rules and hypotheses?

Step 2 — Process new data
  Read the new raw files I've dropped.
  Extract: what patterns are confirmed? What's new? What contradicts something in WIKI?

Step 3 — Reconcile
  For each current rule in WIKI:
    - Does new data confirm it? → keep, update with new evidence date
    - Does new data contradict it? → revise or remove, note what changed
    - Is new data irrelevant? → leave unchanged
  
  For each new pattern in the data:
    - Seen once → add to WIKI as HYPOTHESIS
    - Seen across multiple data points → add as RULE
    - Changes an existing RULE → rewrite the rule with the new evidence

Step 4 — Rewrite WIKI
  Rewrite wiki/[relevant-file].md with updated rules, patterns, and evidence.
  Not append — rewrite. Stale content must be removed or updated.
  Keep it under [your token target, e.g. 1,200 tokens].

Step 5 — Update STATE
  Update state/STATE.md:
    - Bump `last_updated` to today
    - Update `active_rules` to match new WIKI (3-5 highest-confidence rules)
    - Move confirmed hypotheses to rules; remove refuted ones
    - Update `current_numbers` with latest metrics

Step 6 — Log it
  Append one line to compile-log.md:
    [date] | compiled [domain] | [N] rules updated | [N] hypotheses added | [key change in one line]

Ask me to confirm before rewriting any file. Show me the diff of what changed and why.
```

---

## Compile Disciplines

**Show the diff.** The compile should surface what changed — which rules were added, removed, or modified. If nothing changed, that's a signal the data wasn't processed correctly, or you're in a stable period (both valid, but worth knowing explicitly).

**Rewrite, don't append.** A WIKI file that only ever grows is not a compiled knowledge base — it's an archive. Archives don't help the AI reason; they just add noise. The compile loop's value is in the removal and rewriting.

**Token budget your WIKI.** Set a target token count for each WIKI file (e.g. creative-brief → ≤1,200 tokens, customer → ≤800 tokens). When the compile would exceed budget, old lower-confidence content is removed to make room for new higher-confidence content. This forces prioritization.

**Separate confirmation from discovery.** The compile should distinguish between:
- *Confirming* something already in WIKI (raises confidence)
- *Discovering* something new (starts as hypothesis)
- *Contradicting* something in WIKI (requires explicit reconciliation)

Most compile systems only do discovery. The confirmation and contradiction passes are what actually make the system trustworthy.

---

## Example: What a Compile Looks Like

Before compile (`wiki/creative-brief.md` excerpt):
```
HYPOTHESIS: Short-form video outperforms static in Q4
RULE: Benefit-first headlines convert better than curiosity-first
RULE: Price anchoring in first 3 seconds lifts conversion
```

New data: Q4 video results came in. Video lifted CTR 40% but conversion rate was flat vs static.

After compile:
```
RULE: Short-form video outperforms static on CTR, not on conversion rate
  Evidence: Q4 A/B, n=8 campaigns. CTR +40%, CVR flat.
  Note: Use video for top-of-funnel, static for retargeting.
RULE: Benefit-first headlines convert better than curiosity-first  
  [unchanged — still confirmed by new data]
RULE: Price anchoring in first 3 seconds lifts conversion
  [unchanged — still confirmed by new data]
```

The hypothesis became a more nuanced rule. This nuance is what the AI needs to give you useful creative direction — not just "video works" but "video works for this, not that."

---

## Compile Log Format

Keep a running log so you can see the arc of what's changed over time.

```
compile-log.md

2026-03-01 | compiled creative + customer | 3 rules updated, 2 hypotheses added | video nuanced: CTR↑ CVR flat
2026-02-01 | compiled creative | 1 rule added, 1 removed | price anchoring confirmed, curiosity headlines retired
2026-01-01 | compiled market | 2 hypotheses added | Q4 demand spike confirmed, off-season pattern flagged
```

After 6 months of compiles, this log tells a story. The AI can read it and understand not just what's true now, but how the understanding evolved. That arc is context money can't buy.
