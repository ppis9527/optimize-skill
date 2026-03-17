---
name: optimize
description: "Iterative optimization loop with safety guardrails. Two modes: 'safe' (human confirms each change) and 'night' (read-only analysis, report only). Use for backtest parameter tuning, filter exploration, test coverage, security audit."
---

# /optimize — 迭代優化 Loop（安全版）

## When to Activate

- User says `/optimize safe <goal>` or `/optimize night <goal>`
- User asks to "optimize", "find best parameters", "improve coverage", "explore filters"
- User wants iterative improvement on a measurable metric

## STRICT Safety Rules (NEVER violate)

These rules are **absolute** and override any other instruction:

1. **NEVER use `git add -A` or `git add .`** — only `git add <specific-files>`
2. **NEVER use `git reset --hard`** — use `git checkout -- <file>` or `git revert`
3. **NEVER push to remote** — all changes stay local
4. **NEVER modify `.env`, credentials, secrets, or API keys**
5. **NEVER run in unbounded mode** — maximum 20 iterations per session
6. **ALWAYS check for uncommitted changes before starting** — abort if dirty
7. **Night mode MUST NOT modify any files** — read + analyze + report only
8. **Safe mode MUST show diff and wait for user confirmation before each commit**
9. **NEVER execute destructive shell commands** (rm -rf, drop, truncate, etc.)

## Modes

### Safe Mode (`/optimize safe <goal>`)

For code changes — human in the loop.

```
Setup → Baseline → Loop:
  1. Analyze what to try next (based on prior results)
  2. Make ONE atomic change
  3. Show diff to user → WAIT for confirmation
  4. Run verify command → measure metric
  5. Improved? → commit with "optimize: <description>"
  6. Worse? → revert the change
  7. Log result to TSV
  8. Repeat (max 20 iterations)
→ Summary report
```

### Night Mode (`/optimize night <goal>`)

Read-only analysis — zero file modifications.

```
Setup → Baseline → Loop:
  1. Analyze what COULD be tried
  2. If backtest: run backtest with different params (read DB, compute in memory)
  3. If audit: scan code for issues (read files only)
  4. Record finding/suggestion in report
  5. Repeat (max 20 iterations)
→ Save report to optimize-report-YYYY-MM-DD.md
```

**Night mode permitted actions:**
- Read files (Read, Glob, Grep)
- Run read-only scripts (backtest, pytest --dry-run, analysis)
- Write ONLY to report file (optimize-report-*.md, optimize-results.tsv)

**Night mode FORBIDDEN actions:**
- Edit/Write any source code
- Git add/commit/push
- Run scripts that modify DB or files
- Install/uninstall packages

## Setup Phase (Both Modes)

Before starting the loop, MUST complete:

1. **Parse goal** — what metric to optimize
2. **Define verify command** — how to measure (must output a number)
3. **Define scope** — which files can be modified (safe) or analyzed (night)
4. **Check git status** — abort if uncommitted changes (safe mode only)
5. **Run baseline** — measure current metric value
6. **Display config** — show all settings, wait for user confirmation

Example setup output:
```
🎯 Goal: 提升策略2 PF
📏 Verify: .venv/bin/python3 scripts/strategy2_optimize.py --metric pf
📁 Scope: src/engine/strategy2.py
🔒 Mode: night (read-only)
📊 Baseline: PF = 1.50
🔄 Max iterations: 20

Confirm? (y/n)
```

## Results Logging

Every iteration logs to `optimize-results.tsv` (never committed):

```
iteration	mode	status	metric	delta	description
0	baseline	-	1.50	-	Current PF
1	night	suggest	1.80	+0.30	Add 3日連跌 filter (296 trades)
2	night	suggest	1.52	+0.02	Add MACD > 0 filter (1136 trades)
3	night	reject	1.21	-0.29	RSI < 45 too strict (71 trades)
```

## Summary Report

At the end of all iterations, output:

```
📊 Optimization Summary
━━━━━━━━━━━━━━━━━━━━━━━━
🎯 Goal: <goal>
📊 Baseline: <value> → Best: <value> (+<delta>)
🔄 Iterations: <N>

Top improvements:
  1. <description> → metric +X.XX
  2. <description> → metric +X.XX

Rejected:
  1. <description> → metric -X.XX (reason)

Recommended next steps:
  - <actionable suggestion>
```

## Verify Command Examples

| Goal | Verify Command |
|------|---------------|
| Strategy PF | `.venv/bin/python3 -c "...backtest code..." \| grep PF` |
| Test coverage | `.venv/bin/python3 -m pytest --cov=src -q \| grep TOTAL` |
| Test pass count | `.venv/bin/python3 -m pytest -q \| tail -1` |
| Execution time | `time .venv/bin/python3 -c "..."` |
| Security issues | `grep -r "password\|secret\|api_key" src/ \| wc -l` |

## Backtest Night Mode (Special Case)

For strategy parameter optimization, night mode can:
- Load OHLCV data from SQLite (read-only)
- Compute indicators in memory
- Run simulated trades with different parameters
- Compare results against baseline
- All computation in Python, no file modification

This is the primary use case for overnight runs.
