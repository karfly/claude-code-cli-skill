---
name: claude-code-cli
description: Delegate focused coding work to Claude Code through its CLI from Codex, another agent, scripts, or a shell. Use when a task benefits from an independent Claude Code pass for codebase exploration, implementation, review, debugging, test repair, planning comparison, or structured automation with `claude -p`, while preserving the caller's control over cwd, permissions, prompt scope, and output handling.
---

# Claude Code CLI

Use Claude Code as a bounded collaborator, not as an ambient second brain. Give it the exact repository root, task, constraints, permission posture, and expected output.

For current flags and source notes, read [references/cli-surface.md](references/cli-surface.md) before using options beyond the basic recipes below.

## Delegation Loop

1. Start from the intended repository root:

```bash
pwd
git status --short --branch
claude --version
```

2. Choose the smallest useful mode:

- **Answer from supplied context**: `printf '%s\n' "$LOGS" | claude -p --tools "" "Explain the likely failure cause from this input only."`
- **Structured answer**: `claude -p --output-format json "Summarize this project."`
- **Plan before edits**: `claude -p --permission-mode plan "Plan the minimal fix for the failing checkout test. Do not edit files."`
- **Review a diff**: `git diff main | claude -p "Review this diff. Return only actionable findings with file:line when possible."`
- **Bounded implementation**: `claude -p --permission-mode acceptEdits --allowedTools "Read Edit Bash(npm test *) Bash(git diff *)" "Implement the smallest fix for issue X. Run npm test for the touched package."`

3. Inspect the result yourself. Verify changed files, run relevant tests, and keep authorship of the final decision.

## Prompt Contract

Use this shape for delegated work:

```text
You are running in <absolute repo path>.
Goal: <one concrete outcome>.
Context: <issue, failing command, important files, branch/base>.
Scope: <allowed files or directories>.
Do: <actions to take>.
Do not: <destructive actions, broad refactors, commits, pushes, secrets>.
Validation: <commands Claude should run or explain why it cannot>.
Return: <summary format, changed files, tests, open questions>.
```

Prefer one focused delegation over a large vague prompt. Include fresh command output when it matters.

## Operating Rules

- Re-run `claude --help` when relying on a flag; Claude Code changes quickly.
- Keep `cwd` explicit. Use `cd /path/to/repo && claude ...` or set the subprocess working directory.
- Prefer `--output-format json` for scripts and logs; parse `.result` instead of scraping prose.
- Pipe narrow context when possible, such as `git diff`, logs, or failing test output.
- Use `--add-dir` only when Claude needs file access outside the current project.
- Use `--permission-mode plan` or `--tools ""` for research and planning passes.
- Avoid `--dangerously-skip-permissions` unless the environment is intentionally disposable and isolated.
- Tell Claude whether it may edit, run tests, install packages, use network, create worktrees, commit, or push.
- Capture stderr separately in automation; warnings and auth/config problems may not be in stdout.

## Good Delegations

```bash
claude -p --output-format json \
  "Find the likely cause of the failing test shown below. Do not edit files. Return 3 bullets max.

$(npm test -- --runInBand 2>&1)"
```

```bash
claude -p --permission-mode acceptEdits \
  --allowedTools "Read Edit Bash(pnpm test *) Bash(git diff *)" \
  "Fix the regression in packages/api/src/session.ts. Keep the patch minimal. Run pnpm test --filter api. Do not commit."
```

```bash
git diff origin/main...HEAD | claude -p --output-format json \
  "Review this diff for correctness bugs only. Return JSON-friendly Markdown with severity, file:line, and rationale. Do not mention style."
```

## Failure Modes

- If output is generic, rerun with exact files, failing commands, and a narrower return format.
- If Claude edits too much, stop and rerun with explicit scope plus permission/tool limits.
- If permissions block progress, either grant a precise tool pattern or ask for a plan instead of edits.
- If a session needs continuity, use named sessions intentionally; otherwise prefer one-shot `-p` calls.
- If docs and local help disagree, trust the installed `claude --help` for what this machine can run.
