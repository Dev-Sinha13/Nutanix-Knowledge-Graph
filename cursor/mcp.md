---
title: Cursor MCP — obsidian
type: cursor-config
created: 2026-08-31
updated: 2026-08-31
live_path: /Users/dev.sinha/.cursor/mcp.json
---

# MCP — `~/.cursor/mcp.json`

Cursor has no notion of “whatever vault is focused in Obsidian.” The path in this file is the vault the agent sees.

## As of 2026-08-31 (on disk)

This is what is actually in `~/.cursor/mcp.json` today. It does **not** include `Nutanix-Core`. It points at the parent vault and the hackathon vault — the original wrong-path footgun.

```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": [
        "-y",
        "obsidian-mcp",
        "/Users/dev.sinha/Documents/Obsidian Vault",
        "/Users/dev.sinha/hackathon/vault"
      ]
    }
  }
}
```

## Intended for this wiki

Use this if you want the agent on `Nutanix-Core` (keep other vaults if you still need them):

```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": [
        "-y",
        "obsidian-mcp",
        "/Users/dev.sinha/Documents/Obsidian Vault/Nutanix-Core",
        "/Users/dev.sinha/Documents/Obsidian Vault",
        "/Users/dev.sinha/hackathon/vault"
      ]
    }
  }
}
```

After changing the live file: restart Cursor → Settings → MCP → toggle on → trust first run. First start downloads `obsidian-mcp` via `npx` (needs network). Verify: list vaults, read `Home.md`.
