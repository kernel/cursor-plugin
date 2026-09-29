# KERNEL

cloud browsers for your agent, served over KERNEL's hosted mcp server. your agent launches stealth chromium sessions in <30ms, drives them with playwright, a persistent browser repl, or computer-use actions, reuses logged-in profiles and managed auth connections, and records replays.

## install

1. open **cursor settings → plugins**.
2. search for **KERNEL**.
3. click **install**, then sign in to KERNEL when prompted.

## what's inside

| component | path | purpose |
|---|---|---|
| mcp server | `mcp.json` | our hosted mcp server at `https://mcp.onkernel.com/mcp` (streamable http) |
| skill | `skills/kernel-mcp/SKILL.md` | when and how the agent should use the KERNEL mcp tools |

## what the agent can do

| category | tools |
|---|---|
| session lifecycle | `manage_browsers` creates, lists, updates, and deletes browser sessions and reads their telemetry. `manage_browser_pools` keeps pre-warmed browsers ready to acquire |
| driving a browser | `execute_playwright_code`, `browser_repl`, `computer_action`, `webmcp`, `browser_curl`, and `exec_command` |
| state and auth | `manage_profiles` saves cookies and local storage. `manage_auth_connections` runs managed auth for third-party sites. `manage_proxies` and `manage_extensions` configure proxies and chrome extensions |
| other | `manage_replays` records mp4 replays (paid plans). `manage_apps` invokes KERNEL apps. `search_docs` searches our documentation |

the hosted server is the source of truth for tool names and schemas.

## auth and network access

the plugin carries no api key. when it connects, you sign in to KERNEL over oauth 2.1 in your browser. during authorization you can grant org-wide access or limit it to one KERNEL project.

the plugin only talks to `https://mcp.onkernel.com`. it runs no local code, hooks, or install scripts. browsers run in our cloud and bill to the KERNEL account you sign in with.

for ci or other headless setups, you can skip oauth and pass an api key instead:

```json
{
  "mcpServers": {
    "kernel": {
      "type": "http",
      "url": "https://mcp.onkernel.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_KERNEL_API_KEY"
      }
    }
  }
}
```

the mcp server is open source: [kernel/kernel-mcp-server](https://github.com/kernel/kernel-mcp-server).

## docs

- docs: https://kernel.sh/docs
- sign up: https://kernel.sh

## license

mit
