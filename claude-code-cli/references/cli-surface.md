# Claude Code CLI Surface

Reference checked on 2026-06-22 against local `claude --help` from Claude Code `2.1.175` and official Claude Code docs:

- <https://code.claude.com/docs/en/cli-reference>
- <https://code.claude.com/docs/en/headless>
- <https://code.claude.com/docs/en/common-workflows>
- <https://code.claude.com/docs/en/settings>

Always prefer the installed CLI help for exact local behavior:

```bash
claude --version
claude --help
claude <command> --help
```

## Core Invocation

- `claude [prompt]`: start interactive Claude Code in the current directory.
- `claude -p, --print [prompt]`: print a response and exit. Use for agents, scripts, hooks, and CI.
- `--output-format text|json|stream-json`: choose print-mode output. Use `json` for automation and `stream-json` for event streams.
- `--input-format text|stream-json`: choose print-mode input. Use `text` unless implementing an NDJSON integration.
- `--json-schema <schema>`: validate structured output in print mode.
- `--max-budget-usd <amount>`: stop after a print-mode budget.
- `--model <model>`: set a model alias or full model name.
- `--fallback-model <model[,model...]>`: try fallback models in print mode when the primary is unavailable.
- `--effort low|medium|high|xhigh|max`: set effort for the session when supported by the selected model.

Docs may mention newer flags such as `--max-turns` or `--permission-prompt-tool`; verify they exist in local `claude --help` before using them.

## Context and Configuration

- `--add-dir <directories...>`: grant file access to additional directories.
- `--settings <file-or-json>`: load extra settings.
- `--setting-sources <user,project,local>`: choose setting sources.
- `--mcp-config <configs...>`: load MCP servers from JSON files or strings.
- `--strict-mcp-config`: ignore other MCP configurations.
- `--system-prompt <prompt>`: replace the system prompt.
- `--append-system-prompt <prompt>`: append to the default system prompt.
- `--bare`: minimize startup customizations and ambient context.
- `--safe-mode`: disable customizations for troubleshooting.

Settings precedence in the official docs is: managed, command-line arguments, local, project, user.

## Permissions and Tools

- `--permission-mode default|acceptEdits|auto|bypassPermissions|dontAsk|plan`: choose the starting permission mode.
- `--allowedTools, --allowed-tools <tools...>`: allow specific tools or patterns.
- `--disallowedTools, --disallowed-tools <tools...>`: deny specific tools or patterns.
- `--tools <tools...>`: restrict the available built-in tools. Use `--tools ""` for no tools.
- `--allow-dangerously-skip-permissions`: make bypass mode available without defaulting to it.
- `--dangerously-skip-permissions`: bypass permission checks. Reserve for disposable, isolated sandboxes.

Prefer explicit tool patterns for delegated implementation. Examples from local help include `Bash(git *)`, `Bash(npm test *)`, `Edit`, and `Read`.

## Sessions

- `--name, -n <name>`: name a session.
- `--resume, -r [id-or-name]`: resume a session.
- `--continue, -c`: continue the most recent conversation in the current directory.
- `--fork-session`: create a new session ID when resuming.
- `--session-id <uuid>`: use a specific session ID.
- `--no-session-persistence`: avoid saving print-mode sessions.

Use one-shot print mode for clean delegation. Use named sessions only when continuity is part of the task.

## Useful Commands

- `claude agents --help`: manage background agents. Local `2.1.175` supports `--json`, `--cwd`, default model/agent/permission settings, plugin dirs, MCP config, and extra directories.
- `claude mcp --help`: manage MCP servers with `add`, `add-json`, `list`, `get`, `remove`, `serve`, and related commands.
- `claude plugin --help`: manage Claude Code plugins.
- `claude ultrareview --help`: run cloud-hosted multi-agent code review, with `--json` and `--timeout`.
- `claude doctor`: check Claude Code updater health.
- `claude update`: check for updates and install if available.
- `claude install [stable|latest|version]`: install the native build.

Do not call update/install from an agent unless the user asked for that system change.
