---
name: manage-mcp
description: Enable or disable MCP servers per harness (kiro, claude, opencode) and apply changes via setup.
version: 1.0.0
---

# MCP Server Management

Manage which MCP servers are loaded in each agent harness.

## Architecture

```
mcp/servers.json                 ← canonical server definitions (command, args, env, type only)
mcp/<harness>.servers.json        ← per-harness config overrides (same schema as servers.json)
mcp/secrets.json             ← per-server secrets, all harnesses (gitignored, merged third)
mcp/<harness>.secrets.json  ← per-server secrets, one harness (gitignored, merged last, optional)
setup [claude|kiro|opencode]  ← merges all layers, writes to each harness's native config
```

## File Locations

| File | Path |
|------|------|
| Servers | `~/.agents/mcp/servers.json` |
| Kiro overrides | `~/.agents/mcp/kiro.servers.json` |
| OpenCode overrides | `~/.agents/mcp/opencode.servers.json` |
| Secrets (all harnesses) | `~/.agents/mcp/secrets.json` |
| Secrets (per harness) | `~/.agents/mcp/<harness>.secrets.json` |
| Setup script | `~/.agents/setup` |

Where `~/.agents` is a symlink to the `.agents` repo.

## Per-harness behaviour

### Claude
`setup` writes `disabled` into `~/.claude.json` under `mcpServers`. New servers default to `disabled: true`. If no explicit `disabled` is set in any config layer, the existing state in `~/.claude.json` is preserved.

Enable/disable state can also be toggled per-workspace via the `/mcp` dialog — it writes to `projects["<cwd>"].disabledMcpServers`. Run `/reload-plugins` after to apply without restarting.

To suppress a server across all workspaces, remove it from `servers.json` entirely.

### Kiro
`~/.kiro/settings/mcp.json` is Kiro's live config. If you want a repo-managed default, set `"disabled": true` or `"disabled": false` in `~/.agents/mcp/kiro.servers.json`, then run `~/.agents/setup kiro` to apply it.

**Enable a server:**
```bash
python3 -c "
import json; f='$HOME/.kiro/settings/mcp.json'
d=json.load(open(f)); d['mcpServers']['SERVERNAME'].pop('disabled',None)
open(f,'w').write(json.dumps(d,indent=2)+'\n')
"
```

**Disable a server:**
```bash
python3 -c "
import json; f='$HOME/.kiro/settings/mcp.json'
d=json.load(open(f)); d['mcpServers']['SERVERNAME']['disabled']=True
open(f,'w').write(json.dumps(d,indent=2)+'\n')
"
```

Replace `SERVERNAME` with the server key (e.g. `playwright`). Shift-tab to cycle agents and reload.

### OpenCode
`opencode.servers.json` controls which servers are loaded. The setup script defaults all servers to `disabled: true` — **omitting `disabled` is the same as `disabled: true`**. To enable a server you must explicitly set `"disabled": false`.

Disabled servers are excluded from `~/.config/opencode/opencode.json` entirely. State is re-derived on every `setup` run, so `opencode.servers.json` is always authoritative.

**Enable a server:**
```bash
# In ~/.agents/mcp/opencode.servers.json, set explicitly:
{ "playwright": { "disabled": false } }

~/.agents/setup opencode
```

**Disable a server:**
```bash
# In ~/.agents/mcp/opencode.servers.json:
{ "playwright": { "disabled": true } }
# (or remove the entry entirely — omitting disabled defaults to true)

~/.agents/setup opencode
```

## Adding a new MCP server

1. Add the server definition to `servers.json` (standard fields only: `command`, `args`, `env`, `type`)
2. To enable it in OpenCode, add `"servername": { "disabled": false }` to `opencode.servers.json` (omitting the entry keeps it disabled). To disable it in Kiro or Claude, add `"servername": { "disabled": true }` to `kiro.servers.json` or `claude.servers.json`. All three harnesses default new servers to disabled.
3. Add any secrets to `secrets.json` under the server name key (or `<harness>.secrets.json` for harness-specific values)
4. Run `~/.agents/setup` to apply everywhere

## Show current state

```bash
cat ~/.agents/mcp/servers.json
cat ~/.agents/mcp/kiro.servers.json
cat ~/.agents/mcp/opencode.servers.json
```

## Reloading after setup

| Harness | How to reload |
|---------|---------------|
| **Claude** | `/reload-plugins` in the session |
| **Kiro** | Shift-tab to cycle agents (e.g. to planning and back) |
| **OpenCode** | Full restart required (no mid-session reload yet) |
