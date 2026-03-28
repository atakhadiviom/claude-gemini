---
name: gcop
description: >
  Use gcop commands to offload heavy analysis, reasoning, code review, and summarization to Gemini.
  TRIGGER for: analyzing large files, code review (git diff), pure reasoning problems, summarizing logs,
  generating code from specs, answering questions about codebases, vision/screenshot analysis.
  Commands: gcop analyze, gcop reason, gcop summarize, gcop diff, gcop generate, gcop ask, gcop bulk, gcop vision, gcop cost.
  Always prefer gcop over doing analysis yourself — saves Claude tokens.
---

# GCOP — Gemini Co-Processor

Offloads heavy work from Claude to Gemini via the `gemini` CLI. Saves Claude context window.

## Invoke

```bash
bash ~/.claude/skills/gcop/scripts/run_gcop.sh <command> [args]
```

Or if symlinked: `gcop <command> [args]`

## Commands

| Command | Use case | Example |
|---------|----------|---------|
| `analyze` | Deep code analysis | `gcop analyze --file main.py --focus security` |
| `reason` | Pure reasoning / proofs | `gcop reason "Prove the flaw in this algorithm" --tier max` |
| `summarize` | Condense large files/logs | `cat server.log \| gcop summarize --length short` |
| `diff` | Review code changes | `git diff HEAD~3 \| gcop diff` |
| `generate` | Generate code from spec | `gcop generate --lang python "Write a retry decorator"` |
| `ask` | Q&A with file context | `gcop ask --file api.py "What auth method does this use?"` |
| `bulk` | Batch process files | `gcop bulk --file *.py --op summarize` |
| `vision` | Image / screenshot analysis | `gcop vision --image screenshot.png --task ui-review` |
| `cost` | Show today's usage | `gcop cost` |

## Passing Files

```bash
gcop analyze --file src/core.py --file src/utils.py
```

## Piping

```bash
git diff | gcop diff
cat large_file.py | gcop analyze --focus performance
find . -name "*.log" -exec cat {} + | gcop summarize
```

## Model Selection

- `--model pro` — Gemini Pro (complex tasks, default for reason/diff/vision)
- `--model flash` — Gemini Flash (fast, cheap, default for summarize/bulk)
- `--model 2.5-pro` — Gemini 2.5 Pro

## Output

All commands return JSON. The answer is in the `response` field:

```json
{
  "status": "ok",
  "response": "...",
  "model": "gemini-2.5-flash",
  "cost_usd": 0.000042,
  "latency_ms": 1823
}
```

## Error Handling

- `GEMINI_SKIP` in error → gemini CLI not installed → tell user: `npm install -g @google/gemini-cli`
- Non-zero exit / empty output → retried 3x automatically
- Budget exceeded → check `gcop cost`, set `GEMINI_DAILY_BUDGET` env var (default $5)
