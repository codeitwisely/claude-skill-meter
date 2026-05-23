# Contributing

Feedback, bug reports, and PRs are welcome.

## Ideas worth contributing

- **Accurate token tracking** — if Claude Code exposes session token counts (via logs, env vars, or future hook payloads), a PR adding real token counts would be the biggest upgrade
- **Per-skill attribution** — heuristics to map tool call clusters back to a specific skill invocation
- **Export formats** — CSV, JSON summary for external dashboards
- **Alert thresholds** — warn when a single skill chain exceeds a cost threshold

## How to test locally

```bash
claude --plugin-dir ./claude-skill-meter
# Run any skill, then:
/claude-skill-meter:report
```

## Code style

Shell scripts: `bash`, `set -euo pipefail`, `shellcheck`-clean.

## License

MIT — contributions are accepted under the same license.
