# Supply Chain - Message Drop Experiment

**Scenario:** `supply_chain`
**Setting changed:** `failures.message_drop`, `0.0` to `0.1`

## Goal

These are Tier-1 agents, simple deterministic/rule-based scripts, not agents
making decisions with economic incentives. So instead of testing behavior, I
tested how the model handles failure in a multi-step pipeline, using
`supply_chain`'s 4-stage chain (material, product, shipment, delivered).

## Hypothesis

With a 10% failure chance at each of the 4 stages, completion should drop to
about `15 * (0.9)^4 = 10` items.

## What happened

- **Baseline:** all 15 items completed (60 sent, 60 received).
- **After the change:** 6 items completed (40%). 9 messages dropped, split
  as 3 at material, 2 at product, 2 at shipment, 2 at delivered.
- The built-in validator counted only 7 incomplete (not 9), because it
  defines "delivered" as sent-by-retailer, not received-by-supplier. It
  doesn't check whether the final confirmation actually arrived. That's not
  what I expected.

## What surprised me

Each stage showed 17 to 25% of messages dropped, well above the configured
10%. Sample size is small (8 to 15 messages per stage), but the gap was
bigger than I expected. I checked the config was actually set to 0.1 (it
was), then used Claude Code to run the same scenario across more seeds to
see if it converges: average drop rate was 10.9% after 10 runs and 10.8%
after 20 runs, confirming it does converge to around 10%, and the single run
I first looked at was just an unlucky sample.

## Reproducing

```bash
cd experiments/supply-chain-message-drop
nest run supply-chain-baseline.yaml
nest run supply-chain-message-drop.yaml
nest inspect ./evidence/baseline-trace.jsonl
nest inspect ./evidence/message-drop-trace.jsonl
```

Both traces and their `nest inspect` output are checked in under `evidence/`.

## Use of AI

Used Claude Code to read the NANDA Town source directly (scenario, agent,
validator code), run the scenarios, parse trace files for the per-stage drop
counts, and run the multi-seed sweep to check convergence.
