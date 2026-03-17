# /optimize — Safe Iterative Optimization Skill for Claude Code

A safety-first alternative to [autoresearch](https://github.com/uditgoenka/autoresearch). Designed for iterative optimization with strict guardrails — no unbounded loops, no destructive git operations, no auto-deploy.

## Two Modes

| Mode | Command | Use Case |
|------|---------|----------|
| **Safe** | `/optimize safe <goal>` | Code changes with human confirmation each step |
| **Night** | `/optimize night <goal>` | Read-only analysis overnight, report in the morning |

## Safety Guardrails

What makes this different from autoresearch:

| | autoresearch | /optimize |
|--|-------------|-----------|
| Loop limit | Unbounded (never stops) | **Max 20 iterations** |
| Human review | None | **Safe mode: confirm each change** |
| Night mode | None | **Read-only, zero file modifications** |
| `git add` | `-A` (stages everything incl. .env) | **Only named files** |
| `git reset` | `--hard` (destroys uncommitted work) | **`checkout --` or `revert`** |
| Auto deploy | `--auto` push to prod | **Never pushes** |
| Secrets | No protection | **Auto-excludes .env/credentials** |

## Use Cases

### Strategy Backtesting (Night Mode)
```
/optimize night "Find best entry filters for strategy2, metric=PF, baseline=1.50"
```
Runs overnight, explores different filter combinations, produces a report.

### Test Coverage (Safe Mode)
```
/optimize safe "Improve test coverage to 80%, metric=pytest --cov"
```
Writes one test at a time, shows diff, waits for your OK before committing.

### Security Audit (Night Mode)
```
/optimize night "Scan for OWASP Top 10 vulnerabilities"
```
Read-only scan, writes findings to report file.

### Performance Tuning (Safe Mode)
```
/optimize safe "Reduce scan_for_signals execution time, metric=elapsed seconds"
```

## How It Works

```
Setup Phase:
  1. Parse goal + define verify command (must output a number)
  2. Define scope (which files to modify/analyze)
  3. Run baseline measurement
  4. Display config → user confirms

Loop Phase (max 20 iterations):
  Safe Mode:                          Night Mode:
  ┌─────────────────────┐             ┌─────────────────────┐
  │ Analyze what to try  │             │ Analyze what to try  │
  │ Make ONE change      │             │ Run read-only test   │
  │ Show diff → WAIT     │             │ Record suggestion    │
  │ User confirms (y/n)  │             │ (no file changes)    │
  │ Run verify command   │             │ Log to report        │
  │ Keep or revert       │             │                      │
  │ Log to TSV           │             │ Log to TSV           │
  └─────────────────────┘             └─────────────────────┘

Summary: baseline → best, all keep/discard decisions
```

## Installation

### Claude Code

Copy to your skills directory:

```bash
cp -r skills/optimize ~/.claude/skills/optimize
```

### Gemini CLI

Add the `/optimize` section from `gemini-snippet.md` to your `~/.gemini/GEMINI.md`.

## File Structure

```
skills/optimize/
├── SKILL.md                    # Main skill definition + safety rules
└── references/
    └── safety-checklist.md     # Per-iteration safety verification checklist
```

## Results Format

Each iteration logs to `optimize-results.tsv` (never committed):

```
iteration  mode      status   metric  delta   description
0          baseline  -        1.50    -       Current PF
1          night     suggest  1.80    +0.30   Add 3-day consecutive drop filter
2          night     suggest  1.52    +0.02   Add MACD > 0 filter
3          night     reject   1.21    -0.29   RSI < 45 too strict
```

## License

MIT
