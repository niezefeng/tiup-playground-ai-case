---
name: tiup-playground
description: Reproduce, isolate, and verify TiDB, TiKV, and PD issues by bootstrapping local clusters with `tiup playground`. Use when a user asks to verify a TiDB issue with TiUP, reproduce a bug or regression on a specific TiDB version (stable or nightly), test custom topology or config flags (`--db/--kv/--pd`, `--*.config`, `--*.binpath`), inspect running playground instances (`display`), or scale instances in or out for debugging.
---

# TiUP Playground

Use this skill to run local, reproducible TiDB experiments with TiUP and capture evidence for issue verification.

## Workflow

1. Gather reproduction inputs
- Ask for TiDB version (`vX.Y.Z` or `nightly`), topology, SQL or workload, and expected versus actual behavior.
- If details are missing, begin with the smallest default cluster and iterate.

2. Check local prerequisites
```bash
tiup --version
tiup playground --help
```
If `tiup` is not on `PATH`, use `/Users/anthonynie/.tiup/bin/tiup`.
If sandbox denies writes under `~/.tiup`, rerun commands with elevated permissions.

3. Start a playground cluster
- Run in a foreground TTY by default. `tiup playground` is a supervisor process; if it exits, playground control commands stop working.
- Default latest stable:
```bash
tiup playground --tag <tag>
```
- Fast minimal startup for SQL-only repro:
```bash
tiup playground --db 1 --kv 1 --pd 1 --tiflash 0 --without-monitor --tag <tag>
```
- Specific version:
```bash
tiup playground v8.5.5 --tag <tag>
```
- Nightly:
```bash
tiup playground nightly --tag <tag>
```
- Multi-node topology:
```bash
tiup playground v8.5.5 --db 3 --pd 3 --kv 3 --tag <tag>
```
- Custom config or local binary:
```bash
tiup playground --pd.config ~/config/pd.toml --tag <tag>
tiup playground --db.binpath /path/to/tidb-server --tag <tag>
```
- Avoid port conflicts with another playground:
```bash
tiup playground v8.5.5 --port-offset 10000 --tag <tag>
```
- If background is required, use a persistent terminal manager (for example `tmux`) instead of plain `nohup`.

4. Connect and reproduce
```bash
tiup client
```
Or use non-interactive probes:
```bash
mysql -h 127.0.0.1 -P 4000 -u root -e "SELECT tidb_version();"
```
For automation, poll readiness before running repro SQL:
```bash
for i in {1..36}; do
  mysql -h 127.0.0.1 -P 4000 -u root -e "SELECT tidb_version();" && break
  sleep 5
done
```
Run the provided SQL or workload to reproduce the issue.
Capture `SELECT tidb_version();` in results for version evidence.

5. Inspect state and adjust topology
- Show instances:
```bash
tiup playground display --tag <tag>
```
- Scale out:
```bash
tiup playground scale-out --tag <tag> --db 1
```
- Scale in:
```bash
tiup playground scale-in --tag <tag> --pid <pid>
```
Use `display` to identify `pid` values before scaling in.
After `scale-out --db 1`, TiUP prints the new TiDB port; verify it with a targeted query if needed.

6. Capture evidence and clean up
- Keep `--tag` so data remains in `~/.tiup/data/<tag>` after stop.
- Collect logs and configs from the tagged data directory for issue reports.
- Stop foreground playground with `Ctrl+C` in the running session.
- Clean old instances when needed:
```bash
tiup status
tiup clean <instance-name>
```
- If `tiup status` shows `TERM` records from prior failed runs, clean those instance names explicitly.
- Use `tiup clean --all` only when you intentionally want to remove all TiUP playground instances.

## Scope Boundary

This skill is for local issue verification with `tiup playground`, not production deployment. If the task is about real cluster lifecycle operations (`deploy`, `upgrade`, `scale-out` on remote hosts), use `tiup cluster` commands in a separate workflow.

## Reference File

For command templates and quick recipes, read [references/playground-commands.md](references/playground-commands.md).
