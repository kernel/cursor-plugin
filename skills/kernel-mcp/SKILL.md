---
name: kernel-mcp
description: manage cloud browsers, take screenshots, run playwright scripts, and manage browser profiles using kernel's MCP tools. use when the kernel MCP server is connected.
---

# kernel mcp tools

when the kernel mcp server is connected, you can create, automate, and manage cloud browsers directly. browsers spin up in <30ms.

## tools

**browsers:**
- `create_browser` — launch a cloud browser. returns session ID, CDP websocket URL, and live view URL.
- `get_browser` / `list_browsers` — check browser status
- `delete_browser` — terminate a browser session

**automation:**
- `execute_playwright_code` — run playwright/typescript against a browser. `page` object is already in scope. return values come back as the result.
- `take_screenshot` — capture the current browser state

**profiles:**
- `setup_profile` — create or update a profile via guided live browser session
- `list_profiles` / `delete_profile` — manage saved profiles

**apps:**
- `list_apps` — list deployed kernel apps
- `invoke_action` — execute an app action
- `get_deployment` / `list_deployments` — check deployment status
- `get_invocation` — check action results

**docs:**
- `search_docs` — search kernel documentation

## common patterns

### create → automate → screenshot → delete

```
1. create_browser with stealth: true, timeout_seconds: 300
2. execute_playwright_code to navigate and interact
3. take_screenshot to verify
4. delete_browser to clean up
```

always delete when done. always set timeout_seconds as a safety net.

### execute_playwright_code

`page` is already in scope. just write playwright code and return what you need:

```typescript
await page.goto("https://news.ycombinator.com");
const titles = await page.$$eval(".titleline > a", els =>
  els.slice(0, 10).map(e => e.textContent)
);
return titles;
```

```typescript
await page.goto("https://example.com/login");
await page.fill("#email", "user@example.com");
await page.fill("#password", "pass");
await page.click('button[type="submit"]');
await page.waitForURL("**/dashboard");
return await page.title();
```

use this for quick, self-contained tasks. for complex multi-step flows, write sdk code instead.

### profiles for persistent sessions

profiles let agents log in once and stay logged in. kernel manages auth so you don't have to.

1. `setup_profile` with a name — opens a live browser for login
2. log in through the live view
3. create future browsers with `profile_name` to reuse the session

### stealth mode

stealth mode automatically adds a recaptcha solver and residential proxy. use `stealth: true` for any real website. most production sites have bot detection.

for sites with aggressive detection, create a proxy first (`residential` or `mobile` type), then pass `proxy_id` along with `stealth: true`.

## tips

- use `stealth: true` by default for real websites
- always set `timeout_seconds` — sessions can run up to 72 hours max
- delete browsers when done, don't just rely on timeouts
- use `execute_playwright_code` for quick tasks, sdk for complex flows
- take screenshots to verify page state before extracting data
- use `search_docs` when you need help with a specific kernel feature
