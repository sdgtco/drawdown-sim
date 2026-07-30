# Prop Firm Drawdown Survival Simulator

What are the odds of passing a prop firm evaluation before breaching one of its loss limits?

Most discussion of these evaluations focuses on the profit target. Hit 8% and you get funded. But the profit target is rarely what stops people. The loss limits are, and they get a lot less attention. This project models both and reports the probability of each outcome.

## The model

An evaluation is a random walk with two absorbing barriers. Account equity moves trade by trade and the run ends when it touches either one.

Upper barrier, pass: cumulative profit reaches the target, with minimum trading days and the consistency rule satisfied.

Lower barrier, breach: equity drops below the daily loss limit or the maximum drawdown floor.

Neither barrier is guaranteed to get hit, so I impose a horizon and report "neither" as its own outcome rather than folding it into the other two.

The hard part is the trailing maximum drawdown. Under a trailing rule the lower barrier moves. It ratchets up as the account makes new equity highs, and at most firms it stops ratcheting once you are up by the full drawdown amount. So a winning streak raises the floor you can later fall through, and a trader can be net profitable and still breach. Getting this right is most of the work, and it is why a closed-form risk of ruin formula does not cover the problem.

## Method

Monte Carlo simulation. Each path draws a sequence of trade outcomes from the strategy parameters, walks the equity curve forward under the firm's rules, and records which barrier it hit and on what day. Run 10,000 paths and the outcome probabilities and the time-to-absorption distributions fall out of the counts.

The simulation is vectorized with numpy, so all paths advance together as array operations instead of one at a time in a Python loop. The random seed is recorded so the numbers here can be reproduced.

### Inputs

Strategy:

- win rate
- average win
- average loss
- trades per day
- risk per trade, as a fraction of the account

Firm rules:

- starting account size
- profit target
- daily loss limit
- maximum drawdown, static or trailing
- minimum trading days
- consistency rule (no single day above a set share of total profit)

### Outputs

- P(pass), P(breach), P(neither within horizon)
- distribution of days to pass and days to breach
- breach cause, daily loss limit vs maximum drawdown
- expected value in dollars, net of the evaluation fee
- sensitivity heatmap of P(pass) against risk per trade and trades per day

## Why sweep those two parameters

FPFX Technology published data covering 300,000+ evaluation accounts across 10 firms. Roughly 70% of failures came from breaching a loss limit rather than from failing to reach the profit target. The behavioral split is wide. Traders who failed averaged about 6.8 trades per day risking 2 to 3% each. Traders who passed averaged about 3.2 trades per day risking 0.5 to 1%.

That points at position size and trade frequency mattering more than edge, at least across the range people actually trade in. The heatmap is there to test whether that holds up under an explicit model.

## Status

Work in progress.

- [x] README with the question and the method
- [ ] single-path simulator, one outcome to stdout
- [ ] Monte Carlo loop, 10,000 paths, P(pass)
- [ ] trailing drawdown logic
- [ ] consistency rule and minimum trading days
- [ ] sensitivity sweep and heatmap
- [ ] writeup of findings

## Running it

```
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Python 3 with numpy, pandas, matplotlib and pytest. No frameworks.
