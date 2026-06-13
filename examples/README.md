# Examples — Before and After a Compile

This folder shows what the compile loop actually does to your knowledge system.

## What's Here

```
before-compile/
  raw-2024-11-performance.md      ← raw data dropped by the user
  wiki-creative-brief-oct.md      ← WIKI state before the compile

after-compile/
  wiki-creative-brief-nov.md      ← WIKI state after processing November data
```

## What the Compile Did

The raw November data confirmed three hypotheses from October:
- Video lifts CTR but not CVR → upgraded from hypothesis to rule (nuanced)
- Benefit framing beats price framing → upgraded from hypothesis to rule
- Retargeting 3-7 day window outperforms 1-2 day → upgraded from hypothesis to rule

**Result:** The WIKI after the compile has *fewer* hypotheses and *more* rules. It's smaller and more confident. This is the correct direction — a compile should increase signal density, not add volume.

The last table in `after-compile/wiki-creative-brief-nov.md` shows exactly what changed and why. Every compile should surface a version of that table — what upgraded, what was removed, what's still provisional.

## The Domain Here

These examples use ad performance data because it's universally legible. The architecture works for any domain where:
- You have periodic data inputs (monthly, weekly, whatever your cycle is)
- You're trying to accumulate durable rules from noisy data
- The AI needs domain knowledge that compounds across sessions

Swap "creative brief" for "product decisions," "customer research," "market analysis," "codebase architecture" — the loop is the same.
