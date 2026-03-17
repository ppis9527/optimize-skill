# Safety Checklist — MUST verify before EVERY iteration

## Pre-Loop Checks (once at start)
- [ ] `git status` shows clean working tree (safe mode)
- [ ] Verify command runs successfully and outputs a number
- [ ] Scope files exist and are readable
- [ ] No .env / credential files in scope
- [ ] Max iterations set (default 20)

## Per-Iteration Checks (every cycle)

### Safe Mode
- [ ] Change is atomic (ONE logical modification)
- [ ] Only scoped files are modified
- [ ] Diff shown to user
- [ ] User confirmed (y/n)
- [ ] Only modified files staged (`git add <file>`, never `-A`)
- [ ] Commit message starts with "optimize:"
- [ ] If metric worse: `git checkout -- <file>` to revert (not reset --hard)

### Night Mode
- [ ] No files modified (verify with `git status`)
- [ ] No git operations performed
- [ ] Results written only to optimize-report-*.md or optimize-results.tsv
- [ ] All analysis done in memory or via read-only scripts

## Per-Iteration Safety (v1.1)
- [ ] **Degradation check**: metric vs baseline — if worse by >20%, auto-revert + warn
- [ ] **Credential content scan**: grep scope file for `api_key=`, `token:`, `secret`, `password`, `private_key`, `Bearer ` — block if found
- [ ] **History dedup**: check optimize-results.tsv for same parameter change — skip if duplicate
- [ ] **Anomaly check**: metric = 0, negative, or >10x baseline → auto-abort (likely bug)

## Abort Conditions (stop immediately)
- Uncommitted changes detected mid-loop
- Verify command fails to produce a number
- User types "stop" or "abort"
- 3 consecutive errors
- Iteration count reaches max
- Metric anomaly detected (0, negative, or >10x spike)
- Metric degradation >20% from baseline
