---
name: kernel-mcp
description: manage cloud browsers, take screenshots, run playwright scripts, and manage browser profiles using kernel's MCP tools. use when the kernel MCP server is connected.
---

# kernel MCP tools

when the kernel MCP server is connected, you have direct access to cloud browser management. create, automate, and manage browsers without writing SDK code.

## tools

### browser management
- **create_browser** — launch a cloud browser. returns session ID, CDP websocket URL, and live view URL. supports stealth, proxies, profiles, viewports, headless mode.
- **get_browser** / **list_browsers** — inspect browser sessions
- **delete_browser** — terminate a browser and free resources

### automation
- **execute_playwright_code** — run playwright/typescript against a browser. `page` object is pre-configured. return values come back as the result.
- **take_screenshot** — capture the current browser state

### profiles
- **setup_profile** — create or update a browser profile with a guided live session
- **list_profiles** / **delete_profile** — manage saved profiles

### apps
- **list_apps** — list deployed kernel apps
- **invoke_action** — execute an action on a deployed app
- **get_deployment** / **list_deployments** — check deployment status
- **get_invocation** — check action results

### docs
- **search_docs** — search kernel documentation

## common workflows

### browse a website
1. `create_browser` with `stealth: true` if the site has bot detection
2. `execute_playwright_code` to navigate and interact
3. `take_screenshot` to verify
4. `delete_browser` to clean up

### execute playwright code

the `page` object is already in scope:

```typescript
await page.goto("https://news.ycombinator.com");
const titles = await page.$$eval(".titleline > a", els =>
  els.slice(0, 10).map(e => e.textContent)
);
return titles;
```

### use profiles for persistent sessions
1. `setup_profile` with a name — opens a live browser for you to log in manually
2. `create_browser` with `profile_name` to reuse the session
3. the browser starts already logged in

### scrape with stealth
1. `create_browser` with `stealth: true` — automatically adds recaptcha solver + residential proxy
2. `execute_playwright_code` to extract data
3. `delete_browser` when done

## tips
- always call `delete_browser` when done — don't leave browsers running
- use `stealth: true` for any site with bot detection
- set `timeout_seconds` as a safety net (default is 60s, max is 72h)
- use `headless: true` for faster execution when you don't need live view
- profiles save cookies and localStorage — use them to avoid re-authenticating
- no charges for idle time — you only pay when browsers are doing work
