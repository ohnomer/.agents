# mcp/

MCP server configuration for all agent harnesses (Claude, Kiro, OpenCode).

## File naming convention

Every file in this directory follows one of these patterns:

| Pattern | Purpose | Gitignored |
|---------|---------|-----------|
| `servers.json` | Canonical server definitions, shared across all harnesses | No |
| `<harness>.servers.json` | Per-harness config overrides (env tweaks, disabled state) | No |
| `secrets.json` | Secret values applied to all harnesses | Yes |
| `<harness>.secrets.json` | Secret values for one harness only | Yes |
| `*.example` | Copy-and-fill templates for gitignored files | No |

All files share the same top-level schema: an object keyed by server name, values merged via deep merge.

## Schema

```json
{
  "<server-name>": {
    "command": "...",
    "args": ["..."],
    "env": {
      "KEY": "value"
    },
    "type": "stdio",
    "disabled": true
  }
}
```

Only `servers.json` is expected to have the full definition for each server. All other files only need to include the fields they want to override.

### `disabled` state

`setup` enforces the same logical rules for all three harnesses — the mechanism differs because Claude and Kiro persist state in their own config files while OpenCode re-derives it every run.

| Case | Behaviour |
|------|-----------|
| `disabled` explicitly set in any config file | Enforced on every `setup` run, overrides any existing state |
| `disabled` not set, server already exists | Inherited from the current harness config (user toggle is preserved) |
| `disabled` not set, new server | Defaults to `true` |

**Claude** (`~/.claude.json`) and **Kiro** (`~/.kiro/settings/mcp.json`) write `disabled` into their config files. If no explicit value is set, setup preserves whatever is already there.

**OpenCode** (`~/.config/opencode/opencode.json`) enforces disabled state by omitting the server from the config entirely rather than writing a flag — there is no "existing state" to inherit, so it always re-derives from the merged config.

## Merge order

`setup` applies layers in this order for each harness, with later layers winning:

```
servers.json
  ↓
<harness>.servers.json   (harness config: env, disabled, etc.)
  ↓
secrets.json             (shared secrets: API keys)
  ↓
<harness>.secrets.json   (harness-specific secrets, optional)
```

## Current servers

| Server | Command | Notes |
|--------|---------|-------|
| `playwright` | `npx @playwright/mcp@latest` | Browser automation |

## Adding a new server

1. Add the full definition to `servers.json`
2. If it should be disabled by default, add `"disabled": true` to `servers.json` (or a specific `<harness>.servers.json` to target one harness)
3. If it needs secrets, add them to `secrets.json` (or a harness-specific `.secrets.json`)
4. Run `~/.agents/setup` to apply

## Files in this directory

```
servers.json              — canonical definitions
claude.servers.json       — Claude overrides
kiro.servers.json         — Kiro overrides
opencode.servers.json     — OpenCode overrides
secrets.json              — shared secrets (gitignored)
secrets.json.example      — template for secrets.json
claude.secrets.json       — Claude secrets (gitignored, optional)
kiro.secrets.json         — Kiro secrets (gitignored, optional)
opencode.secrets.json     — OpenCode secrets (gitignored, optional)
```
