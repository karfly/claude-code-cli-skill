---
name: "claude-code-cli"
description: "Use when the user wants to delegate a bounded pass to Claude Code CLI (`claude`) from Codex, another coding agent, a shell, or automation: codebase exploration, read-only review, plan comparison, implementation, debugging, test repair, or non-interactive scripting. Verify installed CLI help before using version-sensitive flags, keep cwd/permissions/prompt/output explicit, and reconcile Claude's results before reporting."
---

# Claude Code CLI Delegation

Use Claude Code CLI as a scoped second-pass agent. Keep the caller in control of the repository, permissions, prompt scope, output contract, and final verification.

## Inputs

- `repo`: absolute repository root or current working directory.
- `task`: one concrete exploration, review, plan, implementation, or debug objective.
- `scope`: files, directories, diff range, issue, failing command, or log excerpt.
- `mode`: supplied-context answer, read-only codebase pass, diff review, plan, bounded edits, or session continuation.
- `permissions`: whether Claude may read files, edit files, run tests, use network, install dependencies, create branches/worktrees, commit, or push.
- `output`: prose, JSON, stream JSON, changed files, findings, plan, or test summary.

## Quick Start

1. Verify the environment from the intended repo root:

```bash
pwd
git status --short --branch
command -v claude
claude --version
```

2. Choose the smallest delegation mode that can answer the task.
3. Write a prompt with an explicit goal, scope, permission boundary, validation request, and return format.
4. Run Claude Code CLI.
5. Inspect the result yourself: check stdout/stderr, review any diff, run relevant tests, and separate Claude's changes from pre-existing worktree changes.

If `claude` is missing or unauthenticated, stop and ask the user to install or authenticate Claude Code CLI. Do not silently switch to another model or agent.

## Workflow

### 1) Refresh CLI facts

Before using flags beyond the examples below, read [references/cli-surface.md](references/cli-surface.md) and re-run local help:

```bash
claude --help
claude <command> --help
```

Trust installed help for what this machine can run. Treat official docs as guidance for newer or remote behavior when they disagree with local help.

### 2) Pick a delegation mode

Use these recipes as starting points, then adapt the prompt to the concrete task.

**Supplied context only**

Use when all useful context is already in stdin. Disable tools to avoid accidental repo access.

```bash
printf '%s\n' "$LOGS" | claude -p --tools "" \
  "Explain the likely failure cause from this input only. Return 3 bullets max."
```

**Read-only codebase pass**

Use for architecture questions, file discovery, or independent diagnosis. Make "do not edit" explicit.

```bash
claude -p --permission-mode plan \
  "Find where session expiry is implemented. Do not edit files. Return files, functions, and a concise explanation."
```

**Diff review**

Use for independent review without granting file access beyond the diff.

```bash
git diff origin/main...HEAD | claude -p --output-format json --tools "" \
  "Review this diff for correctness bugs only. Return findings with severity, file:line when possible, and rationale."
```

**Plan comparison**

Use when another independent implementation plan would reduce risk.

```bash
claude -p --permission-mode plan \
  "Plan the smallest safe fix for the failing checkout test. Do not edit files. Include assumptions and validation commands."
```

**Bounded implementation**

Use only when edits are allowed. Limit tools and commands to the task.

```bash
claude -p --permission-mode acceptEdits \
  --allowedTools "Read Edit Bash(pnpm test *) Bash(git diff *)" \
  "Fix the regression in packages/api/src/session.ts. Keep the patch minimal. Run pnpm test --filter api. Do not commit."
```

**Session continuation**

Use `--resume`, `--continue`, `--name`, or `--session-id` only when continuity is required. Prefer one-shot `-p` calls for clean delegations.

### 3) Use a strict prompt contract

```text
You are running in <absolute repo path>.
Goal: <one concrete outcome>.
Context: <issue, failing command, diff range, important files, branch/base>.
Scope: <allowed files/directories or supplied input only>.
Permissions: <read/edit/test/network/install/commit/push boundaries>.
Validation: <commands to run, or explain why they cannot be run>.
Return: <exact format, findings fields, changed files, tests, open questions>.
Do not: <destructive actions, broad refactors, secret exposure, commits, pushes>.
```

Prefer fresh command output over paraphrased context. If the prompt would need more than one unrelated goal, split it into separate delegations.

### 4) Capture and reconcile

- Use `--output-format json` for automation and parse the structured result instead of scraping text.
- Capture stderr separately; auth, settings, permission, and MCP warnings may appear there.
- After edit-capable runs, inspect `git status --short`, `git diff`, and any files Claude says it changed.
- Run local verification yourself when the final answer depends on it.
- Attribute only verified net changes. Do not present Claude's conclusion as fact without checking the relevant repo evidence.

## Stop Rules

- Stop before running `claude update`, `claude install`, `claude doctor`, network-heavy commands, package installs, commits, pushes, branch changes, or worktree creation unless the user requested or approved that system/repo change.
- Stop if the worktree has unrelated dirty changes that overlap the requested scope; ask or narrow the delegation.
- Do not use `--dangerously-skip-permissions` unless the environment is disposable, isolated, and explicitly intended for bypass mode.
- Do not pass secrets, full tokens, private customer data, or unredacted production payloads into prompts or logs.
- If local help and docs disagree, prefer local help for invocation and mention the discrepancy only when it affects the task.

## Reference Map

- `references/cli-surface.md`: verified CLI flags, command families, source priority, and version-sensitive notes.
