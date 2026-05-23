# claude-skill-meter

> **Observability for Claude Code skill invocations.**  
> Know which skills burn the most tool calls — and what that costs you.

Claude Code skills are free to install. They're not free to run.

When you invoke a skill — your own, a community plugin like [drupal-claude-kit](https://github.com/codeitwisely/drupal-claude-kit), or any other — Claude chains tool calls internally. Without observability, you're flying blind on cost.

`claude-skill-meter` adds a lightweight PostToolUse hook that logs every tool invocation to a local JSONL file, then surfaces a per-session breakdown on demand.

---

## What it tracks

| Metric | Captured? |
|---|---|
| Tool call count per type (Bash, Write, Edit…) | ✅ |
| Tool I/O size → token proxy | ✅ |
| Estimated cost (tool I/O layer) | ✅ |
| Model reasoning tokens | ❌ hooks don't expose this |

**Honest limitation:** Claude Code hooks fire on tool use, not on model inference. The cost estimate is a lower bound — the tool I/O portion. For actual billing, [console.anthropic.com/usage](https://console.anthropic.com/usage) is the source of truth.

The tool call breakdown is still genuinely useful: it tells you *which skills are chatty*, which is the right optimization target.

---

## Install

### Option A — Load directly (no install)

```bash
claude --plugin-dir ./claude-skill-meter
```

### Option B — Load from GitHub

```bash
claude --plugin-url https://github.com/codeitwisely/claude-skill-meter/archive/refs/heads/main.zip
```

### Option C — Add to your project's `.mcp.json` / permanent config

Follow the Claude Code plugin manager docs:  
```
/plugin install https://github.com/codeitwisely/claude-skill-meter
```

---

## Usage

Once the plugin is loaded, tracking starts automatically — no setup required.

```
# Show session report
/claude-skill-meter:report

# Reset and start a fresh session
/claude-skill-meter:clear

# Or call the scripts directly
csm-report
csm-report --all     # include archived sessions
csm-report --clear   # archive current session
```

### Example output

```
╔══════════════════════════════════════════════╗
║        claude-skill-meter — Report           ║
╚══════════════════════════════════════════════╝

  Session start : 2026-05-23T14:02:11Z
  Last event    : 2026-05-23T14:18:43Z

── Tool call breakdown ────────────────────────
  Bash                       34 calls
  Read                       18 calls
  Write                       7 calls
  Edit                        4 calls

  Total tool calls : 63

── Token proxy (tool I/O only) ────────────────
  Model (pricing)  : claude-sonnet-4-x
  Input  tokens    : ~8420
  Output tokens    : ~2180

── Estimated tool I/O cost ────────────────────
  ~$0.0578 USD

  ┌─────────────────────────────────────────┐
  │  ⚠  Model reasoning tokens are NOT      │
  │     captured by hooks. This is a lower  │
  │     bound on actual cost.               │
  │     → console.anthropic.com/usage       │
  └─────────────────────────────────────────┘
```

---

## Configuration

### Custom pricing

Copy `config/pricing.json` to `~/.claude-skill-meter/pricing.json` and edit:

```json
{
  "model": "claude-haiku-4-x",
  "input_per_1m": "0.80",
  "output_per_1m": "4.00"
}
```

The plugin checks `~/.claude-skill-meter/pricing.json` first, then falls back to the bundled defaults.

### Environment variables

| Variable | Default | Description |
|---|---|---|
| `CSM_SESSION_DIR` | `~/.claude-skill-meter` | Where session files are stored |
| `CSM_SESSION_MAX_AGE` | `21600` (6h) | Seconds before a session is auto-archived |
| `CSM_PRICING` | `~/.claude-skill-meter/pricing.json` | Custom pricing file path |

---

## Session files

Session data is stored as newline-delimited JSON in `~/.claude-skill-meter/`:

```
~/.claude-skill-meter/
├── current.jsonl              # active session
├── session-20260523-140211.jsonl  # archived sessions
└── pricing.json               # optional custom pricing
```

Each line:
```json
{"ts":"2026-05-23T14:05:32Z","tool":"Bash","input_len":312,"output_len":1840}
```

---

## Requirements

- Claude Code (any recent version)
- `bash`, `jq`, `bc` — standard on macOS and most Linux distros

---

## Works great alongside

- [drupal-claude-kit](https://github.com/codeitwisely/drupal-claude-kit) — Drupal security guardrails for Claude Code
- Any community Claude Code skills plugin

---

## Contributing

Issues and PRs welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

By [CodeItWisely](https://codeitwisely.ai) — building AI-native Drupal delivery workflows.
