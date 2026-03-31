# Kernel Plugin for Cursor

Cloud browsers for AI agents. Launch, automate, and manage browsers in the cloud directly from Cursor.

## What's Included

### MCP Server

Connects Cursor to Kernel's cloud browser platform. Create browsers, run Playwright scripts, manage profiles, and deploy apps — all through natural language in Cursor.

### Skills

- **browser-automation** — Patterns for building browser automations with Kernel's TypeScript and Python SDKs, CDP connections, stealth mode, and profiles.
- **kernel-mcp** — Guide for using Kernel's MCP tools to manage browsers, run Playwright code, and work with profiles and apps.
- **web-scraping** — Web scraping patterns including bot detection avoidance, proxy configuration, dynamic content handling, and concurrent scraping.

### Rules

- **kernel-best-practices** — Best practices for Kernel automation code: cleanup patterns, timeout management, stealth mode usage, and error handling.

## Install

Search for **Kernel** in the Cursor plugin marketplace and install it.

## Setup

1. **Get a Kernel account** at [kernel.sh](https://kernel.sh)
2. Install the plugin — the MCP server connects automatically via OAuth
3. Start asking Cursor to create and automate cloud browsers

For API key authentication, add your key to `mcp.json`:

```json
{
  "mcpServers": {
    "kernel": {
      "url": "https://mcp.onkernel.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_KERNEL_API_KEY"
      }
    }
  }
}
```

## Usage Examples

With the plugin installed, you can ask Cursor things like:

- "Create a stealth browser and scrape the top 10 Hacker News posts"
- "Set up a browser profile for my GitHub account"
- "Take a screenshot of kernel.sh"
- "Write a Playwright script that fills out a form on example.com"
- "Deploy a Kernel app that checks a website every hour"

## Documentation

- [Kernel Docs](https://kernel.sh/docs)
- [TypeScript SDK](https://github.com/onkernel/kernel-ts-sdk)
- [Python SDK](https://github.com/onkernel/kernel-python-sdk)

## License

MIT
