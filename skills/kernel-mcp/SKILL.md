---
name: kernel-mcp
description: Manage cloud browsers, take screenshots, run Playwright scripts, and manage browser profiles using Kernel's MCP tools. Use when the Kernel MCP server is connected.
---

# Using Kernel MCP Tools

When the Kernel MCP server is connected, you have direct access to cloud browser management tools. Use these to create, automate, and manage browsers without writing SDK code.

## Available Tools

### Browser Management

- **`create_browser`** — Launch a new cloud browser. Returns session ID, CDP WebSocket URL, and live view URL.
- **`get_browser`** — Get details about a specific browser session.
- **`list_browsers`** — List all active browser sessions.
- **`delete_browser`** — Terminate a browser session.

### Automation

- **`execute_playwright_code`** — Run Playwright/TypeScript code against a browser. The `page` object is pre-configured and in scope. Return values are sent back as the result.
- **`take_screenshot`** — Capture a screenshot of the current browser state.

### Profiles

- **`setup_profile`** — Create or update a browser profile with a guided live session.
- **`list_profiles`** — List all saved browser profiles.
- **`delete_profile`** — Delete a browser profile.

### Apps

- **`list_apps`** — List deployed Kernel apps.
- **`invoke_action`** — Execute an action on a deployed app.
- **`get_deployment`** / **`list_deployments`** — Check deployment status.
- **`get_invocation`** — Check action execution results.

### Documentation

- **`search_docs`** — Search Kernel documentation for guides and API references.

## Common Workflows

### Create a browser, automate it, then clean up

1. Call `create_browser` with `stealth: true` if the target site has bot detection
2. Use `execute_playwright_code` to run automation steps against the session
3. Call `take_screenshot` to verify the result
4. Call `delete_browser` to clean up

### Execute Playwright code

The `execute_playwright_code` tool runs TypeScript/Playwright code with a `page` object already in scope:

```typescript
// Navigate and extract data
await page.goto("https://news.ycombinator.com");
const titles = await page.$$eval(".titleline > a", els => els.slice(0, 5).map(e => e.textContent));
return titles;
```

```typescript
// Fill a form
await page.goto("https://example.com/login");
await page.fill("#email", "user@example.com");
await page.fill("#password", "pass");
await page.click('button[type="submit"]');
await page.waitForURL("**/dashboard");
return await page.title();
```

### Use profiles for authenticated sessions

1. Call `setup_profile` with a profile name — this opens a live browser for manual login
2. Log in to the target site manually through the live view
3. Future browsers created with `profile_name` will have the saved session

### Deploy and invoke an app

1. Use `list_apps` to see deployed apps
2. Call `invoke_action` with the app name and action parameters
3. Use `get_invocation` to poll for results

## Tips

- Always set `timeout_seconds` when creating browsers to prevent orphaned sessions
- Use `stealth: true` for any site that might block automated browsers
- Call `delete_browser` when done — don't rely solely on timeouts
- Use `execute_playwright_code` for quick tasks; use the SDK for complex multi-step flows
- Screenshots are useful for verifying page state before extracting data
- Use `search_docs` when you need guidance on a specific Kernel feature
