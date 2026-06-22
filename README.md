# Claude Code CLI Skill

A small Codex skill for delegating focused work to Claude Code through the `claude` CLI. It is intentionally agent-agnostic: Codex, another coding agent, shell scripts, or a human can use the same invocation patterns.

## Install

From Codex, ask:

```text
Use $skill-installer to install karfly/claude-code-cli-skill from path claude-code-cli.
```

Manual install:

```bash
tmpdir="$(mktemp -d)"
git clone git@github.com:karfly/claude-code-cli-skill.git "$tmpdir"
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R "$tmpdir/claude-code-cli" "${CODEX_HOME:-$HOME/.codex}/skills/claude-code-cli"
```

Restart Codex after installing a new skill.

## Use

Invoke the skill when you want an independent Claude Code pass for codebase exploration, implementation, review, debugging, test repair, planning comparison, or CLI automation.

Example prompt:

```text
Use $claude-code-cli to delegate a bounded review of the current diff to Claude Code CLI.
```

The skill keeps the main workflow compact and links to a small CLI reference for flags that should be checked against the installed `claude --help`.
