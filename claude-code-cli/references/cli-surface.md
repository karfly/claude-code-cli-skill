# Claude Code CLI Surface

Checked on 2026-06-22 against local `claude --help` from Claude Code `2.1.175`, local subcommand help, and official Claude Code docs:

- <https://code.claude.com/docs/en/cli-reference>
- <https://code.claude.com/docs/en/headless>
- <https://code.claude.com/docs/en/common-workflows>
- <https://code.claude.com/docs/en/settings>

## Source Priority

1. Installed `claude --help` and `claude <command> --help` for commands this machine can run now.
2. Official Claude Code docs for conceptual behavior and newly documented flags.
3. Existing repo scripts/config only after checking they match the installed CLI.

When sources differ, use local help for invocation and docs for context. Do not invent flags from memory.

## Refresh Commands

```bash
claude --version
claude --help
claude agents --help
claude mcp --help
claude plugin --help
claude ultrareview --help
```

## Print Mode

- `-p, --print`: non-interactive response and exit. Prefer this for delegation from another agent, scripts, hooks, and CI.
- `--output-format text|json|stream-json`: choose output shape. Use `json` for automation and `stream-json` for event streams.
- `--input-format text|stream-json`: choose input shape in print mode.
- `--json-schema <schema>`: validate structured output.
- `--max-budget-usd <amount>`: cap print-mode API spend.
- `--no-session-persistence`: avoid saving print-mode sessions.

Local help notes that non-interactive mode skips the workspace trust dialog. Use it only in directories the caller trusts.

## Model and Effort

- `--model <model>`: set a model alias or full model name.
- `--fallback-model <model[,model...]>`: use print-mode fallback models when the primary is unavailable.
- `--effort low|medium|high|xhigh|max`: set effort when supported.

Default to the best available runnable Opus-class model with high reasoning: `--model opus --effort high`. The `opus` alias is preferred because it tracks the best available Opus model for the local CLI/account. If the user explicitly asks for a concrete latest model, try `opus-4.8` only after verifying availability; if the CLI rejects it, fall back to `opus` and mention the fallback.

## Context and Configuration

- `--add-dir <directories...>`: allow access to additional directories.
- `--settings <file-or-json>`: load settings from a file or JSON string.
- `--setting-sources <sources>`: select setting sources such as `user`, `project`, and `local`.
- `--mcp-config <configs...>`: load MCP server config from JSON files or strings.
- `--strict-mcp-config`: ignore other MCP configs.
- `--system-prompt <prompt>`: replace the system prompt.
- `--append-system-prompt <prompt>`: append to the default system prompt.
- `--bare`: minimize startup customizations and ambient context.
- `--safe-mode`: disable customizations for troubleshooting.

Use `--bare` when reproducibility matters more than user/project customization. Use `--safe-mode` for broken local Claude configuration, not as a normal default.

## Permissions and Tools

- `--permission-mode default|acceptEdits|auto|bypassPermissions|dontAsk|plan`: choose the starting permission mode.
- `--allowedTools, --allowed-tools <tools...>`: allow specific tools or patterns.
- `--disallowedTools, --disallowed-tools <tools...>`: deny specific tools or patterns.
- `--tools <tools...>`: restrict built-in tools. Local help says `--tools ""` disables all tools.
- `--allow-dangerously-skip-permissions`: make bypass mode available without defaulting to it.
- `--dangerously-skip-permissions`: bypass permission checks.

Default to `--dangerously-skip-permissions` for Claude delegations, matching the user's preferred dangerous bypass mode. If a request says `--dangerously-bypass`, use the local CLI's `--dangerously-skip-permissions` flag. Useful local examples include `Bash(git *)`, `Bash(npm test *)`, `Edit`, and `Read`. Keep tool patterns narrow and task-specific.

## Sessions

- `--name, -n <name>`: name a session.
- `--resume, -r [value]`: resume by session ID or picker/search value.
- `--continue, -c`: continue the most recent conversation in the current directory.
- `--fork-session`: create a new session ID when resuming.
- `--session-id <uuid>`: use a specific session ID.

Prefer one-shot `-p` for auditable delegation. Use continuation only when prior Claude context is part of the requirement.

## Command Families

- `claude agents`: manage background agents. Local help supports JSON listing, cwd filtering, default model/agent/permission settings, plugin dirs, MCP config, and extra dirs.
- `claude mcp`: manage MCP servers with `add`, `add-json`, `list`, `get`, `remove`, `serve`, and related commands.
- `claude plugin|plugins`: manage Claude Code plugins.
- `claude ultrareview [target]`: run cloud-hosted multi-agent code review with `--json` and `--timeout`.
- `claude auth`: manage authentication.
- `claude project`: manage project state.
- `claude doctor`: check updater health. Local help notes it may spawn stdio servers from `.mcp.json`; run only in trusted directories.
- `claude update|upgrade`: check for updates and install if available.
- `claude install [stable|latest|version]`: install the native build.

Do not call update/install/doctor/project commands from another agent unless the user requested that system or project-state change.

## Version-Sensitive Notes

Official docs may mention flags not present in local `2.1.175` help. Examples observed in docs include flags such as `--max-turns` or permission-prompt tooling. Verify with local help before use.

Claude Code CLI changes quickly. Every reusable automation should either pin and check a minimum CLI version or run `claude --help` during setup and fail clearly when required flags are absent.
