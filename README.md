# .agents

Unified configuration for AI coding agents. One repo to manage context, skills, and MCP servers across [Claude Code](https://code.claude.com/docs), [Kiro](https://kiro.dev/cli/), and [OpenCode](https://github.com/opencode-ai/opencode).

Fork this repo, commit your own configuration, and run `setup` to wire everything into your local agent harnesses.

⚠️ **Your fork will contain personal context, conventions, and potentially sensitive configuration. Consider making it a private repository.**

## Installation

```bash
# Fork this repo on GitHub, then clone your fork
git clone git@github.com:YOUR_USER/.agents.git ~/.agents

# Run setup (installs symlinks and merges MCP config)
~/.agents/setup
```

`setup` will:
1. Create a `~/.agents` symlink if you cloned elsewhere
2. Symlink skills into each harness's skills directory
3. Symlink context files into each harness's rules/steering directory
4. Symlink prompts as slash commands into each harness's commands directory
5. Merge MCP server config (with secrets) into each harness's native format

Re-run `setup` after any change to this repo.

### Prerequisites

macOS or Linux. Windows is not supported.

Python 3 and at least one of these agents installed:
- [Claude Code](https://code.claude.com/docs) (`claude`)
- [Kiro CLI](https://kiro.dev/cli/) (`kiro-cli`)
- [OpenCode](https://github.com/opencode-ai/opencode) (`opencode`)

`setup` auto-detects which are installed and skips the rest. You can also target one:

```bash
~/.agents/setup claude
~/.agents/setup kiro
~/.agents/setup opencode
```

### The `--yolo` flag

```bash
~/.agents/setup --yolo
```

This persistently disables permission prompts in all harnesses (auto-approves tool use). It requires double confirmation. Use at your own risk.

## Structure

```
~/.agents/
├── setup               — wires everything into each harness
├── context/            — shared context loaded every session
│   └── CONTEXT.md      — standing directives, conventions, project notes
├── skills/             — reusable skill definitions (SKILL.md per directory)
│   └── weather/        — example skill
├── prompts/            — slash command prompts distributed to each harness
│   └── *.prompt.md     — prompt files (Kiro .prompt.md format)
├── mcp/                — MCP server definitions and secrets
│   ├── servers.json    — canonical server definitions (committed)
│   ├── secrets.json    — API keys for servers (gitignored)
│   └── *.servers.json  — per-harness overrides
├── config/             — harness-specific config (--yolo settings, etc.)
└── harnesses/          — auto-generated symlinks to ~/.claude, ~/.kiro, ~/.config/opencode
```

## Usage

### Adding context

Drop `.md` files in `context/`. These are loaded into every agent session as standing instructions — use them for coding conventions, project architecture, team preferences, etc.

```bash
echo "Always use TypeScript strict mode." > ~/.agents/context/typescript.md
~/.agents/setup
```

### Adding skills

Create a directory under `skills/` with a `SKILL.md` file:

```bash
mkdir ~/.agents/skills/deploy
cat > ~/.agents/skills/deploy/SKILL.md << 'EOF'
---
name: deploy
description: Deploy services to production via CI/CD pipelines.
---

# Deploy

## How to Use
...
EOF

~/.agents/setup
```

Skills are agent-readable instructions that teach the agent how to perform specific tasks.

### Adding prompts

Drop `.prompt.md` files in `prompts/`. These become slash commands in each harness (e.g., `opsx-explore.prompt.md` → `/opsx-explore`).

Prompts use Kiro's native format — markdown with a `description` frontmatter field:

```markdown
---
description: Enter explore mode - think through ideas and investigate problems
---

Your prompt instructions here...
```

`setup` adapts the file extension per harness:
- **Kiro**: symlinked as-is (`.prompt.md`)
- **Claude**: symlinked as `.md` into `~/.claude/commands/`
- **OpenCode**: symlinked as `.md` into `~/.config/opencode/commands/`

### Adding MCP servers

Add server definitions to `mcp/servers.json`:

```json
{
  "my-server": {
    "command": "npx",
    "args": ["my-mcp-server@latest"],
    "env": {}
  }
}
```

If the server needs secrets, create `mcp/secrets.json` (gitignored):

```json
{
  "my-server": {
    "env": {
      "API_KEY": "sk-..."
    }
  }
}
```

Then run `~/.agents/setup` to merge into all harnesses.

## MCP configuration

Server config is layered and merged in order:

1. `mcp/servers.json` — base definitions (committed)
2. `mcp/<harness>.servers.json` — per-harness overrides (committed)
3. `mcp/secrets.json` — secrets for all harnesses (gitignored)
4. `mcp/<harness>.secrets.json` — per-harness secrets (gitignored)

Later layers deep-merge into earlier ones, so overrides only need the fields they change.

### Enabling/disabling servers

New servers default to **disabled**. Each harness handles this differently:

| Harness | How to enable | State stored in |
|---------|---------------|-----------------|
| Claude | `/mcp` dialog in-session | `~/.claude.json` (per-workspace) |
| Kiro | Edit `~/.kiro/settings/mcp.json`, remove `disabled` key | `~/.kiro/settings/mcp.json` |
| OpenCode | Set `"disabled": false` in `mcp/opencode.servers.json`, re-run setup | `~/.config/opencode/opencode.json` |

`setup` preserves existing enable/disable state for Claude and Kiro. For OpenCode, state is derived from config on every run.

## How it maps to each harness

| What | Claude | Kiro | OpenCode |
|------|--------|------|----------|
| Context | `~/.claude/rules/*.md` | `~/.kiro/steering/*.md` | `instructions` field in config |
| Skills | `~/.claude/skills/*/` | `~/.kiro/skills/*/` | `instructions` field in config |
| Prompts | `~/.claude/commands/*.md` | `~/.kiro/prompts/*.prompt.md` | `~/.config/opencode/commands/*.md` |
| MCP | `~/.claude.json` → `mcpServers` | `~/.kiro/settings/mcp.json` → `mcpServers` | `~/.config/opencode/opencode.json` → `mcp` |

## Roadmap

Potential harness support:

- [ ] [Gemini CLI](https://github.com/google-gemini/gemini-cli)
- [ ] [Codex](https://github.com/openai/codex)
- [ ] Other

## License

MIT
