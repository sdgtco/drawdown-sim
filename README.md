# Prop Firm Drawdown Survival Simulator

**What is the probability of passing a prop firm evaluation before breaching one of its loss limits?**

Prop firm evaluations are usually discussed in terms of the profit target — "hit 8% and you're funded." That framing is misleading. The evaluation is a race between two barriers, and the losing barrier is the one that is almost always hit first. This project models that race explicitly and reports the probability of each outcome.

## The model

An evaluation is a **two-barrier absorbing random walk**. Account equity moves trade by trade, and the walk ends when it touches either:

- **Upper barrier (pass):** cumulative profit reaches the target, with the minimum-trading-days and consistency rules satisfied
- **Lower barrier (breach):** equity falls below the daily loss limit or the maximum drawdown floor

Neither barrier is guaranteed to be reached, so a horizon is imposed and "neither" is reported as a distinct third outcome rather than being folded into one of the other two.

The subtlety is the **trailing** maximum drawdown. Under a trailing rule the lower barrier is not fixed — it ratchets upward as the account makes new equity highs, and (at most firms) it stops ratcheting once the account is up by the drawdown amount. This means a winning streak *raises the floor you can later fall through*. A trader can be net profitable and still breach. Modeling this correctly is where most of the realism lives, and it is the reason a closed-form ruin formula is not sufficient here.

## Method

Monte Carlo simulation. Each path draws a sequence of trade outcomes from the strategy's parameters, walks the equity curve forward under the firm's rule set, and records which barrier was hit and on what day. Aggregating 10,000+ paths gives the outcome probabilities and the full distribution of time-to-absorption.

The simulation is vectorized with numpy — all paths are advanced in parallel as array operations rather than looped one at a time in Python. The random seed is recorded so every result in this README is reproducible.

### Inputs

**Strategy:** win rate · average win · average loss · trades per day · risk per trade (fraction of account)

**Firm rules:** starting account size · profit target · daily loss limit · maximum drawdown (static or trailing) · minimum trading days · consistency rule (no single day exceeds a set share of total profit)

### Outputs

- P(pass), P(breach), P(neither within horizon)
- Distribution of days to pass and days to breach
- Breach cause attribution: daily loss limit vs. maximum drawdown
- Expected value in dollars, net of the evaluation fee
- Sensitivity heatmap: how P(pass) responds to risk per trade and trades per day

## Why these two parameters get the sensitivity sweep

Industry data on prop evaluations (FPFX Technology; 300,000+ accounts across 10 firms) finds roughly 70% of failures come from breaching loss limits rather than from failing to reach the profit target. The behavioral split is stark: traders who fail average ~6.8 trades per day risking 2–3% each, while traders who pass average ~3.2 trades per day risking 0.5–1% each.

That points at position size and trade frequency — not edge — as the dominant controllable variables. The heatmap is the deliverable that tests whether that holds under an explicit model.

## Status

Under construction. Build order:

- [x] README with the question and the method
- [ ] Single-path simulator, one outcome to stdout
- [ ] Monte Carlo loop, 10,000 paths, P(pass)
- [ ] Trailing drawdown logic
- [ ] Consistency rule and minimum trading days
- [ ] Sensitivity sweep and heatmap
- [ ] Written findings

## Running it

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Python 3, numpy, pandas, matplotlib, pytest. No frameworks.
