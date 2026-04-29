# ~/.agents

Shared configuration, context, and MCP server management for AI coding agents (Claude, Kiro, OpenCode).

## Structure

```
~/.agents/
├── AGENTS.md           — this file
├── setup            — wires everything into each harness (run after cloning or editing)
├── mcp/                — MCP server definitions, per-harness config and secrets
│   ├── servers.json            — canonical server definitions
│   ├── claude.servers.json     — Claude-specific config overrides
│   ├── kiro.servers.json       — Kiro-specific config overrides
│   ├── opencode.servers.json   — OpenCode-specific config overrides
│   ├── secrets.json            — API keys and secrets, all harnesses (gitignored)
│   ├── secrets.json.example
│   ├── claude.secrets.json     — Claude-specific secrets (gitignored, optional)
│   ├── kiro.secrets.json       — Kiro-specific secrets (gitignored, optional)
│   └── opencode.secrets.json   — OpenCode-specific secrets (gitignored, optional)
├── context/            — shared context files loaded each session
├── harnesses/          — symlinks to each agent's config directory
│   ├── claude   -> ~/.claude
│   ├── kiro     -> ~/.kiro
│   └── opencode -> ~/.config/opencode
└── skills/             — reusable skill definitions (SKILL.md format)
```

## setup

The setup script wires this repo into each harness. Run it after cloning, adding skills/context, or editing MCP configs:

```bash
bash ~/.agents/setup            # all harnesses
bash ~/.agents/setup kiro       # just kiro
bash ~/.agents/setup claude     # just claude
bash ~/.agents/setup opencode   # just opencode
```

What it does:
- **~/.agents symlink**: creates `~/.agents` pointing to wherever the repo is cloned (if not already there)
- **Skills**: symlinks each `skills/*/` directory into the harness's skills folder
- **Context**: symlinks each `context/*.md` into the harness's rules/steering folder
- **MCP**: merges `servers.json` + `<harness>.servers.json` + `secrets.json` + `<harness>.secrets.json` into each harness's native MCP config

## mcp/

Layered MCP server configuration. Each layer is merged in order:

1. `servers.json` — canonical server definitions (command, args, env, type)
2. `<harness>.servers.json` — per-harness config overrides, same schema as `servers.json`
3. `secrets.json` — API keys and secrets, keyed by server name, applied to all harnesses (gitignored)
4. `<harness>.secrets.json` — harness-specific secret overrides, same schema (gitignored, optional)

Overrides use deep merge, so you only need to specify the fields you want to change.

### Per-harness behaviour

| Harness | Default state | State persistence | Target file |
|---------|--------------|-------------------|-------------|
| Claude | All servers **loaded and enabled** | Per-workspace (`disabledMcpServers` array in `~/.claude.json` under the project path key) | `~/.claude.json` (mcpServers) |
| Kiro | All servers **loaded but disabled** (`disabled: true`) | Global (`disabled` key in `~/.kiro/settings/mcp.json`) | `~/.kiro/settings/mcp.json` (mcpServers) |
| OpenCode | Only **explicitly enabled** servers loaded | Not persisted — re-runs `setup` to apply state from `opencode.servers.json` | `~/.config/opencode/opencode.json` (mcp) |

Key differences:
- **Claude**: `disabled: true` in the server definition is **ignored** by Claude Code. Enable/disable is controlled per-workspace via the `/mcp` dialog, which writes to `disabledMcpServers` for the current directory. There is no global disable — to suppress a server everywhere you must remove it from `mcpServers` entirely.
- **Kiro**: `disabled: true` on the server entry is respected globally. State is shared across all projects.
- **OpenCode**: Disabled servers are excluded from the config file entirely. State does not persist properly across `setup` runs — manage it via `opencode.servers.json`. **Servers default to disabled** — omitting `disabled` is the same as `disabled: true`. To enable a server you must explicitly set `"disabled": false`.

### Enabling/disabling servers

For **Claude**, use the `/mcp` dialog interactively. Running `setup` preserves whatever state is in `~/.claude.json`. To suppress a server across all workspaces, remove it from `servers.json` (or omit it from `claude.servers.json` if it's Claude-only).

For **Kiro**, edit `~/.kiro/settings/mcp.json` directly (remove the `disabled` key to enable, or set `"disabled": true`) and Shift+Tab to reload. Running `setup` preserves whatever state is in the target file.

For **OpenCode**, set `"disabled": false` in `opencode.servers.json` to enable, or `"disabled": true` (or omit the entry) to disable. Re-run `setup opencode` to apply. Disabled servers are not written to the config at all.

## context/

Markdown files containing standing directives that apply across all projects. Symlinked to `~/.kiro/steering/` for Kiro and `~/.claude/rules/` for Claude. OpenCode reads them via the `instructions` field in its config.

## harnesses/

Symlinks to each agent's config directory for easy navigation.

## skills/

Subdirectories each containing a `SKILL.md` file. `setup` symlinks these into each harness's skills directory automatically.
