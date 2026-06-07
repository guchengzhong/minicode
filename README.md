# minicode

minicode is a small, hackable AI coding agent for your terminal. It is built around a real agent loop: LLM streaming, local file tools, shell execution, permissions, session workspaces, Skills, MCP, and a polished interactive CLI.

It is intentionally compact. The core stays focused on coding-agent primitives, while domain-specific workflows can live in temporary scripts, local Skills, or MCP servers.

## Highlights

- Interactive terminal chat with a startup panel, recent activity, live run status, permission prompts, and slash commands.
- OpenAI-compatible provider support with quick plug-and-play API configuration.
- Per-session workspaces so agent artifacts do not clutter your project root.
- Built-in tools for reading, searching, shell commands, writing, editing, patching, todos, and Skills.
- Permission modes for safe review, accepted edits, plan-only runs, and explicit bypass.
- Local Skills and SkillHub install/search flows.
- MCP server lifecycle commands for stdio and HTTP servers.
- Human output, JSON output, and stream-json output for scripts and integrations.

## Requirements

- Bun 1.3+
- An OpenAI-compatible API provider

This repository is currently Bun-first.

## Quick Start

```bash
git clone <your-repo-url>
cd minicode
bun install
bun run typecheck
bun test
```

Run the CLI from source:

```bash
bun run src/cli/index.ts --help
bun run src/cli/index.ts
```

If you have linked or installed the command as `minicode`, use:

```bash
minicode
```

## Configure A Provider

The easiest path is to enter the interactive CLI and configure the provider there:

```bash
minicode
```

Then run:

```text
/config setup
```

You can also configure in one line:

```bash
minicode config use siliconflow <api-key> deepseek-ai/DeepSeek-V3.2
```

Or keep the key in your shell instead of writing it into `minicode.jsonc`:

```bash
export MINICODE_API_KEY="<api-key>"
minicode config use-env siliconflow MINICODE_API_KEY deepseek-ai/DeepSeek-V3.2
```

Supported endpoint aliases:

- `siliconflow` / `sf` -> `https://api.siliconflow.cn/v1`
- `openai` -> `https://api.openai.com/v1`
- `deepseek` -> `https://api.deepseek.com/v1`

You can always pass a full base URL instead of an alias:

```bash
minicode config use https://api.example.com/v1 <api-key> provider/model
```

Provider config is written to project-local `minicode.jsonc`. Secret values are redacted from `config get`, `config list`, and `config show` output.

## Usage

Start interactive chat:

```bash
minicode
minicode chat
```

Run a single task:

```bash
minicode run "summarize this repository"
minicode -p "explain src/cli/index.ts"
```

Resume work:

```bash
minicode -c "continue"
minicode run --session <session-id> "continue from here"
minicode session list
```

Useful output modes:

```bash
minicode run --output-format text "hello"
minicode run --output-format json "hello"
minicode run --output-format stream-json "hello"
```

Useful chat slash commands:

```text
/help
/config
/config setup
/clear
/paste
/tools
/session
/model
/mcp
/skills
/doctor
/exit
```

## Permissions

minicode has permission modes for different levels of autonomy:

```bash
minicode run --permission-mode default "inspect the repo"
minicode run --permission-mode acceptEdits "update the docs"
minicode run --permission-mode plan "plan the refactor"
minicode run --permission-mode bypassPermissions "do the task"
```

Modes:

- `default`: use configured permission rules.
- `acceptEdits`: auto-approve edit/write/patch ask rules, while keeping shell governed by normal rules.
- `plan`: allow read/planning tools and deny write/execute tools.
- `bypassPermissions`: allow all tool calls. `--yes` is an alias.

Interactive chat prompts before risky tools when the config says `ask`.

## Workspaces And Sessions

By default, each run gets a workspace under:

```text
workspace/<session-id>/
```

File tools and shell commands run in that session workspace. This keeps generated files and temporary scripts out of your project root unless you intentionally write there.

Sessions are stored under:

```text
.minicode/state/sessions/
```

## Configuration

Create or edit `minicode.jsonc` in your project root. A full example is available in [`minicode.example.jsonc`](./minicode.example.jsonc).

Common commands:

```bash
minicode config show
minicode config validate
minicode config list
minicode config get model
minicode config set ui.accentColor "#3B8EEA"
minicode config set sandbox.shellTimeoutMs 30000
```

`config set` rewrites `minicode.jsonc` as formatted JSON, so keep comment-heavy config templates under version control.

## Skills

Local Skills are discovered from:

- `.minicode/skills/**/SKILL.md`
- `.opencode/skills/**/SKILL.md`
- `~/.claude/skills/**/SKILL.md`
- `~/.agents/skills/**/SKILL.md`

Commands:

```bash
minicode skill init weather --description "Weather lookup workflow"
minicode skill list
minicode skill show <name>
minicode skill search "react"
minicode skill search --remote "react"
minicode skill path add ./my-skills
minicode skill doctor
minicode skill install <remote-id> --yes
```

Remote SkillHub search/install requires `SKILLHUB_API_KEY`.

## MCP

Use MCP for durable external tools that should not live in minicode core.

```bash
minicode mcp add stdio local-tools -- node ./mcp-server.js
minicode mcp add http remote-tools http://127.0.0.1:9876/mcp
minicode mcp list
minicode mcp test <name>
minicode mcp doctor
minicode mcp disable <name>
minicode mcp enable <name>
minicode mcp remove <name>
```

`mcp list` reports configured servers as `connected`, `failed`, or `disabled`.

## Development

```bash
bun install
bun run typecheck
bun test
```

Optional live-provider smoke:

```bash
bun run smoke:llm
```

## Design Notes

minicode does not try to hard-code every vertical capability into the core. For live data or one-off automation, the agent should use shell commands, temporary scripts, Skills, or MCP tools. Durable workflows can graduate into Skills or MCP servers.

This keeps the project small enough to understand, fork, and modify.

## License

MIT
