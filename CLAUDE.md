# Project context and handover

Drop this file at the root of the repo. Claude Code reads `CLAUDE.md` automatically at the start of a session.

---

## Who I am

- Recent UCR graduate, B.S. Statistics, actuarial track  
- Comfortable with probability and statistical theory. Exam P level material is familiar  
- Beginner at programming. As of today I have no prior Git or Python project experience  
- Two goals at once, and they must not be traded off against each other:  
  1. Build employable, transferable skills (data analyst / actuarial analyst / quant analyst)  
  2. Evaluate whether prop-firm day trading is viable for me, quantitatively rather than by vibes

## Machine state as of this handover

- macOS, zsh shell  
- `git` version 2.39.3 (Apple-supplied) is installed and working  
- Global git identity is configured correctly, one `user.name` and one `user.email`, no duplicates  
- Homebrew was installed to `/opt/homebrew`. Whether the `brew shellenv` line landed in `~/.zprofile` is UNVERIFIED. Check `brew --version` first  
- `gh` (GitHub CLI) is NOT yet installed. Needs `brew install gh`  
- No repository created yet. No Python environment set up yet  
- GitHub account status: not yet confirmed created

First session should finish the setup: verify brew, install gh, `gh auth login`, create the repo, first commit, push.

## How to work with me

- I learn from projects, not from courses. Give me something that runs and either works or errors  
- Structure work in 25 to 40 minute chunks, each ending in one thing that is committed and pushed. Do not hand me a multi-hour task with no checkpoint  
- One project at a time. Finish it before proposing the next  
- Explain the why in two or three sentences, then give me the command or the code. Do not skip the why, and do not write me an essay  
- When I hit an error, walk me through reading the error message rather than just handing me the fix  
- Commit early and often so progress is visible even on unproductive days  
- I have not used the terminal much. Assume nothing about aliases, dotfiles, or tooling I "should" have

## Premise I do not want re-litigated

I already looked at the base rates and I am proceeding with eyes open. Do not spend session time re-arguing whether prop trading is a good idea. The numbers I am working from:

- Prop firm evaluation pass rate is roughly 5 to 10 percent per attempt  
- The FPFX Technology dataset (300,000+ accounts, 100,000 traders, 10 firms) found about 14 percent pass a challenge and about 7 percent of all participants ever receive a payout  
- Average payout when it happens is around 4 percent of funded account size  
- Average spend is roughly $800 across about three attempts  
- Around 70 percent of failures come from breaching loss limits, not from missing profit targets  
- 40 to 50 percent of funded traders lose the account within 90 days  
- Failing traders average about 6.8 trades per day risking 2 to 3 percent each. Passing traders average about 3.2 trades per day risking 0.5 to 1 percent each

Conclusions I have already drawn from these, which should shape the code, not be re-debated:

- Trading is not my income plan. It is a research project that runs alongside getting a job  
- The failure mode is behavioral (overtrading, loss-limit breaches), so every rule must be mechanical and enforced by code, not by willpower in the moment  
- No live or evaluation money until the simulator and backtest say the strategy has positive expectancy AND acceptable survival probability

---

## Project 1: prop firm drawdown survival simulator

This is the first build. It is deliberately scoped small and it doubles as a portfolio piece.

### The question it answers

Given a strategy with a known edge and a firm's specific rule set, what is the probability I pass the evaluation before I breach a limit?

This is a two-barrier absorbing random walk. Reaching the profit target is the upper barrier, breaching trailing max drawdown or the daily loss limit is the lower barrier. I want the probability of each and the distribution of time-to-absorption.

### Inputs

Strategy parameters:

- win rate  
- average win (in R or dollars)  
- average loss  
- trades per day  
- risk per trade as a fraction of account

Firm rule parameters:

- starting account size  
- profit target (dollars or percent)  
- daily loss limit  
- maximum drawdown, with a flag for static vs trailing  
- minimum number of trading days  
- consistency rule, e.g. no single day may exceed 50 percent of total profit

### Outputs

- P(pass), P(breach), P(neither within horizon)  
- Distribution of days to pass and days to breach  
- Which barrier was hit, broken out by daily limit vs max drawdown  
- Expected value in dollars, net of the evaluation fee  
- Sensitivity analysis: how P(pass) moves as risk per trade and trades per day vary. Sweep these two and produce a heatmap

### Build order

1. `README.md` with the question and the method. Commit and push  
2. Single-path simulator. One evaluation, one outcome, printed to stdout  
3. Wrap in a Monte Carlo loop. 10,000 paths, report P(pass)  
4. Add the trailing drawdown logic. This is the subtle part and where most of the realism lives  
5. Add the consistency rule and minimum trading days  
6. Add the sensitivity sweep and the heatmap  
7. Write up findings in the README as if for a hiring manager

### Constraints on the code

- Python 3, standard library plus numpy, pandas, matplotlib. No frameworks  
- Vectorize the Monte Carlo with numpy rather than looping in Python, and explain the speed difference so I learn why  
- Every parameter in a config dict or dataclass at the top, no magic numbers buried in functions  
- Pytest tests on the barrier logic. Deterministic edge cases: a strategy that always wins should pass, one that always loses should breach on day one  
- Set and record the random seed so results are reproducible  
- Type hints and docstrings throughout. This is a portfolio piece

### Why this is a good portfolio piece

It demonstrates Monte Carlo simulation, probability modeling, risk-of-ruin analysis, vectorized numpy, parameter sensitivity analysis, testing, and clear written communication of a quantitative result. Those are the exact bullets on a quant or actuarial analyst job posting.

---

## Skill stack to build, in order

Each item should be learned inside a project, not in the abstract.

1. **Terminal and Git**: navigating with `cd` and `ls`, add / commit / push, branches, `.gitignore`, reading a diff  
2. **Python fundamentals**: functions, dicts, dataclasses, list comprehensions, virtual environments with `venv`, `requirements.txt`  
3. **numpy**: vectorization, broadcasting, random number generation, seeding  
4. **pandas**: loading data, indexing, groupby, resampling time series, joins  
5. **Testing and hygiene**: pytest, type hints, `ruff` for linting, docstrings  
6. **Data acquisition**: pulling market data from an API, handling rate limits, caching to SQLite or Parquet  
7. **Backtesting**: bar-by-bar loop, realistic fees and slippage, no lookahead bias, walk-forward validation  
8. **Statistics applied to trading**: expectancy, variance of returns, Sharpe, maximum drawdown distributions, risk of ruin, Kelly sizing and why fractional Kelly is used in practice  
9. **SQL**: joins, window functions, CTEs. Needed for essentially every data job  
10. **Communication**: a clean README and one chart that makes the finding obvious

## Project sequence after project 1

- **Project 2**: market data pipeline. Pull OHLCV bars from an API into SQLite on a schedule, with proper error handling and caching  
- **Project 3**: honest backtester. Runs a simple rule-based strategy over the stored data, with fees, slippage, and no lookahead  
- **Project 4**: feed the backtest's measured win rate and average win/loss back into the project 1 simulator. That closes the loop and answers the original question with real numbers instead of assumptions  
- **Project 5**: mechanical guardrails. A script that computes position size from the firm's limits and enforces a max-trades-per-day lockout

## Non-negotiables

- No evaluation fee paid until project 4 is complete and the survival probability is known  
- Position size is computed by code, never chosen in the moment  
- Every strategy claim must be backed by a backtest with honest cost assumptions  
- All work lives in Git and is pushed. Unpushed work does not count  
- Career track runs in parallel. Do not let the trading project crowd out job applications and actuarial exam prep

## Parallel career track, for context

Not this repo's job, but it informs priority calls: entry-level actuarial analyst roles want Exam P and FM passed. Data analyst roles want Python, SQL, and a portfolio. Both are served by the same skill stack above, which is why this project sequence was chosen over anything trading-specific and non-transferable.  
